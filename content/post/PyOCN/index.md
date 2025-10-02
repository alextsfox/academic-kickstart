---
title: "PyOCN"
date: 2025-09-28T16:12:15-06:00
draft: false
projects:
    - Hydrology
    - Programming
tags:
    - Hydrology
    - Procrastination
---

# Writing PyOCN
I tried my hand recently at creating an honest-to-god software library. This is the first time I've done this, so it was a pretty major learning experience. I got hone my C programming skills, learn about foreign function interfaces, how software libraries are structured and distributed, and ti distribute a public python package.

The library I made is called PyOCN. It's a collection of methods for working with Optimized Channel Networks (OCNs) based on Carraro et al. (2020). *Generation and application of river network analogues for use in ecology and evolution. Ecology and Evolution.* doi:10.1002/ece3.6479 and mirrors some of the functionality of the OCNet R package (https://lucarraro.github.io/OCNet/). I worked a little bit with these in 2024 during a project at Oak Ridge National Lab. I was impressed by the elegance of the OCN algorithm and was a little frustrated by the limitations of the OCNet package (no shade to the developers of OCNet, I just don't like R very much, and I found the package to be a little slow for my liking, and I always had trouble getting the the optional dependencies to work for me). I tried rewrite the core algorithm in Python, but alas it was incredibly slow. It would take 30 minutes to build a single 64x64 OCN, compared to seconds with OCNet. Looking under the hood of OCNet, I saw that it actually made liberal use of the SPArse Matrix library, which is implemented in Fortran. I tried my hand at doing it in Julia, but still couldn't get the performance I wanted. I shelved the project and forgot about it for a few months. In the meantime, I picked up *C Programming* by K.N. King and wrote some extremely simple programs from the homework problems there. Come September 2025, I'm in the home stretch of writing my PhD thesis and am desperately looking for a distraction. I figured I could try a final time to implement the OCN algorithm, this time in C. I would write the entire backend: not just the data structure, but the entire optimization algorithm too, and wrap it in a python interface. 

# OCN basics
OCNs are an algorithm for generating networks with structures similar to real river networks. The algorithm is as follows:

1. An initial stream network is generated, representing a spanning tree over a 2d grid of raster cells. Each cell in the grid (except the root, or roots if there are several trees that partition the raster) has a single outgoing edge that connects to one of its 8 neighboring cells. 
2. We randomly select a cell and propose to change its outflow to point to a different neighbor, ensuring that the spanning tree structure is maintained (i.e. no cycles are introduced, no edges cross each other, and each cell has a single outflow, except for the root). 
3. We choose whether or not to accept the proposed changed based a simulated annealing method, which is related to the Metropolis-Hastings algorithm. The probability of accepting a proposed change is given by:
$$
P(\mathrm{accept}) = \min\left(1, \exp\left(-\frac{E_\mathrm{proposed} - E_\mathrm{current}}{T}\right)\right)
$$
where $E$ is the energy of the network and $T$ is the "temperature" of the network, and $n$ is the iteration number. Energy is defined as
$$
E = \sum_{i} A_i^\gamma,
$$
where $A_i$ is the cumulative drained area at cell $i$, including itself, and $\gamma$ controls how the energy scales with drained area. 

The algorithm is run for a fixed number of iterations, or until the energy converges to a stable value. $T$ begins at a high value (usually the initial energy) and exponentially decays over time, "annealing" the network as it settles into a low-energy configuration. Note that a proposed change is always accepted if it results in a lower energy state ($E[n] < E[n-1]$).

Lower values of gamma allow very dendritic networks with lots fo branching. The following animation shows the process of optimizing an OCN with $\gamma=0.7$:

<div align="center">
  <img src="generation.gif" alt="Optimizing an OCN">
</div>

This is a great candidate for learning C because the algorithm is fairly simple, but it requires managing a non-trivial data structure and is not vectorizeable, so I wasn't able to "cheat" by using numpy.

