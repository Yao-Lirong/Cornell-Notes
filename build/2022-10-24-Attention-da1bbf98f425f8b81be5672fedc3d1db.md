---
title: Attention, Transformers, and Transfer Learning
date: 2022-10-24
tags:
- CS4787
---

## Sequence Models

- Pad to max length: works alright in most cases, but the following is a common failure case imagine max length is five

  - Hello Yao xx xx xx 
  - Oh hello Yao xx xx

  Though these two sentences are similar in both looking (offset only by 1) and meaning, they look pretty differently after padding. So they are rather different in the padded space. 

- Counting word appearance: this approach ignores order, so it is bad

- Recursive Neural Network: by design, it handles input in a sequential way, which has some limitations (forgetting previous reference, ambiguous reference, ...)

- Transformers: the superior choice, looks data in a parallel way 



## Recursive Neural Network 

RNN kind of resembles Finite State Machine, where you have the state transition function 
$$
s_{i+1} = \delta(s_{i}, x_i)
$$
except the state space RNN operates on is   the continuous real numbers. 

## Attention 

The attention model wants the tokens to interact not by ordering or position (as in RNN), but by how they look similar to each other in the embedding space. 

You can think of them as **differentiable relaxation of the concept of "dictionary"** -- Christian Szegedy
