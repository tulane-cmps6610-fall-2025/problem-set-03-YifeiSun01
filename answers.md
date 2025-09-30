# CMPS 6610 Problem Set 03
## Answers

**Name:**_____Yifei Sun____________________


Place all written answers from `problemset-03.md` here for easier grading.




- **1b.**

Because we are checking by index of every element in the list to see if it is the element we are looking or not, the work and span are both $O(n)$. 



- **1d.**

Work (T₁): Θ(n) — each element is compared once and participates in O(1) combines overall; parallel reduction is work-efficient. 
Span (T∞): Θ(log n) — the reduction combines results up a balanced tree of depth ~log n (comparisons can be done in parallel, then the OR tree dominates).



- **1e.**

We analyze the work $\(W(n)\)$ and span $\(S(n)\)$ of the parallel algorithm using `ureduce`, which splits into subproblems of sizes $\(\lfloor n/3\rfloor\)$ and $\(\lceil 2n/3\rceil\)$ and then applies a constant-time combine $\(f\)$.  

**Work recurrence.**  
Each level does constant extra work per node, and we make two recursive calls covering the entire input.  Hence  
$$W(n) = W\bigl(\lfloor n/3\rfloor\bigr)+W\bigl(\lceil 2n/3\rceil\bigr)+O(1).$$  
By induction (or by the recursion-tree method), this solves to  
$$W(n) = \Theta(n).$$  

**Span recurrence.**  
Since the two recursive calls are executed in parallel, the span satisfies  
$$S(n) = \max\bigl(S(\lfloor n/3\rfloor), S(\lceil 2n/3\rceil)\bigr)+O(1).$$  
Because \(2n/3\) > \(n/3\), the critical path comes from the larger branch, so  
$$S(n) = S(\lceil 2n/3\rceil) + O(1).$$  
That recurrence solves to  
$$S(n) = \Theta(\log_{3/2} n) = \Theta(\log n).$$  

Thus, with `ureduce`, the **work remains $\(\Theta(n)\)$**, and the **span is still $\(\Theta(\log n)\)$** (though with a larger constant factor in the log) compared to a fully balanced binary split.


- **2a.**

```python
procedure DEDUP_SEQ_BY_REDUCE(A)  // Input: sequence A of length n; Output: distinct elements in first-appearance order
    S := EMPTY_SEQUENCE()         // seen list in first-appearance order
    for k from 1 to n do          // must iterate sequentially to preserve "first occurrence"
        x := A[k]

        // membership via parallel tree reduction:
        // build B = [ (y == x) for y in S ] and reduce with OR and identity False
        found := PARALLEL_TREE_REDUCE_OR( map(y -> (y == x), S) )

        if found == False then
            APPEND(S, x)          // first time we see x
        end if
    end for
    return S
end procedure
```

I used the reduce method in the previous question. In previous question we implemented a method that checks if an element in an unordered list or not. We can do this question by iterate over every element in the list we are given and keep a new dedupcilated list and check if every lement is in that deduplicated list or not. If it is not in that list we can append it to that list.

Work–Span Proof for DEDUP_SEQ_BY_REDUCE

Setup. At step k (1 ≤ k ≤ n), the seen-list S has size k−1. The algorithm computes B = map(y -> (y == x), S) and then performs a parallel tree reduce with OR and identity False to obtain found.

Fact 1 (cost of a size-m parallel reduce). A balanced tree reduce with an associative operator and identity does Θ(m) work and Θ(log m) span. This is the standard bound for parallel reduce/“fold” and follows from the depth of the reduction tree. 

Per-step costs. At step k, building B does Θ(k) work, and reducing B does Θ(k) work and Θ(log k) span. Hence the step-k membership check has
$$W_k = \Theta(k),\quad S_k = \Theta(\log k).$$

Total work. Summing over k gives
$$W(n) = \sum_{k=1}^{n} W_k = \sum_{k=1}^{n} \Theta(k) = \Theta\left(\sum_{k=1}^{n} k\right) = \Theta(n^2).$$

