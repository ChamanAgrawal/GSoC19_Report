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

We switched to a "Rule-based" design pattern where each rule is a different class with a parent "Rule" class for common functionalities among different rules.
