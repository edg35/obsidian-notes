---
status: done
tags:
  - analysis
date: 2026-01-23
---
# Analysis of Algorithms

- accuracy, proof of correctness
- time complexity
- space complexity
- best case, average case, worst case scenarios
- optimality
- amortized analysis
- approximation, probabilistic guaranties

# Sample Problem


> [!question] Problem Statement
> - Given an array A[1 : n], find the largest element
> - **Input**: Array A of size n
> - **Output**: Maximum element in A

Algorithm:
1. Initialize Max = a[1]
2. for i = 2 to n
	1. if a[i] > Max, set
	2. max = a[i]
3. return Max

Proof By Induction:
- Base case: i = 1, Max = a[i]
- Induction step: 
	- assume Max = max {A[1],........,A[k]}.
	- at k+1: update max if A[K+1] > max
	- then max = max {A[1],........,A[k+1]}.
- Conclusion: max is the largest element in A

## Time Complexity:

How do we "measure" the running time of an algorithm?
- Experiments: plot the run times for various input sizes
- experiments need actual implementation, experiments only limited inputs
- comparing two algorithms need same software and hardware env
- general methods that don't look at implementations?

## Model of Computation

we will use the random access machine (RAM) model of computation

- a simple operation takes exactly one time step
- loops and subroutines are built from simple operations, and their runtime is the sum of the steps of those operation
- Memory access is unlimited and take one time steps, regardless of the whether the dat is stored in cache, RAM, or disk
- The running time is measure by counting the number of steps an algorithm takes for a given input

## Asymptotic Notation

![[Screenshot 2026-01-23 at 1.00.15 PM.png]]


$$
log^a n << n^e <<c^n
$$