Total span. The outer loop is sequential (each decision depends on all prior steps), so spans add:
$$S(n) = \sum_{k=1}^{n} S_k = \sum_{k=1}^{n} \Theta(\log k) = \Theta\!\big(\log(n!)\big) = \Theta(n \log n).$$
The equality $\sum_{k=1}^{n} \log k = \log(n!)$ is immediate, and Stirling-type bounds give $\log(n!) = n \log n - n + O(\log n)$, hence $\log(n!) = \Theta(n \log n)$. 

Conclusion. Under the standard SPARC-style cost model for map/reduce, DEDUP_SEQ_BY_REDUCE has
$$W(n) = \Theta(n^2),\quad S(n) = \Theta(n \log n).$$
These bounds match the classic results for tree reductions (Θ(m) work, Θ(log m) span) applied to a sequential sequence of membership checks. 







- **2b.**

## SPARC #1 — Concatenate then Deduplicate (no local pass)

**Idea.**  
Concatenate all lists $\(A_0, A_1, \dots, A_{m-1}\)$ into one long sequence $\(E\)$ (length $N = m \cdot n$).  
Then scan left to right; for each element $\(x\)$ in $\(E\)$, keep it in the output list $\(S\)$ **iff** it does **not** already appear in the prefix of $\(S\)$.  
We implement membership check `rsearch(S, x)` by a tree-based reduction:  
$$\text{rsearch}(S, x) = \mathrm{REDUCE OR}\bigl(\mathrm{MAP}(\lambda y. y = x, S)\bigr)$$
which has **span** $\(O(\log |S|)\)$ because the reduction is associative.

---

### Spec / Pseudocode (SPARC style)

```text
PROC multi_dedup_concat(A : ARRAY[m] of LIST[α]) : LIST[α]
VAR
  E : LIST[α];     // concatenation of all A[i]
  S : LIST[α];     // output, preserving first-appearance in A0..A_{m−1}
BEGIN
  E ← []  
  FOR i ← 0 TO m − 1 DO
    E ← E ++ A[i]
  OD

  S ← []
  FOR x IN E DO
    IF NOT rsearch(S, x) THEN
      APPEND(S, x)
    FI
  OD

  RETURN S
END

// Membership via tree-reduction of OR
FUNC rsearch(S : LIST[α], x : α) : BOOL =
  REDUCE_OR( MAP(λ y. (y == x), S) )
// REDUCE_OR is implemented as a binary-tree associative OR reduction → span O(log |S|)

```
## Work / Span Analysis for `multi_dedup_concat`

Let $N = m \cdot n$ be the total number of elements after concatenation.

### Work $T_1$

In the $k$-th outer iteration, `rsearch(S, x)` compares $x$ against each element of the current set $S$ with $|S| = k-1$. This is $\Theta(k)$ equality tests. Summing over all iterations,
$$T_1 = \sum_{k=1}^{N} \Theta(k) = \Theta\big(N^2\big).$$

### Span $T_{\infty}$

The outer loop is inherently sequential, but each membership test is implemented as a **balanced OR-reduction** over $|S|$ predicates, which has depth $\Theta(\log k)$ at the $k$-th step. Therefore,
$$T_{\infty} = \sum_{k=1}^{N} \Theta(\log k) = \Theta\big(N \log N\big).$$

### Conclusion

The concatenate-then-deduplicate algorithm using `rsearch` has **work** $\Theta(N^2)$ and **span** $\Theta(N \log N)$.

Using this method, the span and work are basically the same as 2a), except the N here is m times n as 2a).

Another way of doing this may be first do local deduplication for every list using rsearch, and then concat the deduplicated list to a longer list and deduplicate this list. We introduce new work in local deduplication but also reduce work in the final deduplication of concatenated list.


- **2c.**

I think the reduce operation on sequence in checking if an element in a list is useful here from the rsearch we did before. It basically utilizes the associativity of binary operation True OR False equals True. One True in the sequence results in True as the result. So we can use divide and conquer here to do binary division and aggregation to use more processors to do the job instead of doing this sequentially and not concurrently. But I think in terms of checking duplicated elements, it is difficult to parallelize. Maybe here is some hash method that can do this more quickly. The thing about removing duplicated is that if we divide a longer sequence into smaller sequences and deduplicate every one of the smaller sequences, we still have to deduplicated their aggregated deduplicated lists again. Two lists are both deduplicated do not mean their concatenated list is deduplicated.






