# Google Summer of Code 2019
### Project: Refactor RSK and implement new insertion rules
### Mentor: [Travis Scrimshaw](https://people.smp.uq.edu.au/TravisScrimshaw/)
### Organization: [SageMath](http://www.sagemath.org/)
<br/>
<br/>

SageMath is an open-source mathematical software with many components implemented in Python. It can do computations in diverse fields of mathematics, like algebra, combinatorics, graph theory, number theory, calculus, etc. SageMath uses the [Sage Trac development server](trac.sagemath.org) for development and discussion.

The trac server consists of tickets, like a Github issue, that contains all the information related to discussions, commits, and codebase changes.

As part of GSoC'19, I have worked on SageMath's combinatorics module. The project was broadly divided into three parts:

* Refactoring the existing code to a modular design pattern.
* Implementing new algorithms related to Robinson–Schensted–Knuth (RSK) correspondence.
* Implementing/modifying combinatorial object classes to support new algorithms.

The RSK correspondence provides a one-to-one mapping between matrices and pairs of objects called semistandard Young tableaux, which are 2D arrays with certain conditions. So, for each matrix, the RSK algorithm gives a unique pair of semistandard Young tableaux. The RSK algorithm constructs the first tableau by successively inserting the values from the matrix according to a specific rule called insertion rule, while the second tableau records the evolution of the first tableau's shape during the construction. An insertion rule defines the procedure of inserting a new element into a Young tableau. Different insertion rules result in different correspondences, and many of these have been thoroughly studied in combinatorics.

