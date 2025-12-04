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
# Writing PyOCN: A Library for Generating Simulated River Networks
I recently took a shot at building an actual software library, something I haven't really done before. I've made loads of scripts and one-off software projects for my own use or for research use, but never something meant to be distributed to the public. It turned into a substantial learning experience for me. I had to sharpen my C programming skills, learn how foreign function interfaces work, understand how software libraries are structured, and figure out how to distribute a Python package publicly.

The result of my work is `PyOCN`: a collection of tools for working with Optimal Channel Networks (OCNs) inspired by the `OCNet` R package developed in Carraro et al. (2020), *Generation and application of river network analogues for use in ecology and evolution. Ecology and Evolution.* doi:10.1002/ece3.6479. It mirrors some of the functionality of the OCNet R package (https://lucarraro.github.io/OCNet/), but implemented in python and C, and with some substantial performance improvements.

I first used OCNs in 2024 during a project as an intern at Oak Ridge National Lab. I liked the elegance of the algorithm but found the OCNet R package limiting: many hydroligists prefer to work in python over R, and the OCNet R package had some substantial performance and dependency issues. I tried rewriting the algorithm in Python, but it was painfully slow: roughly 30 minutes to generate a 64×64 grid that OCNet handles in seconds. Digging through OCNet revealed that it relies heavily on SPArse Matrix (SPAM), a Fortran library, so my pure-Python rewrite was doomed. I tried again in Julia, still without the performance I wanted. Eventually I shelved the project. A few months later, while procrastinating on my PhD thesis, I decided to give it one more shot, this time implementing the entire backend in C and wrapping it with Python bindings.

# How do river networks form?
I want to give a brief overview of how river networks form and are structured in the real world before diving into the specifics of the OCN algorithm. River networks form in drainage basins, or watersheds. A drainage basin is a region of land where all precipitation that falls within it drains to a common outlet, such as a river mouth or lake. The boundaries of a drainage basin are defined by topographic highs, such as ridges or hills, which direct the flow of water downhill towards the outlet. This topographic map of the cirque of the towers in Wyoming shows a very obvious example of a watershed: all the water that falls within the cirque drains east into the North Popo Agie river.

<div align="center">
  <img src="cirqueofthetowers.jpg" alt="Topographic map of the cirque of the towers in Wyoming. Source: USGS">
</div>

Like trees, river networks are fractal in nature: in the above map, while all of the water in the cirque eventually ends up in the North Popo Agie river, it gets there by flowing through a series of ever-larger tributaries. Rivers start flowing in headwater streams, which are small channels that might only carry water during and immediately after rain events. These headwater streams converge to form larger streams, which in turn converge to form even larger rivers. This branching structure continues until the water reaches the main river channel, which carries the water to its final destination. You might hear about rivers having a "source:" the Mississippi river, for example, is said to "begin" at Lake Itasca in Minnesota. This might make sense from a political or cultural standpoint, but hydrologically speaking, the "source" of the Mississippi is actually the entire network of hundreds of thousands of tributaries that feed into it, starting from tiny headwater streams across the Midwest, Rocky Mountains, Appalachians, and the South (in fact, the Mississippi River isn't even the largest river in its own drainage basin: the Missouri River, which flows into the Mississippi just north of St. Louis, is longer, and the Ohio River, which flows into the Mississippi near Cairo, Illinois, moves more water each year).

<div align="center">
  <img src="mississippi.png" alt="Map of the Mississippi river network. Source: American Rivers">
</div>

Why does this branching structure form, though? The answer lies in the interplay between water flow, erosion, and landscape topography. When water flows over the land surface, it exerts a force on the soil and rock, causing erosion. As it pushes against and erodes the rock, the water in the river channel expends energy. The amount of energy expended in erosion is sometimes called the "power" of the river. Like many things in nature, river networks tend to evolve towards configurations that minimize energy expenditure. This means that rivers will naturally adjust their paths and shapes to reduce the amount of energy they need to transport water and sediment downstream. In OCNs, the two main factors that are assumed to control erosional power (and energy expendature) are discharge (the amount of water flowing through a channel) and channel slope (the steepness of the channel). Higher discharge and steeper slopes lead to more erosional power, which in turn leads to more energy expenditure. Therefore, river networks will tend to evolve towards configurations that balance these two factors in a way that minimizes overall energy expenditure. 

Over time, streams will find paths that minimize energy expenditure. This might happen during a flood event, causing a river network to spill its banks and carve out a new channel or shortcut an oxbow bend. It might also happen gradually over thousands of years as small erosion events slowly change the landscape. The result is a branching network of streams and rivers that efficiently transports water from the landscape to the ocean, while minimizing energy expenditure along the way. OCNs don't capture all of these physical processes, but they do a good job of capturing the large-scale fractal structure of river networks.

# OCN basics
OCNs are models of river network structure based on the principle of optimality: river networks evolve to minimize the total energy expenditure of transporting water through the network. A stream with high erosional power will meet resistance from the landscape as it incises its channel. In the case of OCNs, erosional power of a single segment $i$ of a river network is assumed to depend on (1) mean discharge ($Q_i$) and (2) channel slope ($s_i$). Both mean discharge and channel slope are assumed to depend on drained area ($A_i$):

$$
\begin{align}  
Q_i \propto& A_i \\
s_i \propto& A_i^{\gamma - 1}\\
E_i \propto& Q_i s_i \propto A_i^\gamma \\
\end{align}
$$

The energy dissapated by the entire river network in aggregate is then given by summing over all segments:

$$
E = \sum_i A_i^\gamma,
$$

Note that the outlet of the river network always contributes the same amount of energy dissapation, since it must always drain the same area (the entire watershed).

OCNs are generated by numerically minimizing this total energy. The parameter $\gamma$ tells the OCN how to weigh the importance of slope vs discharge in the energy budget. $\gamma$ takes values between 0 and 1, with common choices being around 0.5. The value of $\gamma$. In networks with $\gamma \approx 0$, small watersheds are assumed to be the steepest, and dominate the energy budget: the optimal OCN will be dominated by large watersheds. In networks with $\gamma \approx 1$, all watersheds are equally steep, and energy is dissapated evenly across the network: the optimal OCN could have a wide variety of watershed sizes.

To illustrate this, consider the elevation map of the following two extreme OCNs, optimized with $\gamma = 0.99$ (left) and $\gamma = 0.01$ (right). The unique streams in the low-$\gamma$ network are far more tortuous (top right), fewer in number, there are very few headwater catchments (bottom right), and the elevation provile is very flat (never exceeding 4 m in elevation). In contrast, the high-$\gamma$ network has many short, straight streams (top left), a large number of small headwater catchments (bottom left), and grows to almost 170 m in elevation.

<div align="center">
  <img src="ocn_compare_structure.png" alt="OCNs optimized with low and high gamma values">
</div>

The algorithm for generating an OCN follows a myopic search pattern, that mimics how river networks evolve over time through local erosion events. The basic steps of the algorithm are as follows:

1. Initialize a directed acyclic graph with no crossing that spans all of the cells in a 2D grid. Each cell drains into one of its 8 immediate neighbors (except the root, which has not outlet). This represents the initial structure of the river network, before optimizing.
2. Choose a random cell in the grid. Propose changing that cell’s outflow to point towards a random, different neighbor (assuming it doesn't break the graph structure). This represents a possible local erosion event.
3. Accept or reject the proposed change based on the Metropolis criterion, before going back to step 2. The Metropolis criterion accepts changes that lower the total energy of the network, and sometimes accepts changes that increase the total energy, to allow the network to explore a wider range of configurations. The probability of accepting a proposed change is given by:

$$
P(\mathrm{accept}) = \min\left(1, \exp\left(-\frac{E_\mathrm{proposed} - E_\mathrm{current}}{T}\right)\right),
$$

where $E$ is the total network energy and $T$ is a free parameter called "temperature." Note that a change is always accepted if it lowers the networks energy. Temperature is typically chosen by specifying an "annealing schedule," which provides $T$ as a function of timetime, typically given as an exponential decay. $T$ starts high to allow the network to explore a wide range of possible configurations, and decays over time to make the optimization algorithm more greedy as it settles into a low-energy state. The algorithm runs until convergence ($\Delta E / E < \varepsilon$) or a fixed number of iterations is reached. 

Below is an example of running the optimization algorithm with $\gamma = 0.375$. Blue/green colors indicate low elevation, and brown/white colors indicate high elevation. Since this OCN was optimized with a periodic boundary condition, the boundary of the watershed can change over time as flowpaths "jump" across the the boundary. I think it's quite beautiful, like an amoeba squirming around under a microscope.

<div align="center">
  <img src="generation.gif" alt="Optimizing an OCN">
</div>

More information on OCNs and network scaling dynamics can be found in:

Rinaldo, A., Rigon, R., Banavar, J.R., Maritan, A., Rodriguez-Iturbe, I. (2014). *Evolution and selection of river networks: Statics, dynamics, and complexity*. PNAS 111(7), pp 2417-2424. doi:10.1073/pnas.1322700111

# Algorithm implementation
The most popular existing implementation of the OCN algorithm is the `OCNet` R package developed by Carraro et al. (2020). This implementation uses a matrix representation of the flow network. The implementation is rather elegant, but I found it difficult to follow at first. Therefore I'll talk about my implementation (in `PyOCN`) of the OCN algorithm, and then I'll compare it to the `OCNet` approach. 

## The PyOCN implementation
In `PyOCN`, I represent the flow network directly as a directed acyclic graph (DAG), represented in code as an array of nodes with attributes indicating which neighbors they connect to and drain towards. Rerouting a cell's outflow (step 2 of the algorithm) involves pointing a node's outflow to a different neighbor (and updating the old and new neighbors' inflow edges accordingly). This change may introduce cycles in the graph, which are invalid in a river network. To check for cycles, `PyOCN` follows the downstream path from both the new and old downstream neighbors of the target node. If it ever encounters a single node twice, it knows that a cycle was created, and it goes back to step 1 (after undoing the re-route). The re-route changes the drained areas of all cells downstream of the source cell, which must be updated by traversing the graph again to reflect the new flow path. The energy of the modified network can then be computed by determining how the change in drained areas affects the total energy, and the proposal can be accepted or rejected based on the Metropolis criterion (step 3 of the algorithm).

## The OCNet implementation
The implementation in `OCNet` is quite different, relying on sparse matrix operations to represent and manipulate the flow network. In `OCNet`, the flow network is represented as a sparse adjacency matrix $\mathbf{W}$, where each entry $\mathbf{W}_{ij}$ indicates whether cell $i$ drains into cell $j$ ($\mathbf{W}_{ij}$ = 1) or not ($W_{ij}$ = 0). Rerouting a cell's outflow involves updating the adjacency matrix to reflect the new connection (and the loss of the old connection). The modified adjacency matrix is then checked for validity by performing an incremental topological sort to detect cycles. A topological sort is an ordering of the nodes in a directed graph such that for every directed edge from node $i$ to node $j$, $i$ comes before $j$ in the ordering. If a cycle exists in the graph, then the sort will fail, and the re-route will have to be un-done. The topological sort also comes in handy for computing drained areas efficiently by leveraging an identity of the adjacency matrix: $(\mathbb{I} - \mathbf{W}^T)\vec A = \vec 1$, where $\vec A$ is the vector of drained areas, and $\vec 1$ is a vector of ones. This linear system can be solved for $\vec A$ to yield the drained areas for all cells. This can be done very efficiently (in $O(n)$ time) if $\mathbf{W}$ is upper triangular. Luckily, the topological sort that was done to verify the integrity of the graph provides a permutation of the nodes $\mathbf{P}$ such that the permuated adjacency matrix $\mathbf{PWP}^T$ is upper triangular. Finally, the energy of the modified network can be computed using the updated drained areas, and the proposal can be accepted or rejected based on the Metropolis criterion (step 3 of the algorithm).

## Comparing the two implementations
Here's a diagram comparing the structure of the two approaches. On the left is the DAG approach used in `PyOCN`, and on the right is the adjacency matrix approach used in `OCNet`.

<div align="center">
  <img src="duality.png" alt="OCNet vs PyOCN">
</div>

This diagram roughly shows how the two implementation differ.

<div align="center">
  <img src="algos.png" alt="OCNet vs PyOCN">
</div>

I find the implementation of `PyOCN` more approachable, but it does require more bookkeeping and code to handle traversing the graph and updating drained areas. The `OCNet` approach is more elegant and easier to code, but it took me a while to understand how the various pieces fit together. Regardless, both implementations yield identical results, and the optimization time should scale roughly linearly with the number of cells in the grid. I suspect that the `PyOCN` implementation is faster in practice, since it can be easily optimized to only consider the relevant parts of the graph each iteration, whereas the `OCNet` implementation has to perform global sparse matrix operations each iteration.

# libocn
OCNs make for a good first C project: the algorithm is conceptually straightforward but requires careful handling of a nontrivial data structure (a spanning tree over a fixed grid with 0 crossed edges). And because it’s not vectorizable, I can't just shove my code into a bunch of NumPy arrays and expect a significant speed-up.

`OCNet` splits the implementation of the OCN algorithm between R and Fortran: R handles the logic, main optimization loop, and user interface, while Fortran handles the computationally intensive sparse matrix manipulations. In `PyOCN`, I decided to implement everything in C (except for the user interface), which I compiled into a shared library, `libocn`, that could be called from Python.

As I mentioned in the previous section, in `libocn`, I represent the flow network as a DAG. This is constructed as a flat array of `Vertex` structs. Each `Vertex` struct represents a single cell in the grid, and contains fields for the drained area, outflow direction, and a bitmask indicating which of its 8 neighbors it has edges to:

```c
typedef struct {
    double area;    // Cumulative drained area
    char outflow;   // Direction of outflow (0-7)
    char edges;     // Bitmask of connected neighbors
} Vertex;
```

The bitmask encodes which of the 8 neighbors have edges. For example, a cell with edges in the cardinal directions N, SE, and SW has mask `00101001` (where bit 0 is N, 3 is SE, and 5 is SW):

A `FlowGrid` stores an array of `Vertex` structs, along with metadata about the grid as a whole:

```c
typedef struct {
    int nrows;
    int ncols;
    Vertex *grid;
    double total_energy;
} FlowGrid;
```

The `*grid` field is a pointer to an array of `Vertex` structs, stored as a 1d array in row-major order of shape `(nrows * ncols)`. I experimented with block-tiling instead of row-major ordering to improve cache locality, but the indexing overhead seemed to outweigh the benefits, and added complexity.

# PyOCN
I compiled the backend into a shared library, `libocn.so`, and interfaced with it using Python’s `ctypes` library to create the `PyOCN` package. Apparently, I could have actually built `libocn` as a native Python extension module using the `PyModule` C API, which would have allowed for tighter integration with Python. However, that approach requires a lot more boilerplate code to handle converting between Python objects and C structs than I was willing to write for this small project, and likely limits the compatability of `libocn` with languages other than Python. Instead, `ctypes` let me redefine the C header files in Python so it could navigate `libocn` as a foreign library, and then I just built Python functions and classes that wrapped the raw C functions in a user-friendly interface.

The Python side centers on the `OCN` class, which wraps a `FlowGrid` structure and its associated optimization functions, in addition to providing constructors, customizable optimization routines, and various export, visualization, and analysis utilities. For example, you can convert graphs built in the `networkx` package to `OCN` object, export `OCN` objects to `networkx` directed graphs or to raster DEMs, and perform various hydrologic analyses on them such as calculating flow accumulation and stream order.

In PyOCN, you can create and optimize an OCN in just a few lines of code:

```python
import PyOCN as po
import matplotlib.pyplot as plt
# Create an OCN on a 64x64 grid with gamma=0.5, 
# with a periodic boundary condition and "hip-roof" initial structure
ocn = po.OCN.from_net_type("H", dims=(64, 64), 
    gamma=0.5, wrap=True, random_state=8472)
# optimize the OCN
ocn.fit()
# Visualize the resulting DEM
res = ocn.to_xarray()
res.elevation.plot(cmap="terrain")
plt.show()
```

<div align="center">
    <img src="example.png" alt="Output DEM from PyOCN example code">
</div>


# Performance comparison
Both of these operations (the DAG approach and the adjacency matrix approach) should have time complexity $O(N)$ in the worst case, where $N$ is the number of cells in the grid. I can't directly compare performance between the two different algorithms since `OCNet` is implemented in R and Fortran, while `PyOCN` is implemented in Python and C. Additionally, `OCNet` only delegates out the most basic backend stuff (like performing the raw sparse matrix operations) to Fortran, whereas `PyOCN` does everything except for the interface in `C`. However, in my tests, my `PyOCN` is much faster than `OCNet`. How much of this is to do with my algorithm and how much of it is to do with my choice of language is impossible to say for now. On an M1 Macbook Pro, `PyOCN` is consistently ~20-60x faster than `OCNet`. They both seem to scale roughly as $O(N)$ in practice, which aligns with my rough complexity computation.

<div align="center">
  <img src="time_compare.png" alt="Scaling performance of PyOCN vs OCNet. The shaded regions indicate the standard deviation across 5 ensemble runs.">
</div>

# Building and distributing
Distributing a Python package with a C backend is not trivial, as it turns out. I had to learn the Python build chain, including how to get `setup.py` to compile the C code automatically. I didn’t want to distribute wheels (a pre-compiled binary Python package) for every platform, and I didn’t want users compiling the code manually, so the `setup.py` route made the most sense.

You can now install `PyOCN` from PyPI with `pip install PyOCN`.

The source code is on GitHub: https://github.com/alextsfox/PyOCN

Documentation is here: https://pyocn.readthedocs.io

# Future Work
Currently, `PyOCN` implements the DAG version of the OCN algorithm. I would like to try implementing the adjacency matrix version as well, to compare performance and code complexity. Right now, `PyOCN` also has some basic export, visualization, and analysis tools: you can export the OCN as a raster DEM or as a graph in `networkx`, but I would like to see some more advanced analysis tools added: stream order calculations, hydrologic unit delineation, and watershed extraction are high on my list.

# Wrapping up
This project was a huge success for me: I learned a lot about C programming, Python packaging, and software design, and I created an alternative implementation of the OCN algorithm that is an order of magnitude faster than the existing library.

`PyOCN` is still a hobby project. If you need well-tested academic software, `OCNet` is likely the more stable choice. But if you want a fast, Python-native way to play with OCNs and simulated river networks, `PyOCN` does the job well (and with love).

If you end up using it for something, I’d genuinely like to hear about it.

Thanks for reading!