- **3b.**


### Work
We have the recurrence
$$T_1(0)=\Theta(1),\qquad T_1(n)=T_1(n-1)+\Theta(1).$$
By unrolling or induction,
$$T_1(n)=\Theta(n).$$
A matching lower bound $T_1(n)=\Omega(n)$ holds because any correct algorithm must read all $n$ inputs, hence
$$T_1(n)=\Theta(n).$$

### Span
Each state $s_i$ depends on $s_{i-1}$, forming a dependency chain of length $n$. Each step contributes $\Theta(1)$ depth, so
$$T_{\infty}(0)=\Theta(1),\qquad T_{\infty}(n)=T_{\infty}(n-1)+\Theta(1),$$
which solves to
$$T_{\infty}(n)=\Theta(n).$$

### Conclusion
The iterative parenthesis matcher using `iterate` has **work** $\Theta(n)$ and **span** $\Theta(n)$.


- **3d.**


## Work / Span Proof for the scan-based parens matcher (map → scan → reduce)

Assume parallel `map` and the work-efficient `scan` with contraction. Let input size be $n$.

### Map phase
Each of the $n$ items is mapped independently with $O(1)$ work.
$$W_{\text{map}}(n)=\Theta(n),\quad S_{\text{map}}(n)=\Theta(1).$$

### Scan phase (work-efficient up-sweep/down-sweep with contraction)
The work satisfies the recurrence $$W_{\text{scan}}(1)=\Theta(1),\quad W_{\text{scan}}(n)=W_{\text{scan}}(n/2)+\Theta(n),$$
so $$W_{\text{scan}}(n)=\Theta(n).$$
The span satisfies $$S_{\text{scan}}(1)=\Theta(1),\quad S_{\text{scan}}(n)=S_{\text{scan}}(n/2)+\Theta(1),$$
so $$S_{\text{scan}}(n)=\Theta(\log n).$$

### Reduce phase (tree reduction to min over prefix sums)
Balanced associative reduction over $n$ items has linear work and logarithmic span:
$$W_{\text{red}}(1)=\Theta(1),\quad W_{\text{red}}(n)=W_{\text{red}}(n/2)+\Theta(n)\Rightarrow W_{\text{red}}(n)=\Theta(n),$$
$$S_{\text{red}}(1)=\Theta(1),\quad S_{\text{red}}(n)=S_{\text{red}}(n/2)+\Theta(1)\Rightarrow S_{\text{red}}(n)=\Theta(\log n).$$

### Whole algorithm (sequential composition of phases)
Add the per-phase costs:
$$W(n)=W_{\text{map}}(n)+W_{\text{scan}}(n)+W_{\text{red}}(n)=\Theta(n),$$
$$S(n)=S_{\text{map}}(n)+S_{\text{scan}}(n)+S_{\text{red}}(n)=\Theta(\log n).$$

**Conclusion.** The scan-based solution achieves work $\Theta(n)$ and span $\Theta(\log n)$.



- **3f.**

### Work
We have the recurrence
$$W(1) = Θ(1),\quad W(n) = 2\,W\bigl(n/2\bigr) + Θ(1).$$
By the Master Theorem (case $\(a = 2, b = 2, f(n)=Θ(1)\)$), since $\(f(n) = O(n^{\log_b a - ε}) = O(n^{1 - ε})\)$, we get
$$W(n) = Θ(n).$$

### Span
We get the recurrence for span:
$$S(1) = Θ(1),\quad S(n) = S(n/2) + Θ(1).$$
Solving it yields
$$S(n) = Θ(\log n).$$

Thus, the divide-and-conquer solution runs with **work** $\(Θ(n)\)$ and **span** $\(Θ(\log n)\)$.


- **4a.**




- **4b.**





- **4c.**




