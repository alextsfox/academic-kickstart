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
I recently took a shot at building an actual software library, something I haven't really done before. I've made loads of scripts and one-off software projects for my own use or for research use, but never something meant to be distributed to the public. It turned into a substantial learning experience for me. I had to sharpen my C programming skills, learn how foreign function interfaces work, understand how software libraries are structured, and figure out how to distribute a Python package publicly.

The result of my work is creating PyOCN: a collection of tools for working with Optimized Channel Networks (OCNs) based on Carraro et al. (2020), *Generation and application of river network analogues for use in ecology and evolution. Ecology and Evolution.* doi:10.1002/ece3.6479. It mirrors some of the functionality of the OCNet R package (https://lucarraro.github.io/OCNet/), but implemented in python and C, and with some substantial performance improvements.

I first used OCNs in 2024 during a project as an intern at Oak Ridge National Lab. I liked the elegance of the algorithm but found the OCNet R package limiting: many hydroligists prefer to work in python over R, and the OCNet R package had some substantial performance and dependency issues. I tried rewriting the algorithm in Python, but it was painfully slow: roughly 30 minutes to generate a 64×64 grid that OCNet handles in seconds. Digging through OCNet revealed that it relies heavily on SPArse Matrix (SPAM), a Fortran library, so my pure-Python rewrite was doomed. I tried again in Julia, still without the performance I wanted. Eventually I shelved the project. A few months later, while procrastinating on my PhD thesis, I decided to give it one more shot, this time implementing the entire backend in C and wrapping it with Python bindings.

# OCN basics
OCNs generate synthetic river networks with a realistic branching structure. The idea is simple:

1. Initialize a spanning-tree network on a 2D grid. Each cell drains to one of its 8 neighbors (except the root, which has not outlet).
2. Randomly propose changing a cell’s outflow to a different neighbor while maintaining a spanning tree (no cycles, no multi-outflow nodes) without any crossed edges.
3. Accept or reject the proposal based on whether it lowers the network's "energy." This is done using a simulated annealing approach, where the probability of accepting a proposed change is given by:

$$
P(\mathrm{accept}) = \min\left(1, \exp\left(-\frac{E_\mathrm{proposed} - E_\mathrm{current}}{T}\right)\right),
$$

where $E$ is the network energy and $T$ is the temperature. Energy is

$$
E = \sum_i A_i^\gamma,
$$

with $A_i$ the drained area at cell $i$. $\gamma$ is a free parameter that controls the "steepness" of the network. Low $\gamma$ values produce more dendritic networks, and high values produce more parallel networks. An annealing schedule controls how $T$ decays over time, typically given as an exponential decay.

The algorithm runs until convergence ($\Delta E / E < \varepsilon$) or a fixed number of iterations. Temperature starts high and decays, allowing the network to settle into a low-energy structure.

Finally, a DEM can be generated from the OCN by assigning slopes to each edge in the network based on the following equation:

$$
\frac{z_i - z_{\mathrm{downstream}}}{\vert \vec x_i - \vec x_{\mathrm{downstream}} \vert} = k A_i^{\gamma - 1}
$$

where $z_i$ and $\vec x_i$ are the elevation and position of cell $i$, respectively, $z_{\mathrm{downstream}}$ and $\vec x_{\mathrm{downstream}}$ are the elevation and position of the downstream cell, respectively, and $k$ is a free parameter describing the vertical exaggeration of the landscape. $z_i$ for the root cell is set to zero.

Below is an example of an OCN optimizing with $\gamma = 0.375$, created with `PyOCN`. Blue/green colors indicate low elevation, and brown/white colors indicate high elevation. Since this OCN was optimized with a periodic boundary condition, the boundary of the watershed can change over time as flowpaths "jump" across the the boundary.

<div align="center">
  <img src="generation.gif" alt="Optimizing an OCN">
</div>

OCNs make for a good first C project: the algorithm is conceptually straightforward but requires careful handling of a nontrivial data structure (a spanning tree over a fixed grid with 0 crossed edges). And because it’s not vectorizable, I can't just shove my code into a bunch of NumPy arrays and expect a significant speed-up.

# libocn
OCNet represents the flow network using a sparse adjacency matrix, which is implemented in the SPAM Fortran library. I took a different approach: representing the grid directly as a directed acyclic graph (DAG) in a C library I created called `libocn`. Each cell in the grid is a `Vertex` structure:

```c
typedef struct {
    double area;    // Cumulative drained area
    char outflow;   // Direction of outflow (0-7)
    char edges;     // Bitmask of connected neighbors
} Vertex;
```

The bitmask encodes which of the 8 neighbors have edges. For example, a cell with edges in the cardinal directions N, SE, and SW has mask 0b00101001 (where bit 0 is N, 3 is SE, and 5 is SW):

```
7  O  1
   |
6  X  2
 /   \
5  4  3
```

A `FlowGrid` is a flat array of `Vertex` structs plus metadata:

```c
typedef struct {
    int nrows;
    int ncols;
    Vertex *grid;
    double total_energy;
} FlowGrid;
```
This design keeps memory usage low and adjacent cells close in memory. The `*grid` field is a pointer to an array of `Vertex` structs, stored as a 1d array in row-major order of shape `(nrows * ncols)`. I experimented with block-tiling instead of row-major ordering to improve cache locality, but the indexing overhead seemed to outweigh the benefits, and added complexity.

The tradeoff of this approach compared to the sparse matrix method is that traversing the graph takes more bookkeeping compared to using an adjacency matrix: the sparse matrix method is "prettier," in that matrix operations can be used to manipulate the graph directly. However, since most operations are local (changing a cell's outflow only affects that cell and its neighbors), we rarely need to operate on the entire graph at once.

# PyOCN
I compiled the backend into a shared library, `libocn.so`, and interfaced with it using Python’s `ctypes` library to create the `PyOCN` package. I learned later that I could have actually build `libocn` as a native Python extension module using the `PyModule` API, which would have allowed for tighter integration with Python. However, that approach requires a lot more boilerplate code to handle converting between Python objects and C structs than I was willing to write for this small project. Instead, `ctypes` let me redefine the C header files in Python so it could navigate `libocn` as a foreign library, and then I just built Python functions and classes that wrapped the raw C functions in a user-friendly interface.

The Python side centers on the `OCN` class, which wraps a `FlowGrid` structure and its associated optimization functions, in addition to providing constructors, customizable optimization routines, and various export, visualization, and analysis utilities. 

# Differences between the libocn and OCNet approaches
The approach that I take to optimizing the OCN has identical results to `OCNet`, but the implementation differs in the data structure used to represent the flow network.

As I mentioned earlier, `OCNet` uses a sparse adjacency matrix to represent the flow network, while `libocn` uses a custom DAG structure. This leads to some differences in how proposals are generated and evaluated. 

<div align="center">
  <img src="duality.png" alt="Data structure comparison">
</div>

In `OCNet`, changing a cell's outflow involves updating the adjacency matrix to reflect the new connection. Cycles in the modified network are detected by attempting to perform an incremental topological sort of the graph: if the sort fails, a cycle exists. The algorithm then leverages a nice identify to compute the drained areas: $(\mathbb{I} - \mathbf{W}^T)\vec A = \vec 1$. In this equation, $\mathbf{W}$ is the adjacency matrix, $\vec A$ is the vector of drained areas, and $\vec 1$ is a vector of ones. Solving this linear system yields the drained areas for all cells efficiently. For an upper triangular adjacency matrix $\mathbf{W}$, this can be solved with a forward-substitution algorithm, with has time complexity $O(N)$. $\mathbf{W}$ is permute to be upper triangular by permuting it according to the the topological sort order from the previous step. 

In contrast, the DAG approach that `libocn` takes requires explicitly traversing the graph to detect cycles and to update drained areas after modifying a flow path. Cycles are detecting by first traversing the graph downstream from the proposed new outflow cell to see if we reach the source cell. If we do, a cycle would be created, and the proposal is rejected. We then repeat this process, traversing downstream from the old outflow cell. To update the drained areas, we traverse the graph downstream from the old outflow cell, decrementing the drained areas of each visited cell by the drained area of the source cell. We then repeat this process, traversing downstream from the new outflow cell and incrementing the drained areas. 

Both of these operations (the DAG approach and the adjacency matrix approach) have time complexity $O(N)$ in the worst case, where $N$ is the number of cells in the grid. The DAG approach that `libocn` uses is easier to follow, conceptually, but is messier to implement in practice. In contrast, the adjacency matrix approach that `OCNet` uses is far more elegant, but has several steps that are hard to follow at first. 

Here's a diagram comparing the two approaches.

<div align="center">
  <img src="algos.png" alt="OCNet vs PyOCN">
</div>

# Building and distributing
Distributing a Python package with a C backend is not trivial. I had to learn the Python build chain, including how to get `setup.py` to compile the C code automatically. I didn’t want to distribute wheels (a pre-compiled binary Python package) for every platform, and I didn’t want users compiling the code manually, so the `setup.py` route made the most sense.

You can now install `PyOCN` from PyPI with `pip install PyOCN`.

The source code is on GitHub: https://github.com/alextsfox/PyOCN

Documentation is here: https://pyocn.readthedocs.io

# Future Work
Currently, `PyOCN` implements the DAG version of the OCN algorithm. I would like to try implementing the adjacency matrix version as well, to compare performance and code complexity. Right now, `PyOCN` also has some basic export, visualization, and analysis tools: you can export the OCN as a raster DEM or as a graph in `networkx`, but I would like to see some more advanced analysis tools added: stream order calculations, hydrologic unit delineation, and watershed extraction are high on my list.

# Wrapping up
`PyOCN` is still a hobby project. If you need well-tested academic software, `OCNet` is likely the more stable choice. But if you want a fast, Python-native way to play with OCNs and simulated river networks, `PyOCN` does the job well (and with love).

If you end up using it for something, I’d genuinely like to hear about it.

Thanks for reading!
