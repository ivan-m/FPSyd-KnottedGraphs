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

## {id="graph-8}

Notes
:   * The empty graph!

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
    * Can be easily amended to cover hypergraphs
    * No `edge = (vertex, vertex)`, so can add in extra stuff
      (e.g. directionality)!

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

Tie a knot in it
================

## Tying the knot

> * Using laziness to create a value that depends on itself.
> *
>     ```haskell
>     fibs = 1 : 1 : zipWith (+) fibs
>                                (tail fibs)
>     ```

## The idea

. . .

Can we use tying the knot at the conceptual/data structural level?

## How to go about it

> 1. Assume we have an existing graph data structure
> 2. Define a new graph data structure in terms of the existing one
> 3. ~~Recurse~~ Try to set `new == existing`
> 4. Profit ?!?

## Levels

|                          |          |               |
|--------------------------+----------+---------------|
| Theoretical              | Vertices | Arcs          |
| Existing graph structure | Nodes    | Edges         |
| New graph structure      | Objects  | Relationships |

## Specification

```haskell
class Graph g where
  type Vertex g
  type Arc    g

  vertices :: g -> Set (Vertex g)
  arcs     :: g -> Set (Arc g)
  incident :: g -> Vertex g -> Set (Arc g)
  ends     :: g -> Arc g -> Set (Vertex g)
```

Notes
:   * Assume some `Set` type
         - Actually `Bag` or `MultiSet`
    * For `ends`, non-hyper graphs require a 2-Set.
    * Should `incident` and `ends` return a `Maybe`?

## Existing structure

```haskell
instance Graph (GrEx node edge) where
  type Vertex (GrEx node edge) = node
  type Arc    (GrEx node edge) = edge

  ...
```

Notes
:   * This lets us plug in our preferred node and edge types.

## New Structure (1)

```haskell
data Node object relationship a where
  Object :: object -> Node Object

  Relationship :: relationship
                  -> Node Relationship

getO :: Node o r a -> Maybe o
getR :: Node o r a -> Maybe r
```

Notes
:   * Pseudo-GADT usage
    * Blindly assume that we can have an implicit `forall a. Node a`
      when we need it.
    * Deal with what the identifiers are later.

## New Structure (2)

```haskell
data Edge = Edge (Node Object)
                 (Node Relationship)

newtype GrNew object relationship
  = GrNew (GrEx (Node object relationship)
                Edge)
```

Notes
:   * Order of values in `Edge` is arbitrary
    * Using implicit `forall`

## New Structure (3)

```haskell
instance Graph (GrNew o r) where
  type Vertex (GrNew o r) = o
  type Arc    (GrNew o r) = r
  vertices   = mapMaybe getO . vertices
  arcs       = mapMaybe getR . vertices
  incident g = mapMaybe getR . incident g
                 . Object
  ends     g = mapMaybe getO . incident g
                 . Relationship
```
Notes
:   * Blindly unwrapping `GrNew` here... if only there was a library
      that could help with that...
    * Also assuming `mapMaybe` works on `Set` values
    * Lots of mirroring between objects and relationships here...

Unravelling the knot
====================

## Objects & Relationships

```haskell
data Edge = Edge (Node Object)
                 (Node Relationship)
```

> * No object has an edge with another object
> * No relationship has an edge with another relationship
> * So why are they combined in the same datatype?

## Nodes and edges are bipartite {data-background="images/bipartite.png" data-background-size="auto 100%"}

## Split them apart

> * Before: `G = (object | relationship) + edges`
> * Now:    `G = objects + relationships + edges`

Notes
:    * We now have three sets rather than the two found in the
       definition.

## Do we still need the `edges` set?

> * Explicit realisation of the `incident` and `ends` methods.
> * Why not attach those values directly to the `objects` and
>   `relationships`?
> * Convert these `Set`s into dictionaries.

## Haven't you just re-invented adjacency lists?

> * Not quite.
> * I've augmented it with an _incidence list_.
> * Most obvious implementations don't do this.
> * FGL in particular is an optimised adjacency list.

## What does this mean?

> * Most people are used to having to have an arbitrary `Node` type.
> * But what about edges?
> * With this, we need to decide upon an identifier for edges as well.

## The usual approach

`type Edge = (Node, Node)`

. . .

### 😠

## Why do this?

> * I'm trying to create an _arbitrary_ (hyper)graph structure.
> * I believe directionality, labels, planarity, etc. are _extra_
>   information that provide extra meta-relationships.
> * To differentiate multiple edges, we can't just rely on `(Node, Node)`.

Notes
:   * Some people still complain about having to get an `Int` to use
      as the identifier in FGL.

Simple, directed graphs
=======================

## To continue on though, let's use this simplified example.

Notes
:   * No multiple edges or loops
    * Directionality (could do with undirected, but have to have
      2-sets).


## Implementation

```haskell
data Simple node
  = Gr (Set node) (Set (node, node))

instance Graph (Simple node) where
  type Vertex (Simple node) = node
  type Arc    (Simple node) = (node, node)

  ...
```

Notes
:   * Assuming no metadata
    * Requires some more work for implementing `incident`

## Algebra of graphs

> * Recent series of blog posts by Andrey Mokhov.
> * Adds the following functions:
>
>     ```haskell
>     vertex  :: Vertex g -> g
>     overlay :: g -> g -> g
>     connect :: g -> g -> g
>     ```

Notes
:   * Not going to go through in detail
    * Just show how to implement

## vertex

```haskell
vertex n = Gr (singleton n) empty
```

## overlay

```haskell
-- Graph union
overlay (Gr n1 e1) (Gr n2 e2)
  = Gr (n1 `union` n2)
       (e1 `union` e2)
```

## connect

```haskell
-- As with overlay,
-- but add edges from n1 to n2
connect (Gr n1 e1) (Gr n2 e2)
  = Gr (n1 `union` n2)
       (e1 `union` e2
           `union` (zip n1 n2))
```

Notes
:   * Assuming a `zip` on `Set`s.

## This is actually how it was already done

> * Andrey had already used this exact datatype.
> * But I've "derived" it.

ex relatio
==========

## Is this new/novel?\?

> * Probably not that much.
> * I just wanted to see where I would get to.

## What's next?

> * Try and extend this further if possible.
>       - Help please!
> * Use this to implement a new library.

---
# reveal.js settings
theme: solarized
transition: zoom
backgroundTransition: slide
center: true
history: true
css: custom.css
...