Here is the summary of the all work I did in the project (reference ticket [#27846](https://trac.sagemath.org/ticket/27846)).

## Summary

| Ticket No | Ticket Summary | Status |
|:---------:|:---------------|:------:|
| [#27852](https://trac.sagemath.org/ticket/27852) | [Refactor structure of RSK class](#ticket-27852-refactor-structure-of-rsk-class) | **Closed** |
| [#25070](https://trac.sagemath.org/ticket/25070) | [Implement dualRSK and coRSK algorithm](#ticket-25070-implement-dualrsk-and-corsk-algorithm) | **Closed** |
| [#28228](https://trac.sagemath.org/ticket/28228) | [Implement Semistandard and Standard super tableaux](#ticket-28228-implement-semistandard-and-standard-super-tableaux) | **Closed** |
| [#24894](https://trac.sagemath.org/ticket/24894) | [Implement super RSK algorithm](#ticket-24894-implement-super-rsk-algorithm) | **Needs Review** |
| [#28229](https://trac.sagemath.org/ticket/28229) | [Extend ShiftedPrimedTableau for primed diagonal entries](#ticket-28229-extend-shiftedprimedtableau-for-primed-diagonal-entries) | **Positive Review** |
| [#28222](https://trac.sagemath.org/ticket/28222) | [Implement Shifted Knuth Correspondence](#ticket-28222-implement-shifted-knuth-correspondence) | **Needs Review** |
<br/>


## Status
The implementation of all the above-listed algorithms and combinatorial objects is complete and can be used directly without further development. Many more correspondences extending RSK can be found in the mathematical literature and can be implemented by following the rule design pattern developed in this project.


## Details
### [Ticket #27852: Refactor structure of RSK class](https://trac.sagemath.org/ticket/27852)
The earlier implementation of the RSK correspondence had three insertion rules already implemented, namely RSK insertion, Edelman-Greene insertion, and Hecke insertion. The old implementation had nest functional calls and was difficult to extend. So, the primary goal of this project was to switch to an extensible design pattern, which was covered in this ticket.

#### Rule based class structure
In a rule based design, each rule is a different class with a parent "Rule" class for common functionalities among different rules. This design can be useful in research since anyone can implement his/her insertion algorithm by simply implementing their insertion() and reverse_insertion() methods.

##### Structure of the parent rule class
```
 Rule
 ├── to_pairs                       # Validate and format input
 ├── forward_rule                   # Iterate over a biword/permutation and insert entries in a tableau 
 ├── backward_rule                  # Iterate over a tableau and insert entries in an array 
 ├── insertion                      # Insert an entry in the tableau according to the bijection algorithm
 ├── reverse_insertion              # Remove an entry from the tableau according to the reverse bijection algorithm
 ├── _forward_format_output         # Change tableau to different output format as given by the user
 └── _backward_format_output        # Change array to different output format as given by the user
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
The Robinson-Schensted-Knuth algorithm is an iterative procedure and starts with two empty semi-standard tableaux P<sub>0</sub> and Q<sub>0</sub>. Starting at t = 0, the procedure iterates over the input pair ( j<sub>t</sub>, k<sub>t</sub> ) and sets P<sub>t+1</sub> = P<sub>t</sub> <-- k<sub>t</sub>, and sets Q<sub>t+1</sub> = Q<sub>t</sub> + a new box containing j<sub>t</sub>. The new box is added in Q<sub>t</sub> at the same location where the row insertion on P<sub>t</sub> ended. The final pair ( P<sub>t</sub>, Q<sub>t</sub> ) formed after iteration, is the image of given input under the Robinson-Schensted-Knuth correspondence.

**Reference**: Donald E. Knuth. [Permutations, matrices, and generalized Young tableaux](http://projecteuclid.org/euclid.pjm/1102971948). Pacific J. Math. Volume 34, Number 3 (1970), pp. 709-727.

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

**Reference**: Paul Edelman, Curtis Greene. [Balanced Tableaux](https://doi.org/10.1016/0001-8708(87)90063-6). Advances in Mathematics 63 (1987), pp. 42-99.

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
The Hecke RSK algorithm returns a pair of an increasing tableau P and a set-valued standard tableau Q. The construction is similar to the classical RSK algorithm: we first insert x into the first row of P, then into the next row of the resulting tableau, and so on, until the construction terminates. There are two cases in Hecke insertion procedure. Suppose we are inserting x into row R of P then:

* **Case 1**: If there exists an entry y in row R such that x < y, then let y be the minimal such entry. We replace this entry y with x if the result is still an increasing tableau.
 
* **Case 2**: No such y exists, then we append x to the end of R. If the result is an increasing tableau we add the box that we have just filled with x in P to the shape of Q and fill it with the one-element set { i }, and otherwise we find the bottommost box of the column containing the rightmost box of row R, and add i to the entry of Q in this box (this entry is a set, since Q is set-valued).

**Reference**: A. Buch, A. Kresch, M. Shimozono, H. Tamvakis, and A. Yong. [Stable Grothendieck polynomials and K-theoretic factor sequences](https://arxiv.org/abs/math/0601514). Math. Ann. 340 Issue 2, (2008), pp. 359--382.

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


### [Ticket #25070: Implement dualRSK and coRSK algorithm](https://trac.sagemath.org/ticket/25070)
Dual RSK algorithm is a bijection between strict biwords, or a {0, 1} - matrices, and pairs of same shaped tableaux ( P, Q ).

A strict biword is a pair of two lists [a<sub>1</sub>, a<sub>2</sub>, ..., a<sub>n</sub>] and [b<sub>1</sub>, b<sub>2</sub>, ..., b<sub>n</sub>] that satisfy the strict inequalities (a<sub>1</sub>, b<sub>1</sub>) < (a<sub>2</sub>, b<sub>2</sub>) < ... < (a<sub>n</sub>, b<sub>n</sub>) in lexicographic order.

Under dual RSK insertion, when a number k<sub>i</sub> is inserted into the i<sup>th</sup> row of P, it bumps out the first integer greater or equal to k<sub>i</sub> in this row.

**Reference**: 
* Richard P. Stanley. RSK<sup>*</sup>, Chapter-7, *Enumerative Combinatorics*, Volume 2. Cambridge University Press, 2001.
* Darij Grinberg, Victor Reiner. [Hopf Algebras In Combinatorics](https://arxiv.org/src/1409.8356v5/anc/HopfComb-v73-with-solutions.pdf) the third solution to Exercise 2.7.12(a)

```python
sage: P,Q = RSK(Word([2,3,3,2,1,3,2,3]), insertion=RSK.rules.dualRSK)
sage: ascii_art((P, Q))
(   1  2  3    1  2  8 )
(   2  3       3  6    )
(   2  3       4  7    )
(   3      ,   5       )
sage: RSK(Word([1,1,3,4,4]), [1,4,2,1,3], insertion=RSK.rules.dualRSK)
[[[1, 2, 3], [1], [4]], [[1, 1, 4], [3], [4]]]
sage: P = Tableau([[1, 2, 5], [3], [4]])
sage: Q = Tableau([[1, 2, 3], [4], [5]])
sage: RSK_inverse(P, Q, insertion=RSK.rules.dualRSK)
[[1, 2, 3, 4, 5], [1, 4, 5, 3, 2]]
sage: RSK_inverse(P, Q, 'word', insertion=RSK.rules.dualRSK)
word: 14532
sage: RSK_inverse(P, Q, 'matrix', insertion=RSK.rules.dualRSK)
[1 0 0 0 0]
[0 0 0 1 0]
[0 0 0 0 1]
[0 0 1 0 0]
[0 1 0 0 0]
```

CoRSK insertion gives a bijection between strict cobiwords and pairs of same shaped tableaux (P, Q) where P is a semistandard tableau while Q is a row strict tableau. CoRSK uses the classical RSK algorithm for insertion.

A strict cobiword is a pair of two lists [a<sub>1</sub>, a<sub>2</sub>, ..., a<sub>n</sub>] and [b<sub>1</sub>, b<sub>2</sub>, ..., b<sub>n</sub>] that satisfy the strict inequalities (a<sub>1</sub>, b<sub>1</sub>) < (a<sub>2</sub>, b<sub>2</sub>) < ... < (a<sub>n</sub>, b<sub>n</sub>), where the binary relation < on pairs of integers is defined by having (u<sub>1</sub>, v<sub>1</sub>) < (u<sub>2</sub>, v<sub>2</sub>) if and only if either u<sub>1</sub> < u<sub>2</sub> or (u<sub>1</sub> = u<sub>2</sub> and v<sub>1</sub> > v<sub>2</sub>).

 **Reference**:
 * William Fulton. [Young Tableaux](https://www.cambridge.org/core/books/young-tableaux/A7570B10D82AE7233E25E5D6F70A07B6) Section A.4. Cambridge University Press, 1997.
 * Darij Grinberg, Victor Reiner. [Hopf Algebras In Combinatorics](https://arxiv.org/src/1409.8356v5/anc/HopfComb-v73-with-solutions.pdf) the second solution to Exercise 2.7.12(a)

```python
sage: P,Q = RSK(Word([2,3,3,2,1,3,2,3]), insertion = RSK.rules.coRSK)
sage: ascii_art((P, Q))
(   1  2  2  3  3    1  2  3  6  8 )
(   2  3             4  7          )
(   3            ,   5             )
sage: M = to_matrix([1, 1, 3, 3, 4], [3, 2, 2, 1, 3]); M
[0 1 1]
[0 0 0]
[1 1 0]
[0 0 1]
sage: P,Q = RSK(M, insertion = RSK.rules.coRSK)
sage: ascii_art((P, Q))
(   1  2  3    1  3  4 )
(   2          1       )
(   3      ,   3       )
sage: RSK_inverse(P, Q, insertion=RSK.rules.coRSK)
[[1, 1, 3, 3, 4], [3, 2, 2, 1, 3]]
sage: RSK_inverse(P, Q, insertion=RSK.rules.coRSK, output='matrix')
[0 1 1]
[0 0 0]
[1 1 0]
[0 0 1]
```


### [Ticket #28228: Implement Semistandard and Standard super tableaux](https://trac.sagemath.org/ticket/28228)
A semistandard super tableau is a tableau whose entries are primed positive integers, which are weakly increasing in rows and down columns. Also, the letters of even parity(unprimed) strictly increase down the columns, and letters of odd parity(primed) strictly increase along the rows.

A standard super tableau is a semistandard super tableau whose entries are in bijection with positive primed integers 1', 1, 2' ... n.

#### Class structure
There are two types of classes in SageMath, element classes, and parent classes. Element class objects are generic elements of a structure, while the parent classes are model for a (mathematical) set of a structures.

*Class StandardSuperTableaux* and *Class SemistandardSuperTableaux* are the parent classes of classes *StandardSuperTableau* and *SemistandardSuperTableau* respectively.

Element classes in this tickets are:
```
 SemistandardSuperTableau           # Inherits Tableau class
 ├── __classcall_private__()        # Similar to __call__() method
 ├── __init__()                     # Initialize class object
 ├── _preprocess()                  # Preprocess object input and changes integers to PrimedEntry
 └── check()                        # Check validity of input, SemistandardSuperTableau possible or not
 
 StandardSuperTableau               # Inherits SemistandardSuperTableau class
 ├── __classcall_private__()        # Similar to __call__() method
 ├── check()                        # Check validity of input, StandardSuperTableau possible or not
 └── is_standard()                  # Check if a given tableau is StandardSuperTableau or not
```

Parent classes in this tickets are:

```
 SemistandardSuperTableaux          # Inherits SemistandardTableaux class
 ├── __classcall_private__()        # Similar to __call__() method
 ├── __contains__()                 # Check if input can be a SemistandardTableau or not
 │
 └── class SemistandardSuperTableaux_all         # Set of all Semi standard super tableaux
     ├── __init__()                              # Initialize class object
     └── _repr_()                                # Represents class object
 
 StandardSuperTableaux               # Inherits SemistandardSuperTableaux class
 ├── __classcall_private__()         # Similar to __call__() method
 ├── __contains__()                  # Check if input can be a SemistandardTableau or not
 │
 ├── class StandardSuperTableaux_all             # Set of all Standard super tableaux
 │   ├── __init__()                              # Initialize class object
 │   └── _repr_()                                # Represents class object
 │
 ├── class StandardSuperTableaux_size            # Set of all Standard super tableaux of a fixed size
 │   ├── __init__()                              # Initialize class object
 │   ├── _repr_()                                # Represents class object
 │   ├── __contains__()                          # Check if input can be a Standard tableau of given size or not
 │   └── cardinality()                           # Count the number of possible Standard tableau of given size
 │
 └── class StandardSuperTableaux_shape           # Set of all Standard super tableaux of a fixed shape
     ├── __init__()                              # Initialize class object
     ├── __contains__()                          # Check if the input can be a Standard tableau of given shape or not
     ├── _repr_()                                # Represents class object
     ├── cardinality()                           # Count the number of possible Standard tableau of a given shape
     └── __iter__()                              # Iterate over the list of all Standard tableau of a given shape
```
**Reference**: Robert Muth. [Super RSK correspondence with symmetry](https://arxiv.org/abs/1711.00420).

Examples of SemistandardSuperTableau:
```python
sage: T = SemistandardSuperTableau([['1p',2,"3'"],[2,3]])
sage: ascii_art(T)
 1'  2 3'
  2  3
sage: T.shape()
[3, 2]
sage: T = sage.combinat.super_tableau.SemistandardSuperTableaux_all()
sage: [[1,2],[2]] in T
True
sage: [[1,3,2]] in T
False
```
Examples of StandardSuperTableau:
```python
sage: T = StandardSuperTableau([["1'",1,"2'",2,"3'"],[3,"4'"]])
sage: ascii_art(T)
 1'  1 2'  2 3'
  3 4'
sage: T.shape()
[5, 2]
sage: T.is_standard()
True
sage: SST = StandardSuperTableaux()
sage: T = SST([["1'",1,"2'",2,"3'"],[3,"4'"]])
sage: T.pp()
 1'  1 2'  2 3'
  3 4'
sage: SST = StandardSuperTableaux(3)  # set of standard super tableaux of size 3
sage: SST.first()
[[1', 1, 2']]
sage: SST.last()
[[1'], [1], [2']]
sage: SST.cardinality()
4
sage: SST.list()
[[[1', 1, 2']], [[1', 2'], [1]], [[1', 1], [2']], [[1'], [1], [2']]]
sage: SST = StandardSuperTableaux([3,2])  # set of standard super tableaux of shape [3, 2]
sage: SST.first()
[[1', 2', 3'], [1, 2]]
sage: SST.last()
[[1', 1, 2'], [2, 3']]
sage: SST.cardinality()
5
sage: SST.list()
[[[1', 2', 3'], [1, 2]],
 [[1', 1, 3'], [2', 2]],
 [[1', 2', 2], [1, 3']],
 [[1', 1, 2], [2', 3']],
 [[1', 1, 2'], [2, 3']]]
```


### [Ticket #24894: Implement super RSK algorithm](https://trac.sagemath.org/ticket/24894)
Super RSK algorithm is a combination of row and column insertion. It provides a bijection between restricted super biwords and pairs of semistandard super tableaux of the same shape. The insertion is like the classical RSK bumping along the rows while a dual RSK like bumping along the columns. Row or column bumping is decided based on either the bumping entry is unprimed or primed, respectively.

**Reference**: Robert Muth. [Super RSK correspondence with symmetry](https://arxiv.org/abs/1711.00420).
```python
sage: P,Q = RSK(["1p", "2p", 2, 2, "3p", "3p", 3, 3], 
....:           ["3p", 1, 2, 3, "3p", "3p", "2p", "1p"], insertion='superRSK')
sage: ascii_art((P, Q))
(  1' 2' 3'  3   1'  2  2 3' )
(   1  2 3'      2'  3  3    )
(  3'         ,  3'          )
sage: RSK_inverse(P, Q, insertion=RSK.rules.superRSK)
[[1', 2', 2, 2, 3', 3', 3, 3], [3', 1, 2, 3, 3', 3', 2', 1']]
sage: P,Q = RSK(["1p", 1, "2p", 2, "3p", "3p", "3p", 3], 
....:           [3, "2p", 3, 2, "3p", "3p", "1p", 2], insertion='superRSK')
sage: ascii_art((P, Q))
(  1'  2  2 3'   1' 2' 3'  3 )
(  2'  3  3       1  2 3'    )
(  3'         ,  3'          )
sage: RSK_inverse(P, Q, insertion=RSK.rules.superRSK)
[[1', 1, 2', 2, 3', 3', 3', 3], [3, 2', 3, 2, 3', 3', 1', 2]]
```


### [Ticket #28229: Extend ShiftedPrimedTableau for primed diagonal entries](https://trac.sagemath.org/ticket/28229)
A shifted primed tableau is a tableau of shifted shape in the alphabet X' = \{1' < 1 < 2' < 2 < ... < n' < n\} such that:
* the entries are weakly increasing along rows and columns; and
* a row can not have two repeated primed elements while, a column cannot have two repeated non-primed elements.

The *ShiftedPrimedTableau* class was already implemented in SageMath, but the old implementation does not allow primed entries in the main diagonal of the shifted tableau. I have added an optional bool parameter *primed_diagonal* to the ShiftedPrimedTableau class and made the required modifications in methods like check(), iter() and other parent classes. If *primed_diagonal* is True, then the shifted tableau can have primed entries in the main diagonal other it can not.

#### Class structure
Element class
```
 ShiftedPrimedTableau                # Inherits ClonableArray class
 ├── __classcall_private__()         # Similar to __call__() method
 ├── __init__()                      # Initialize class object
 ├── _preprocess()                   # Preprocess object input and changes integers to PrimedEntry
 ├── check()                         # Check validity of input, ShiftedPrimedTableau possible or not
 ├── is_standard()                   # Check if entries of tableau are in one-to-one mapping with {1', 1,...,n} 
 ├── __eq__()                        # True if two ShiftedPrimedTableau are equal
 ├── __ne__()                        # True if two ShiftedPrimedTableau are unequal
 ├── __hash__()                      # Return hash of given tableau
 ├── _repr_()                        # String representation of class object
 ├── _repr_list()                    # String representation of object as a list of tuples
 ├── _repr_tab()                     # Return a nested list of strings representing the elements
 ├── _repr_diagram()                 # Return a string representation of object as an array
 ├── _ascii_art_()                   # ASCII representation of object
 ├── _unicode_art_()                 # Unicode representation of object
 ├── _ascii_art_table()              # ASCII/Unicode representation of object
 ├── pp()                            # Pretty print object
 ├── _latex_()                       # Latex code for object
 ├── max_entry()                     # Return minimum unprimed letter greater than all the entries in tableau 
 ├── shape()                         # Return the shape of the underlying partition of given object
 ├── restrict()                      # Restrict the tableau to all the numbers less than or equal to given number
 ├── restriction_outer_shape()       # Return the outer shape of the restriction of the object to given number
 ├── restriction_shape()             # Return the skew shape of the restriction of the object to given number
 ├── to_chain()                      # Return the chain of partition to the given skew object
 └── weight()                        # Return the weight of given object
 ```

Parent class
```
ShiftedPrimedTableaux                # Parent class for ShiftedPrimedTableau
├── __classcall_private__()
├── __init__()
├── _element_constructor_()          # Construct an object from given object as an element, if possible
├── _contains_tableau()
│
├── class ShiftedPrimedTableaux_all                # Set of all Shifted primed tableaux
│   ├── __init__()
│   ├── _repr_()
│   └── __iter__()
│
├── class ShiftedPrimedTableaux_shape              # Set of all Shifted primed tableaux of given shape
│   ├── __classcall_private__()
│   ├── __init__()
│   ├── _repr_()
│   ├── _contains_tableau()
│   ├── __iter__()
│   ├── module_generators()                        # Return the generator of given object as a crystal
│   └── shape()
│
├── class ShiftedPrimedTableaux_weight             # Set of all Shifted primed tableaux of given weight
│   ├── __init__()
│   ├── _repr_()
│   ├── _contains_tableau()
│   └── __iter__()
│
└── class ShiftedPrimedTableaux_weight_shape       # Set of all Shifted primed tableaux of given shape and weight
   ├── __init__()
   ├── _repr_()
   ├── _contains_tableau()
   └── __iter__()
```

**Reference**: Bruce E. Sagan. [Shifted tableaux, Schur Q-functions, and a conjecture of R. Stanley](https://users.math.msu.edu/users/sagan/Papers/Old/sts-pub.pdf). Journal of Combinatorial Theory, Series A Volume 45 (1987), pp. 62-103.

```python
sage: T = ShiftedPrimedTableau([["2p",2,3],["2p","3p"],[2]], skew=[2,1])
sage: unicode_art(T)
┌───┬───┬───┐
│ 1 │ 1 │ 3'│
└───┼───┼───┤
    │ 2'│ 3'│
    └───┴───┘
sage: T.pp()
 .  .  2' 2  3 
    .  2' 3'
       2 
sage: T = ShiftedPrimedTableau([[1,1,2.5],[1.5,2.5]], primed_diagonal=True)
sage: T.pp()

sage: ShiftedPrimedTableaux(shape=[3,2]).first()
[(1, 1, 1), (2, 2)]
sage: SPT = ShiftedPrimedTableaux(weight=(3,3), primed_diagonal=True)
sage: SPT.first()
[(1, 1, 1, 2, 2, 2)]
sage: SPT.last()
[(1', 1, 1, 2'), (2', 2)]
sage: SPT.cardinality()
20
sage: ShiftedPrimedTableaux(weight=(3,3), primed_diagonal=False).cardinality()
6
sage: SPT = ShiftedPrimedTableaux(weight=(1,2), shape=[2,1])
sage: SPT.list()
[[(1, 2'), (2,)]]
sage: SPT = ShiftedPrimedTableaux(weight=(1,2), shape=[2,1],primed_diagonal=True)
sage: SPT.list()
[[(1, 2'), (2,)], [(1, 2'), (2',)], [(1', 2'), (2,)], [(1', 2'), (2',)]]
sage: ShiftedPrimedTableaux([4,2,1], max_entry=3, primed_diagonal=False).cardinality()
24
sage: ShiftedPrimedTableaux([4,2,1], max_entry=3, primed_diagonal=True).cardinality()
192
```


### [Ticket #28222: Implement Shifted Knuth Correspondence](https://trac.sagemath.org/ticket/28222)
Like super RSK algorithm, shifted Knuth insertion is also a combination of row and column insertion. It provides a correspondence between a primed matrix A and a pair (P, Q) of shifted tableaux of the same shape. For each entry, the insertion routine starts with row bumping, and if the bumped element is from the main diagonal, then the bumping procedure switches to column bumping and continues.

**Reference**: Bruce E. Sagan. [Shifted tableaux, Schur Q-functions, and a conjecture of R. Stanley](https://users.math.msu.edu/users/sagan/Papers/Old/sts-pub.pdf). Journal of Combinatorial Theory, Series A Volume 45 (1987), pp. 62-103.
```python
sage: P, Q = RSK([[0,0.5,1.5], [1,0.5,0], [1.5,0,0]], insertion='shiftedKnuth')
sage: unicode_art((P, Q))
⎛ ┌───┬───┬───┬───┬───┐  ┌───┬───┬───┬───┬───┐ ⎞
⎜ │ 1 │ 1 │ 1 │ 3'│ 3 │  │ 1 │ 1 │ 1 │ 2'│ 3'│ ⎟
⎜ └───┼───┼───┼───┴───┘  └───┼───┼───┼───┴───┘ ⎟
⎜     │ 2'│ 2 │              │ 2 │ 3'│         ⎟
⎝     └───┴───┘        ,     └───┴───┘         ⎠
sage: RSK_inverse(P, Q, insertion=RSK.rules.shiftedKnuth)
[[1, 1, 1, 2, 2, 3, 3], [2', 3', 3, 1, 2', 1, 1]]
sage: P, Q = RSK([2,6,5,1,7,4,3,1,2], insertion=RSK.rules.shiftedKnuth)
sage: unicode_art((P, Q))
⎛ ┌───┬───┬───┬───┬───┬───┐  ┌───┬───┬───┬───┬───┬───┐ ⎞
⎜ │ 1 │ 1 │ 2 │ 5 │ 6 │ 7 │  │ 1 │ 2 │ 4'│ 5 │ 7'│ 8'│ ⎟
⎜ └───┼───┼───┼───┴───┴───┘  └───┼───┼───┼───┴───┴───┘ ⎟
⎜     │ 2 │ 3 │                  │ 3 │ 6'│             ⎟
⎜     └───┼───┤                  └───┼───┤             ⎟
⎜         │ 4 │                      │ 9 │             ⎟
⎝         └───┘            ,         └───┘             ⎠
sage: RSK_inverse(P, Q, insertion=RSK.rules.shiftedKnuth)
[[1, 2, 3, 4, 5, 6, 7, 8, 9], [2, 6, 5, 1, 7, 4, 3, 1, 2]]
```
