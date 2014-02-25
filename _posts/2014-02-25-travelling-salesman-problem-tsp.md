---
layout: post
title: "Travelling Salesman Problem (TSP)"
description: "A functional implementation to solve the problem of the shortest tour."
category: Research Problems
tags: [Haskell, Optimization]
mathjax: true
---
{% include JB/setup %}

# Travelling Salesman Problem

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

<div style="display:inline-block;width:100%;">
<img alt="Euclidean points" src="http://users.cs.cf.ac.uk/C.L.Mumford/howard/FI1.gif" style="float:left;border:1px green solid" width="250"> 
<img alt="TSP tour" src="http://users.cs.cf.ac.uk/C.L.Mumford/howard/FI8.gif" style="float:right;border:1px green solid" width="250">
</div>

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
already in the tour and $r$ as the free vertex selected as potinted above. The
vertex $r$ must be inserted in the tour obeying the follow equation 
$$\min C_{ir} + C_{jr} - C_{ij}$$

To understand better the problem and to compare the performance of different data
structures for indexing (storage of the free vertexes) and the tour, two different
implementation were done: the simplest, where both free vertexes and tour were
stored with simple lists (for a heap and for spacial indexing, respectively); and the
fastest, where the free vertexes were stored in a 
[B-Tree](http://en.wikipedia.org/wiki/B-tree), as a priority queue, and the tour was
stored in a [K-d Tree](http://en.wikipedia.org/wiki/K-d_tree), for spacial indexing.

#### First version - Lists

In this first implementation, both sets of vertexes, the free ones and the tour, are
stored using simple lists. The pour performance of this implementation comes from 
the necessity of a full transversal of the lists for some operations over them.

A naive implementation would spend $O(n^2)$ to search for the
farthest vertex from the tour and $O(n^3)$ to run the whole
algorithm, i.e., to execute FI until all free vertexes are added to the tour.
To avoid repeated calculations and to keep the cost as $O(n^2)$,
when the farthest vertex is being searched, the distances between the free vertexes
and the tour can be updated considering only the last inserted vertex of the tour.
