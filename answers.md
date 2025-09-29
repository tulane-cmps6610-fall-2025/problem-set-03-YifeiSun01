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








- **2b.**







- **2c.**










- **3b.**




- **3d.**





- **3f.**




- **4a.**




- **4b.**





- **4c.**




