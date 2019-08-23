# Google Summer of Code 2019
### Project: Refactor RSK and implement new insertion rules
### Mentor: [Travis Scrimshaw](https://sites.google.com/view/tscrim/home)
### Organization: [SageMath](http://www.sagemath.org/)
<br/>
<br/>

SageMath is an open source mathematics system implemented mostly in Python. It includes a wide area of mathematics, covering topics like algebra, combinatorics, graph theory, number theory, calculus, etc. SageMath uses the [Sage Trac development server](trac.sagemath.org) for development and discussion.

The Trac server consists of tickets with each ticket having a unique ticket number. Like a GitHub issue, these tickets contains all the information related to discussions, commits and codebase changes. 

My work as part of GSoC'19 was in the combinatorics module of SageMath was divided into two parts, refactoring the existing code to a modular design patern and implementing new algorithms related to Robinson–Schensted–Knuth (RSK) correspondence. I have also implemented and modified some combinatorial object classes as part of the project.

Here is the summary of the all work I did in the project (Reference ticket [#27846](https://trac.sagemath.org/ticket/27846)).

## Summary

| Ticket No | Ticket Summary | Status |
|:---------:|:---------------|:------:|
| #27852 | [Refactor structure of RSK class](https://trac.sagemath.org/ticket/27852) | **Closed** |
| #28058 | [Implement dualRSK algorithm](https://trac.sagemath.org/ticket/28058) | **Closed** |
| #25070 | [Implement coRSK algorithm](https://trac.sagemath.org/ticket/25070) | **Closed** |
| #28228 | [Implement Semistandard and Standard super tableaux](https://trac.sagemath.org/ticket/28228) | **Closed** |
| #28229 | [Extend ShiftedPrimedTableau for primed diagonal entries](https://trac.sagemath.org/ticket/28229) | **Needs Review** |
| #24894 | [Implement superRSK algorithm](https://trac.sagemath.org/ticket/24894) | **Needs Review** |
| #28222 | [Implement Shifted Knuth Correspondence](https://trac.sagemath.org/ticket/28222) | **Needs Review** |
<br/>

## Details


### [Ticket #27852: Refactor structure of RSK class](https://trac.sagemath.org/ticket/27852)

This ticket covered the complete fisrt haft of my project. The old code of RSK correspondence have three rules already implemented, namely RSK insertion, Edelman-Greene insertion and Hecke insertion but the implementions were tightly coupled so extending the module was difficuly. So, the first was to switch to an extensible design pattern.

#### Rule based class structure
In a "Rule-based" design each rule is a different class with a parent "Rule" class for common functionalities among different rules.

##### Structure of the parent rule class
```
 Rule
 ├── to_pairs                       # Validate and format input
 ├── forward_rule                   # Iterate over a biword/permutation and insert entries in a tableau 
 ├── backward_rule                  # Iterate over a tableau and insert entries in an array 
 ├── insertion                      # Insert an entry in the tableau according to the bijection algorithm
 ├── reverse_insertion              # Remove an entry from tableau according to the reverse bijection algorithm
 ├── _forward_format_output         # Change tableau to different output format as given by user
 └── _backward_format_output        # Change array to different output format as given by user
```

##### Function call structure
```
 RSK
 └── forward_rule
     ├── to_pairs
     ├── insertion
     └── _forward_format_output
 
 RSK_inverse
 └── backward_rule
     ├── to_pairs
     ├── reverse_insertion
     └── _backward_format_output
```

#### Robinson-Schensted-Knuth insertion
The Robinson-Schensted-Knuth algorithm starts by initializing two semi-standard tableaux P<sub>0</sub> and Q<sub>0</sub> as empty tableaux then while iterating over the input generalized permutation starting at t = 0, take the pair ( j<sub>t</sub>, k<sub>t</sub> ) from the input permutation and set P<sub>t+1</sub> = P<sub>t</sub> <-- k<sub>t</sub>, and define Q<sub>t+1</sub> by adding a new box filled with j<sub>t</sub> to the tableau Q<sub>t</sub> at the same location the row insertion on P<sub>t</sub> ended. When the iteration completes, the pair ( P<sub>t</sub>, Q<sub>t</sub> ) formed is the image of given input under the Robinson-Schensted-Knuth correspondence.

**Reference**: Donald E. Knuth [Permutations, matrices, and generalized Young tableaux](http://projecteuclid.org/euclid.pjm/1102971948). Pacific J. Math. Volume 34, Number 3 (1970), pp. 709-727.

```python
sage: P, Q = RSK([1, 2, 2, 2], [2, 1, 1, 2], insertion=RSK.rules.RSK)
sage: ascii_art((P, Q))
(   1  1  2    1  2  2 )
(   2      ,   2       )
sage: RSK_inverse(P, Q, insertion=RSK.rules.RSK)
[[1, 2, 2, 2], [2, 1, 1, 2]]
```

#### Edelman-Greene insertion
The Edelman-Greene algorithm or the Coxeter-Knuth algorithm is defined for reduced word of a permutation. This algorithm is similar to the standard row insertion except that if k<sub>i</sub> and k<sub>i + 1</sub> both exist in row i, we only set k<sub>i+1</sub> = k<sub>i + 1</sub> and continue.

**Reference**: Paul Edelman, Curtis Greene [Balanced Tableaux](https://doi.org/10.1016/0001-8708(87)90063-6). Advances in Mathematics 63 (1987), pp. 42-99.

```python
sage: P, Q = RSK([1, 3, 2, 5, 4, 1], insertion='EG')
sage: ascii_art((P, Q))
(   1  2  4    1  2  4 )
(   2  5       3  5    )
(   3      ,   6       )
sage: RSK_inverse(P, Q, insertion='EG')
[[1, 2, 3, 4, 5, 6], [1, 3, 2, 5, 4, 1]]
sage: RSK_inverse(*RSK([1, 1, 1, 2], [1, 2, 3, 4], insertion=RSK.rules.EG), insertion=RSK.rules.EG)
[[1, 1, 1, 2], [1, 2, 3, 4]]
```
####  Hecke insertion
The Hecke RSK algorithm returns a pair of an increasing tableau P and a set-valued standard tableau Q. The construction is similar to the classical RSK algorithm, we first insert x into the fisrt row of P, then into the next row of the resulting tableau, and so on, until the construction terminates. The Hecke insertion has two cases in insertion. Suppose we are inserting x into row R of P then:

* **Case 1**: If there exists an entry y in row R such that x < y, then let y be the minimal such entry. We replace this entry y with x if the result is still an increasing tableau.
 
* **Case 2**: No such y exists, then we append x to the end of R. If the result is an increasing tableau we add the box that we have just filled with x in P to the shape of Q, and fill it with the one-element set { i }, and otherwise we find the bottommost box of the column containing the rightmost box of row R, and add i to the entry of Q in this box (this entry is a set, since Q is set-valued).

**Reference**: A. Buch, A. Kresch, M. Shimozono, H. Tamvakis, and A. Yong [Stable Grothendieck polynomials and K-theoretic factor sequences](https://arxiv.org/abs/math/0601514). Math. Ann. 340 Issue 2, (2008), pp. 359--382.

```python
sage: w = [5, 4, 1, 3, 4, 2, 5, 1, 2, 1, 4, 2, 4]
sage: P,Q = RSK(w, insertion=RSK.rules.Hecke)
sage: ascii_art((P, Q))
(   1  2  4  5      (1,)  (4,)     (5,) (7,) )
(   2  4  5         (2,)  (9,) (11, 13)      )
(   3  5            (3,) (12,)               )
(   4               (6,)                     )
(   5         ,  (8, 10)                     )
sage: RSK_inverse(P, Q, insertion=RSK.rules.Hecke, output='list')
[5, 4, 1, 3, 4, 2, 5, 1, 2, 1, 4, 2, 4]
```
