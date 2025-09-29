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

Work–Span Proof for DEDUP_SEQ_BY_REDUCE

Setup. At step k (1 ≤ k ≤ n), the seen-list S has size k−1. The algorithm computes B = map(y -> (y == x), S) and then performs a parallel tree reduce with OR and identity False to obtain found.

Fact 1 (cost of a size-m parallel reduce). A balanced tree reduce with an associative operator and identity does Θ(m) work and Θ(log m) span. This is the standard bound for parallel reduce/“fold” and follows from the depth of the reduction tree. 

Per-step costs. At step k, building B does Θ(k) work, and reducing B does Θ(k) work and Θ(log k) span. Hence the step-k membership check has
$$W_k = \Theta(k),\quad S_k = \Theta(\log k).$$

Total work. Summing over k gives
$$W(n) = \sum_{k=1}^{n} W_k = \sum_{k=1}^{n} \Theta(k) = \Theta\!\left(\sum_{k=1}^{n} k\right) = \Theta(n^2).$$

Total span. The outer loop is sequential (each decision depends on all prior steps), so spans add:
$$S(n) = \sum_{k=1}^{n} S_k = \sum_{k=1}^{n} \Theta(\log k) = \Theta\!\big(\log(n!)\big) = \Theta(n \log n).$$
The equality $\sum_{k=1}^{n} \log k = \log(n!)$ is immediate, and Stirling-type bounds give $\log(n!) = n \log n - n + O(\log n)$, hence $\log(n!) = \Theta(n \log n)$. 

Conclusion. Under the standard SPARC-style cost model for map/reduce, DEDUP_SEQ_BY_REDUCE has
$$W(n) = \Theta(n^2),\quad S(n) = \Theta(n \log n).$$
These bounds match the classic results for tree reductions (Θ(m) work, Θ(log m) span) applied to a sequential sequence of membership checks. 







- **2b.**







- **2c.**










- **3b.**




- **3d.**





- **3f.**




- **4a.**




- **4b.**





- **4c.**




