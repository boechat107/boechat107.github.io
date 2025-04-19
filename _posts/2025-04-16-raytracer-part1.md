---
layout: single
title: "Building a Ray Tracer: OCaml Modules (Part 1)"
description: "Working with OCaml modules to start building a Ray Tracer."
category: Programming
tags: [OCaml, Ray Tracer]
---

This is the first post on building a Ray Tracer using OCaml. I'm following
the book *The Ray Tracer Challenge* from Jamis Buck.

## Summary

* *Points* and *Vectors* are both *tuples*, but with different meanings. We can
  share their code while keeping their types distinct.

* In OCaml, *functor* means something different from Haskell. It's way to
  create modules from other modules, like a function that takes a module and
  returns another module. The syntax looks like `module MyFunctor ...`.

* We should place module interfaces -- defined with `module type` -- in `.ml`
  files. I found some public libraries using the suffix `_intf.ml` to name
  their files containing module interfaces. This allows both `.ml` and
  `.mli` files to reference the same module interface.

* I should probably use the library [`Owl`](https://github.com/owlbarn/owl)
  when the performance becomes an issue.

## Using Type Constructors

I started the definition of points and vectors like this:

``` ocaml
type point = Point of float * float * float

type vector = Vector of float * float * float
```

One issue with these definitions is that implementing equality functions for
points and vectors would duplicate code.

``` ocaml
let float_eq x y =
  Float.equal x y
  || Float.abs (x -. y) < 0.0001

let eq_point p1 p2 =
  match p1, p2 with
  | Point (p1_x, p1_y, p1_z), Point (p2_x, p2_y, p2_z) ->
     float_eq p1_x p2_x
     && float_eq p1_y p2_y
     && float_eq p1_z p2_z
```

From the example above, we can see how an `eq_vector` would basically duplicate
code. I wanted a single implementation that could handle both types, but that
should respect the semantic differences of each concept.

I decided to try using OCaml functors.

## Functors

``` ocaml
module Make () = struct
  type nt = float
  type t = {x : nt; y : nt; z : nt}

  let create x y z = {x; y; z}

  let eq p1 p2 =
    float_eq p1.x p2.x
    && float_eq p1.y p2.y
    && float_eq p1.z p2.z
end

module Vector = Make ()
module Point = Make ()
```

By using *functors*, we can define `Point` and `Vector` as modules with shared
implementation but distinct types. For example, we can compare two points using
`Point.eq`:

``` ocaml
Point.eq
  (Point.create 1. 2. 3.)
  (Point.create 1. 2. 3.00001);;
- : bool = true
```

And the compiler rejects semantically incorrect comparisons:

``` ocaml
Point.eq
  (Point.create 1. 2. 3.)
  (Vector.create 1. 2. 3.)
Error: This expression has type Vector.t
       but an expression was expected of type Point.t
```

We can write a function to add a vector to a point like this:

``` ocaml
let add_pv (p: Point.t) (v: Vector.t) =
  Point.create (p.x +. v.x) (p.y +. v.y) (p.z +. v.z)
```

Finally, I wanted to define a `.mli` file to expose the signature of my
functions and modules. However, to define the signature of `Vector` and
`Point`, I had to define an interface. Since the functor in the `.ml` file and
the modules in `.mli` should both reference this interface, I had to put it in
a third file that I called `linalgebra_intf.ml`.

``` ocaml
(* ==== linalgebra_intf.ml ==== *)
module type Coordinate = sig
  type nt = float
  type t = {x : nt; y : nt; z : nt}

  (** [create x y z] returns a new coordinate value. *)
  val create : nt -> nt -> nt -> t

  (** [eq x y] returns true if the two coordinates are equivalent. *)
  val eq : t -> t -> bool
end


(* ==== linalgebra.mli ==== *)
val float_eq : float -> float -> bool

module Point : Linalgebra_intf.Coordinate
module Vector : Linalgebra_intf.Coordinate

val add_pv : Point.t -> Vector.t -> Point.t

(* ==== linalgebra.ml ==== *)
let float_eq x y =
  Float.equal x y
  || Float.abs (x -. y) < 0.0001

module Make () = struct
  type nt = float
  type t = {x : nt; y : nt; z : nt}

  let create x y z = {x; y; z}

  let eq p1 p2 =
    float_eq p1.x p2.x
    && float_eq p1.y p2.y
    && float_eq p1.z p2.z
end

module Vector = Make ()
module Point = Make ()

let add_pv (p: Point.t) (v: Vector.t) =
  Point.create (p.x +. v.x) (p.y +. v.y) (p.z +. v.z)
```

## Conclusion

OCaml's type and module system can help us model different concepts and
safeguard their mathematical relationships even if their low-level
implementations are very similar -- like `Point` and `Vector`.
