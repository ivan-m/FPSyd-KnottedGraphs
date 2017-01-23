---
title: Knotted Graphs
author: Ivan Lazar Miljenovic
date: 25 January, 2017
...

What is a graph?
================

Non-graphs
==========

## {id="not-graph-1" data-background="images/plot.png" data-background-size="auto 100%"}

Notes
:   * A plot
    * Visual representation of the relationships between variables

## {id="not-graph-2" data-background="images/chart.png" data-background-size="auto 85%"}

Notes
:   * A chart
    * Any form of graphical representation of data
    * The visualisations of graphs later on are charts.

## {id="not-graph-3" data-background="images/function.png" data-background-size="auto 85%"}

Notes
:   * A graph of a function
    * Usually lazily referred to as "a graph"
    * Makes me squick less than the other types

## {id="not-graph-4" data-background="images/cow.jpg" data-background-size="auto 100%"}

Notes
:   * This is a cow.  It goes "moo".

Let's see some graphs!
======================

## {id="graph-1" data-background="images/Konigsberg.png" data-background-size="auto 100%"}

Notes
:   * Konigsberg bridge problem: what started graph theory.

## {id="graph-2" data-background="images/bipartite.png" data-background-size="auto 100%"}

Notes
:   * A bipartite graph

## {id="graph-3" data-background="images/flow.png" data-background-size="auto 100%"}

Notes
:   * Flow/control graph

## {id="graph-4" data-background="images/planar.png" data-background-size="auto 100%"}

Notes
:   * Planar graph

## {id="graph-5" data-background="images/pandoc.jpg" data-background-size="auto 100%"}

Notes
:   * Formats that pandoc can convert from/to

## {id="graph-6" data-background="images/social.gif" data-background-size="auto 100%"}

Notes
:   * The social web

## {id="graph-7" data-background="images/knot.png" data-background-size="auto 100%"}

Notes
:   * A knot graph; _not_ what this talk is about

Definitions
===========

## What we use graphs to represent

. . .

> [A] **graph** is a structure amounting to a set of objects in which
> some pairs of the objects are in some sense "related".

Notes
:   * From Wikipedia
    * In other words, it models _relationships_.

## The usual suspects

> * Adjacency list
> * Adjacency matrix
> * Incidence matrix

Notes
:   * Most Haskell definitions are based on the adjacency list
    * Adjacency list: `[(Node, [Node], ...)]`; can be extended to also
      store edges in a separate list (but usually isn't)
    * Adjacency matrix: nodes ⨯ nodes
    * Incidence matrix: nodes ⨯ edges

## A more formal definition

. . .

> A graph `G` consists of a set `E(G)` of **edges** and a (disjoint)
> set `V(G)` of **vertices**, together with a relation of
> **incidence** which associates with each edge two vertices, not
> necessarily distinct, called its **ends**.

Notes
:   * Tutte, "A theory of 3-connected graphs"

## This sounds familiar...

. . .

So we have objects (nodes and edges) and a relationship (incidence).

## A graph is, well, a graph!

> *
>     ```haskell
>     data Graph nodes edges
>       = Gr (Graph {nodes | edges}
>                   incidence)
>     ```
> * If only it was that simple...

---
# reveal.js settings
theme: solarized
transition: zoom
backgroundTransition: slide
center: true
history: true
css: custom.css
...
