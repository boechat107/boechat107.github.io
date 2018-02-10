---
layout: single
title: "Travelling Salesman Problem (TSP)"
description: "A functional implementation to solve the problem of the shortest tour."
category: Research Problems
tags: [Haskell, Optimization]
mathjax: true
gallery:
    - url: http://users.cs.cf.ac.uk/C.L.Mumford/howard/FI1.gif
      image_path: http://users.cs.cf.ac.uk/C.L.Mumford/howard/FI1.gif
      alt: "Euclidean points"
      title: "Euclidean points"
    - url: http://users.cs.cf.ac.uk/C.L.Mumford/howard/FI8.gif
      image_path: http://users.cs.cf.ac.uk/C.L.Mumford/howard/FI8.gif
      alt: "tsp tour"
      title: "TSP tour"

---

This post comes from a work done over an undergraduate course,
[Programming Languages 2](http://www.inf.ufes.br/~raulh/), where the
[*Travelling Salesman Problem*](http://en.wikipedia.org/wiki/Travelling_salesman_problem)
was studied to analyse and apply different data structures, like binary tree variations.
The implementations discussed below are found at
[my github](https://github.com/boechat107/tsp_furthest_insertion_haskell).

The TSP problem can be summarized as:
given a set of Euclidean 2D points, the problem consist of finding the
shortest possible tour, which should pass over each point just once and come back to
the initial tour.

{% include gallery caption="" %}

Two different algorithms, or heuristics, were used to construct the tour (the path to
visit all vertices or cities):

* [*Farthest Insertion*](http://users.cs.cf.ac.uk/C.L.Mumford/howard/FarthestInsertion.html)
(FI)
* [*Double Minimum Spanning Tree*](http://en.wikipedia.org/wiki/Minimum_spanning_tree)
(DMST)

## Implementations and efficiency conjectures

### Farthest Insertion

The Farthest Insertion's heuristic consists of two basic actions:

* searching for the farthest free vertex (one that isn't yet in the tour) from the
tour;
* inserting the selected vertex in the tour in a way that the new tour is the
shortest possible path.

The distance between a free vertex and the tour is the distance between this vertex
and the closest vertex of the tour.
For this, an algorithm like
[*Nearest Neighbor*](http://en.wikipedia.org/wiki/Nearest_neighbour_algorithm)
can be used, selecting the free vertex whose distance from the tour is the greatest.

Suppose $C$ as the distance between two vertexes, $i$ and $j$ as vertexes
already in the tour and $r$ as the free vertex selected as pointed above. The
vertex $r$ must be inserted in the tour obeying the follow equation
$$\min C_{ir} + C_{jr} - C_{ij}$$

To understand better the problem and to compare the performance of different data
structures for indexing (storage of the free vertexes) and the tour, two different
implementation were done: the simplest, where both free vertexes and tour were
stored with simple lists (for a heap and for spacial indexing, respectively); and the
fastest, where the free vertexes were stored in a
[B-Tree](http://en.wikipedia.org/wiki/B-tree), as a priority queue, and the tour was
stored in a [K-d Tree](http://en.wikipedia.org/wiki/K-d_tree), for spacial indexing.

#### First version --- Lists

In this first implementation, both sets of vertexes, the free ones and the tour, are
stored using simple lists. The pour performance of this implementation comes from
the necessity of a full transversal of the lists for some operations over them.

A naive implementation would spend $O(n^2)$ to search for the
farthest vertex from the tour and $O(n^3)$ to run the whole
algorithm, i.e., to execute FI until all free vertexes are added to the tour.
To avoid repeated calculations and to keep the cost as $O(n^2)$,
when the farthest vertex is being searched, the distances between the free vertexes
and the tour can be updated considering only the last inserted vertex of the tour.

#### Second version --- B-tree and Kd-tree

In the second implementation, the free vertexes are stored in a B-tree and tour in a
Kd-tree. The main advantage of using a B-tree for the free vertexes is the
possibility of indexing them respectively to their distance to the tour, working like
a priority queue, in an always balanced tree. This means that we can take the
farthest vertex and update the tree always in $O(\log n)$, even for the worst case.

Every time we add a new vertex to the tour, creating therefore a new edge, the
distance of every remaining free vertex should continue the same or be decreased. We
can check this by searching for the new farthest vertex, in $O(\log n)$, and
recalculating its distance to the tour; if it didn't changed, we don't need to worry
about the tree's indexes and no updates need to be made; otherwise, we take the
vertex and inserted it again in the tree and repeat the procedure of searching the
farthest vertex and updating its distance. Note that unnecessary distance updates
are always avoided.

On the other hand, the Kd-tree is used as an efficient data structure to look for the
best insert position in the tour. We could say that the Kd-tree organizes their
vertexes different rectangles determined by the $x$ and $y$ coordinates (a visual
representation is given in the figure below taken from
[Wikipedia](http://en.wikipedia.org/wiki/K-d_tree)). The organization of the
tree favors searches involving multidimensional search keys, like the coordinates $x$
and $y$ in a 2D space. So, an insertion of a vertex by its coordinates can be done
in $O(\log n)$ too.

{% include figure
    image_path="http://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/Kdtree_2d.svg/500px-Kdtree_2d.svg.png"
    alt="kd-tree"
    caption="Visual representation of a Kd-tree structure" %}

Like the first implementation, the main loop of the algorithm is executed until
we don't have any remaining free vertex. The efficiency difference comes from
the search and insertion of the farthest vertex. The B-tree allows us to search
for the farthest vertex in $O(\log n)$, since we are using a balanced tree
and the distance update is done considering only the last vertex inserted in the
tour. As the Kd-tree divides the 2D space in two half-spaces, we can insert a
vertex using its $x$ and $y$ coordinates in $O(\log n)$. Therefore, this second
implementation of the Farthest Insertion can run in $O(n \log n)$.
