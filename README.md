# Making code fast, for macroeconomists
This is a Jupyter notebook lecture on "making code fast" that I have given several times over the years. It's designed primarily for quantitative macroeconomists, or researchers doing similar kinds of structural work, and it's written in Python, but most of the lessons are more general.

[Browse the lecture notebook here on GitHub here](https://github.com/mrognlie/fast-code-macro/blob/main/making_macro_code_fast.ipynb). (To download for yourself, click this and then the "download raw file" button on the top right once you're there.)

## Outline
Throughout, I emphasize the importance of *profiling* your code, both with simple tools like `%time` and `%timeit` and also more advanced tools like `%prun` and `%lprun` when necessary, to see where the bottlenecks are.

There are then six main principles:

1. **Compiled is quick, the interpreter inches**
    * Python and similar languages are "slow" because pure Python works through the interpreter, which has a lot of overhead. Moving bottleneck operations to compiled code, either by vectorizing them or by compiling them with a tool like Numba, is essential for speed.

2. **Memory matters**
    * Reading and writing to memory can dominate the direct computational cost of a routine. It's often useful to avoid unnecessary reads and writes (e.g. creating intermediate arrays and reading from them), and also to make sure that data is stored in an efficient, contiguous way.
  
3. **Big O is a big deal**
   * The algorithmic complexity of a function—whether it takes $O(N)$, $O(N \log N)$, or $O(N^2)$ operations, where $N$ is the size of the input—is crucial in determining how fast it will be in practice, especially for larger inputs.

4. **Favor fast functions**
   * Some functions, and implementations of functions, are far faster than others. For instance, when multiplying dense matrices, it's far, far better to use the built-in matrix multiplication than to implement equivalent functionality yourself. You should learn what operations are fast, and then build on those as much as possible.

5. **Savor sparsity and structure**
   * Although built-in matrix multiplication is extremely well-optimized for arbritrary dense matrices, in applications we often work with matrices that are either sparse (mostly 0s), heavily structured (e.g. a product or Kronecker product of two matrices), or both. Taking advantage of this structure, rather than explicitly constructing and working with large matrices, delivers huge gains.
   
6. **Don't duplicate drudgery**
   * If the results of one computation can be shared across many runs of another computation, try to cache or precompute them, rather than recalculating them every time. 



## How are these principles applied in practice for heterogeneous-agent models?
The lecture provides simple, self-contained examples of each principle, but these principles have been very useful in practice when working on heterogeneous-agent and sequence-space methods. For those interested, some examples include the following. (For more detailed context, see e.g. the [sequence-space Jacobian paper](https://shade-econ.github.io/sequence_space_jacobian.pdf) or lectures 3 and 7 of [the course materials here](https://github.com/mrognlie/econ411-3).)

* The overall transition matrix across idiosyncratic states in the standard incomplete markets (SIM) model is highly structured: it's the product of one matrix that maps between exogenous states $e$ and $e'$ and leaves the endogenous asset state unchanged, and another very sparse matrix that maps from the state $(e,a)$ to next period's assets, leaving the exogenous state unchanged. It is *extremely inefficient* to explicitly construct this matrix, even if it's represented as a sparse matrix. Instead, we store and apply the two parts of the product separately, using an efficient approach for each. (See e.g. the forward iteration and expectation function parts of [lecture 1 here](https://github.com/shade-econ/nber-workshop-2023?tab=readme-ov-file#first-lecture-online).) Under the hood, we *always* do this, even when—as in the SSJ paper—we use the full transition matrix as an object in the math. This is a classic application of **lesson 5: savor sparsity and structure**.


* For the SIM model, we rely on either built-in NumPy functions, which are compiled under the hood, or custom Numba-compiled functions for all bottleneck operations. This reflects **lesson 1: compiled is quick, the interpreter inches**. In our basic code, we only need a single Numba function, to apply the "asset lottery" part of the transition matrix. But [once we speed up our code (using careful profiling)](https://github.com/mrognlie/econ411-3/blob/main/notebooks/econ411_3_lecture3_supplement_speed.ipynb), we also replace several other key parts with Numba functions. One of these functions is `interpolate_monotonic`, which reduces the cost of interpolation from $O(N\log N)$ to $O(N)$, in an example of **lesson 3: big O is a big deal** very similar to the one in this lecture.


* For the two operations in the SIM model that can be efficiently represented as dense matrix multiplication—taking expectations with respect to $e'$ conditional on $e$, and iterating the distribution forward from $e$ to $e'$—we make sure to use built-in matrix multiplication, rather than doing a more manual computation, because matrix multiplication is so fast under the hood. This is an example of **lesson 4: favor fast functions**.


* The key improvement of the "fake news algorithm" in the [SSJ paper](https://shade-econ.github.io/sequence_space_jacobian.pdf) is that when compared to a brute-force approach, it reduces the number of costly, bottleneck operations—backward and forward iterations—from $O(T^2)$ to $O(T)$, where $T$ is the truncation horizon. This is an example of **lesson 3: big O is a big deal**.
   * This improvement is enabled by the insight that we can exploit symmetry to avoid redundant iterations. For instance, all that matters for the policy function is the distance to a shock, so if we've calculated how policy at time 10 responds to an income shock at time 15, we already know how policy at time 30 will react to an income shock at time 35. Similarly, we can avoid repeated forward iteration by doing a single pass to precalculate [expectation functions](https://github.com/mrognlie/econ411-3/blob/main/notebooks/econ411_3_lecture7_supplement_expfunctions.ipynb), which can then be applied to find the aggregate delayed effect of any perturbation to the distribution. This is **lesson 6: don't duplicate drudgery**.

   * Interestingly, the third step in the fake news algorithm is still $O(T^2)$ (it involves multiplying $T\times N$ and $N\times T$ matrices, where $N$ is the size of the idiosyncratic state space, so including $N$ it is $O(N T^2)$), but since this is just a giant matrix muliplication, it's still fast and rarely a bottleneck—another example of **lesson 4: favor fast functions**.


* One of the benefits of the sequence-space Jacobian approach more generally is that Jacobians can be *reused*. For instance, if we precompute the general equilibrium Jacobian that maps impulses of exogenous shocks to the resulting impulses of outcomes, then we can reuse this and compute results for millions of different shocks, almost instantaneously. This is an example of **lesson 6: don't duplicate drudgery**. In likelihood-based estimation, we can apply this to evaluate many different shock process parameters almost instantly; and although we need to redo some computations when model parameters change, we can often still reuse the household Jacobian. (See the SSJ paper and [this notebook](https://github.com/shade-econ/nber-workshop-2025/blob/main/notebooks/lecture6_estimation.ipynb) for examples.)
