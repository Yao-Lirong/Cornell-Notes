---
title: Advanced Data Types
date: 2020-02-04
tags:
- Cornell
- 20SP
- CS3110
---

From Textbook: [Advanced Data Types](https://www.cs.cornell.edu/courses/cs3110/2020sp/textbook/data/advanced.html)

---

## Algebraic Data Types

以前我们的 variants 比较像 `enum`，但是现在我们更像一个abstract class

<!--more-->

```ocaml
(*definition*)
type t = C1 | C2 of t2| ... | Cn (*of tn*)
(* t: 'a * 'a * ... *)

(*expression*)
C e
---or---
C
(* e: (e1:'a, e2:'a, ...) *)

(*pattern matching*)
C p
(* p: (e1:'a, e2:'a, ...) *)
```

examples: 

```ocaml
type point = float * float 

type shape =
  | Point  of point
  | Circle of point * float (* center and radius *)
  | Rect   of point * point (* lower-left and 
                               upper-right corners *)

let area = function
  | Point _ -> 0.0
  | Circle (_,r) -> pi *. (r ** 2.0)
  | Rect ((x1,y1),(x2,y2)) ->
      let w = x2 -. x1 in
      let h = y2 -. y1 in
        w *. h

type string_or_int =
| String of string
| Int of int

type string_or_int_list = string_or_int list

let rec sum : string_or_int list -> int = function
  | [] -> 0
  | (String s)::t -> int_of_string s + sum t
  | (Int i)::t -> i + sum t

let three = sum [String "1"; Int 2]
```





When do we need `[],(),{} | of`?

```ocaml
type node = {value:int; next:mylist}
and mylist = Nil | Node of node
```

- `[]`: list
- `()`: Constructor of a tuple 
- `{}`: Constructor of a record
- `|`: delineate different variants inside a type
- `of`: defining the construction of an algebraic type 



### Recursive Variants

```ocaml
type intlist = Nil | Cons of int * intlist

type 'a tree = 
  | Leaf 
  | Node of 'a node
and 'a node = { 
  value: 'a; 
  left:  'a tree; 
  right: 'a tree
}
```

### Parametrized Variants

No matter what kind of types we define, either a variant, a record, or a tuple. We need the type parameter `'a or ('a,'b)` when we define it. 

```ocaml
(* [Option] makes it safer to return nothing*)
type 'a option = None | Some of 'a

type 'a mylist = Nil | Cons of 'a * 'a mylist
type ('a,'b) pair = {first: 'a; second: 'b}
type ('a,'b) test = 'a * 'b
```

Similarly, when you want to declare a variable that has a parametrized type, you also need to give the type parameter.

```ocaml
type 'a tree = Leaf of 'a | Node of ('a * 'a tree * 'a tree)
let x:'a tree = Leaf 5
let x:int tree = Leaf 5
```

If you do `let x:tree = Leaf 5`, the compiler won't know what type you are talking about. 

### Polymorphic Variants

They would be better off with the name "anonymous variants," because you want to use them when these variants are only used in this specific function and not anywhere else. 

The constructor of polymorphic variants start with a " ` "

```ocaml
(* note: no type definition *)

let f = function
  | 0 -> `Infinity
  | 1 -> `Finite 1
  | n -> `Finite (-n)
  
val f : int -> [> `Finite of int | `Infinity ]

let lst = [`Pos 5; `Zero; `Neg (~-4); `Pos 3];;
val lst : [> `Neg of int | `Pos of int | `Zero ] list =
  [`Pos 5; `Zero; `Neg (-4); `Pos 3]
```

### Pattern Matching

```ocaml
type 'a tree = 
  | Leaf 
  | Node of 'a node

and 'a node = { 
  value: 'a; 
  left:  'a tree; 
  right: 'a tree
}

(* [mem x t] returns [true] if and only if [x] is a value at some
 * node in tree [t]. 
 *)
let rec mem x = function
  | Leaf -> false
  | Node {value; left; right} -> value = x || mem x left || mem x right
```



## Exceptions

### The Basics

```ocaml
(*Definition: it is just a special kind of "type"*)
exception E of t


(*Call an Exception*)
raise e

(*syntactic sugar*)
failwith "Not Good"
raise (Failure ("Not Good"))
```

### Pattern Matching

The following code says: try evaluating `e`. If it produces an exception packet, use the exception patterns from the original match expression to handle that packet. If it doesn't produce an exception packet but instead produces a normal value, use the non-exception patterns from the original match expression to match that value.

```ocaml
match 
  try e with
    | q1 -> e1
    | ...
    | qn -> en
with
  | r1 -> e1
  | ...
  | rm -> em
```