# libocn
The OCNet R package is built on top of the SPArse Matrix library. In OCNet, the flow grid is represented as an adjacency matrix, where each row and column represents a cell in the grid, and a non-zero entry indicates an edge between two cells. This is a convenient representation for many graph algorithms because it allows for direct matrix operations to manipulate the graph. However, I chose a different implementation, which was to directly represent the network as a directed acyclic graph (DAG). Each cell in the grid is a `struct` with fields for outflow direction, drained area, and connected edges. 
```c
typedef struct {
    double area;       // Cumulative drained area
    char outflow;        // Direction of outflow (0-7)
    char edges;     // Bitmask of connected neighbors (8-bit integer)
} Vertex;
```

`edges` is a bitmask representing whether there is an edge connecting the cell to one of its 8 neighbors. For example, the following cell (in the middle) has the edges `0b00101001` (N, SE, and SW):

```c
7  O  1
   |
6  X  2
 /   \
5  4  3
```

`outflow` is an integer representing the direction of outflow. A `FlowGrid` is simply an array of `Vertex` structs, along with some metadata about grid size and total energy.

```c
typedef struct {
    int nrows;          // Number of rows in the grid
    int ncols;          // Number of columns in the grid
    Vertex *grid;      // 2D array of Vertex structs
    double total_energy; // Total energy of the network
} FlowGrid;
```

My thinking here is that this representation is pretty memory efficient. Each Vertex only needs 16 bytes, and adjacent cells will be stored relatively close together in memory. I also had the idea to implement the array in memory using block-tiling, which should be more cache-local than using a row-major layout. Ultimately, this made little difference: either it didn't matter that much for the sizes of arrays I was working with, or the overhead of computing the indices was so high that it was never faster than using a simple row-major layout. All operations on the network are local: changing a flow direction only affects the cell itself, and we rarely need to do operations on the entire graph at once (in most circumstances), meaning that we never have to worry about doing operations the entire adjecency matrix. The downside is that it takes a little bit of work to traverse the graph, since we need to be able to convert between 2D grid corrdinates, the 1D array index, and the bitmask representation of the edges. 

# PyOCN
I compiled libocn to a shared library, `libocn.so`. I learned that there are 2 primary methods of interfacing with C code from Python: using python's `ctypes` or using `PyModule` to write a C extension module. Using the latter allows for tighter integration with Python, since it turns the C code into a first-class Python module that can be imported directly into the Python runtime environment, but it also requires a lot more boilerplate code to handle converting between Python objects and C structs. I opted for the latter, which seemed simpler. In `ctypes`, you basically redefine the C header files for your library in native python so it know how to navigate the libary, and then just tell python where to find the `.so` or `.dll` file. You then just build python functions and classes that wrap these raw C functions in a user-friendly interface. 

The main feature of PyOCN is the `OCN` class, which wraps a `FlowGrid` struct and provides methods for constructing it from various inputs, calling the optimization algorithm from libocn, and exporting the network to various formats. I think it works pretty well. It's got nice progress bars and everything, and it can run pretty quickly. The .gif above was generated in about 2 minutes on my laptop, compared to much longer using OCNet.

# Building and distributing
My next hurdle was to figure out how to distribute the package. I've never actually done this before, so I had to learn about the entire python build chain. It turns out that distributing a python package with a C extension is not super straightforward. Either you have to expect users to compile the C code themselves (which is what I'd been doing up to that point), have their package manager do it for them through some setup.py trickery (which is what I opted for), or distribute pre-compiled binaries (called wheels) for each platform I wanted to support (I definitely did not want to do this).

Writing docs was new to me as well. When distributing a package with online documentation, you need to have specially-formatted documentation for packages like Sphinx to be able to parse them.

I figured it out in the end though, and you should be able to install PyOCN from PyPI with `pip install PyOCN`. The source code is available on GitHub at https://github.com/alextsfox/PyOCN and the documentation is at https://pyocn.readthedocs.io.

# Wrapping up
PyOCN is of course just a hobby project for now, so you should definitely use OCNet if you would prefer to put your trust in the full academic peer review process rather than the blog of some guy who doesn't quite yet have a PhD. But if you want to play around with OCNs in Python and make some pretty graphs (quickly), PyOCN is a good option. If one of the two people who end up seeing this *do* end up using PyOCN for something, I'd love to hear about it! 

Thanks for reading!