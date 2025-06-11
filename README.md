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

### Practical applications to heterogeneous-agent and sequence-space problems
See [examples.md](https://github.com/mrognlie/fast-code-macro/blob/main/examples.md) for discussion of how these six principles are applied to problems I've faced in heterogeneous-agent and sequence-space macro.

### Speed vs. software engineering
The focus of this lecture is on computational speed and efficiency, but good code isn't all about speed. Instead, there is a range of other considerations—readability, modularity, robustness, and so forth.

These considerations are outside the scope of this lecture, but for some thoughts on these issues geared toward a macro audience (intended mainly as notes to myself and my RAs) you can [check out this note](https://mrognlie.github.io/coding_guide.pdf).