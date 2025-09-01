---
title: The Substitution Model
date: 2020-04-14
tags: 
- Cornell
- 20SP
- CS3110
---

## [Introduction](https://www.cs.cornell.edu/courses/cs3110/2020sp/textbook/interp/simpl_subst_model.html)

When we evaluate `let` expressions like `let x = 42 in x+1`, we want to find a way to formally denote that it evaluates to 43 because `x` in `x+1` is substituted with 42. More generally, `let x = v1 in e2 --> e2 with v1 substituted for x`. That's where substitution comes into play. We want to express the right side of the arrow with a new notation `e2 {v1/x}`, which means `e2` with all occurrences of `x` substituted for `v1`. 

## [Substitution in Let Expressions](https://www.cs.cornell.edu/courses/cs3110/2020sp/textbook/interp/subst_simpl.html)

Suppose we have the following code

```ocaml
let x = e in 
	let x = e1 in e2

let x = e in 
	let y = e1 in e2
```

After one step of evaluation, we have 

```ocaml
(let x = e1 in e2){e/x}  =  let x = e1{e/x} in e2
(let y = e1 in e2){e/x}  =  let y = e1{e/x} in e2{e/x}
```

> Both of those cases substitute `e` for `x` inside the binding expression `e1`. That's to ensure that expressions like `let x = 42 in let y = x in y` would evaluate correctly: `x` needs to be in scope inside the binding `y = x`, so we have to do a substitution there regardless of the name being bound.
>
> But the first case does not do a substitution inside `e2`, whereas the second case does. That's so we *stop* substituting when we reach a shadowed name. Consider `let x = 5 in let x = 6 in x`. We know it would evaluate to `6` in OCaml because of shadowing. Here's how it would evaluate with our definitions of SimPL:
>
> ```ocaml
>     let x = 5 in let x = 6 in x
> --> (let x = 6 in x){5/x}
>   = let x = 6{5/x} in x      ***
>   = let x = 6 in x
> --> x{6/x}
>   = 6
> ```
>
> However, if we use the second case to evaluate it, it will evaluate to 5, which is incorrect.

## [Substitution in Functions](https://www.cs.cornell.edu/courses/cs3110/2020sp/textbook/interp/subst_lambda.html)

### Substitution Rule of Function Application

```Ocaml
x{e/x} = e
y{e/x} = y
(e1 e2){e/x} = e1{e/x} e2{e/x}
```

#### Call by Value

```ocaml
e1 e2 ==> v
  if e1 ==> fun x -> e
  and e2 ==> v2
  and e{v2/x} ==> v
```

It requires the value of `e2` to be evaluated to a value before apply function `e1`. 

#### Call by Name

```Ocaml
e1 e2 ==> v
  if e1 ==> fun x -> e
  and e{e2/x} ==> v
```

This can lead to greater efficiency if the value of `e2` is never needed.

### Substitution Rule of Function Definition

#### Problem We Encountered (only in Call by Name case)

```ocaml
(fun x -> e'){e/x} = fun x -> e'
(fun y -> e'){e/x} = fun y -> e'{e/x}
```

These rules are not correct. Consider this following case:

```ocaml
(fun y -> x){z/x} = fun y -> x{z/x} = fun y -> z
(fun z -> x){z/x} = fun z -> x{z/x} = fun z -> z
```

Note that according to Principle of Name Irrelevance, these two functions are exactly the same function. However, after substitution, the second one became the identity function. Clearly, something went wrong here. 

What causes this problem is **variable capturing**. In the second function, the expression we are trying to substitute x with (z in {z/x}) is captured by the function argument (z in fun z -> ...) . Namely, the substitute and the argument, which shouldn't relate to each other at all, become the same thing in this situation. 

#### Capture-Avoiding Substitution

```ocaml
(fun x -> e'){e/x} = fun x -> e'
(fun y -> e'){e/x} = fun y -> e'{e/x}  if y is not in FV(e)
```

`FV(e)` means free variables of `e` and represent the variables that are not bound in `e`. Or you can think of `FV` as the variables that can be substituted in `e`. We only step when `y` is not in `FV(e)` because it means that `y` is already bound to something uniquely, so the problem that two different expressions denoted in the same variable bound to a same thing will not happen. 

```Ocaml
FV(x) = {x}
FV(e1 e2) = FV(e1) + FV(e2)
FV(fun x -> e) = FV(e) - {x}
```

This definition prevents the substitution `(fun z -> x){z/x}` from occurring, because `z` is in `FV(z)`. 

However, this new stepping rule prevents us from stepping when y is in FV(e). In this case, we just have to change y to a different name and the function stays the same according to the Principle of Name Irrelevance. By changing its name, we mean replace every occurrence of y by, say, y1. Notice we say "replace", not "substitution" as a rule we defined for our model. Absolutely anywhere we see y, we replace it by something else. 

#### Implementation of Capture-Avoiding

> 1. Augment the evaluation relation to maintain a stream (i.e., infinite list) of unused variable names. Each time you need a new one, take the head of the stream. But you have to be careful to use the tail of the stream anytime after that. To guarantee that they are unused, reserve some variable names for use by the interpreter alone, and make them illegal as variable names chosen by the programmer. For example, you might decide that programmer variable names may never start with the character `$`, then have a stream `<$x1, $x2, $x3, ...>` of fresh names.
>
> 2. Use an imperative counter to simulate the stream from the previous strategy. For example, the following function is guaranteed to return a fresh variable name each time it is called:
>
>    ```ocaml
>    let gensym =
>      let counter = ref 0 in
>      fun () -> incr counter; "$x" ^ string_of_int !counter
>    ```