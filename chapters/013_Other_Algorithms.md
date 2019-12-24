\null\clearpage

Other Useful Algorithms
========================================

\epigraph{Fancy algorithms are slow when n is small, and n is usually small.}{\textit{Rob Pike}}

World Generation
----------------

### Maze Generation

Maze generation is the base of a great majority of dungeon generation systems, you can create a maze, carve out a few rooms, put an entrance and an exit and you have a nice quick dungeon!

There are many ideas that can be used to generate a maze, some are based on a prepared map that gets refined into a maze, some other are based on walls instead of tiles, here we will see some of the many algorithms that exist.

#### Depth-First Search

The Depth-First Search (DFS) algorithm is known in the world of tree and graph structure as a traversal algorithm. We can use a randomized DFS algorithm as a simple maze-generation algorithm.

<!--TODO: Recursive DFS version -->

\placeholder

This algorithm can involve a big deal of recursion, which can lead to a *stack overflow*~[g]~ in your program, stopping the algorithm from working and your game in its entirety. It is possible to work around this issue by using an explicit stack, instead of using the call stack.

<!--TODO: Explicit stack DFS version -->
\placeholder

This algorithm, being taken from a Depth-First search algorithm, is biased towards creating very long corridors.

#### Recursive Backtracker Algorithm

The Recursive Backtracker Algorithm is one of the many algorithms used to generate mazes, in the case of a 2D grid this algorithm involves recursive calls to a function that takes care of carving a passage between two tiles.

<!-- TODO: Normal recursive backtracker -->
\placeholder

The issue with this algorithm is that it requires the full maze to be kept in memory, which could be a problem for larger worlds. Also, since this algorithm is based on recursion, there is a real danger of *stack overflow*~[g]~ that can be worked around by using an explicit stack.

<!--TODO: Recursive Backtracker with explicit stack and while loops -->

\placeholder

<!-- TODO: Introduce the user to maze generation with the recursive backtracker algorithm -->

#### Randomized Kruskal's Algorithm

This algorithm is based on a randomized version of the minimum-spanning tree algorithm known as Kruskal's algorithm.

\placeholder

This algorithm, being based on a minimum-spanning tree algorithm, is biased towards creating a big number of short dead ends.

Noise Generation
-----------------

### Perlin Noise

\placeholder

<!-- TODO: Noise generation algorithm, runs in O(2^n) for n dimensions -->