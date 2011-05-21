---
title: GSOC2011 Mocapy
permalink: wiki/GSOC2011_Mocapy
layout: wiki
---

Mocapy++ is a machine learning toolkit for training and using Bayesian
networks. It has been used to develop probabilistic models of
biomolecular structures. The goal of this project is to develop a Python
interface to Mocapy++ and integrate it with Biopython. This will allow
the training of a probabilistic model using data extracted from a
database. The integration of Mocapy++ with Biopython will provide a
strong support for the field of protein structure prediction, design and
simulation.

Introduction
------------

Discovering the structure of biomolecules is one of the biggest problems
in biology. Given an amino acid or base sequence, what is the three
dimensional structure? One approach to biomolecular structure prediction
is the construction of probabilistic models. A Bayesian network is a
probabilistic model composed of a set of variables and their joint
probability distribution, represented as a directed acyclic graph. A
dynamic Bayesian network is a Bayesian network that represents sequences
of variables. These sequences can be time-series or sequences of
symbols, such as protein sequences. Directional statistics is concerned
mainly with observations which are unit vectors in the plane or in
three-dimensional space. The sample space is typically a circle or a
sphere. There must be special directional methods which take into
account the structure of the sample spaces. The union of graphical
models and directional statistics allows the development of
probabilistic models of biomolecular structures. Through the use of
dynamic Bayesian networks with directional output it becomes possible to
construct a joint probability distribution over sequence and structure.
Biomolecular structures can be represented in a geometrically natural,
continuous space. Mocapy++ is an open source toolkit for inference and
learning using dynamic Bayesian networks that provides support for
directional statistics. Mocapy++ is excellent for constructing
probabilistic models of biomolecular structures; it has been used to
develop models of protein and RNA structure in atomic detail. Mocapy++
is used in several high-impact publications, and will form the core of
the molecular modeling package Phaistos, which will be released soon.
The goal of this project is to develop a highly useful Python interface
to Mocapy++, and to integrate that interface with the Biopython project.
Through the Bio.PDB module, Biopython provides excellent functionality
for data mining biomolecular structure databases. Integrating Mocapy++
and Biopython will allow training a probabilistic model using data
extracted from a database. Integrating Mocapy++ with Biopython will
create a powerful toolkit for researchers to quickly implement and test
new ideas, try a variety of approaches and refine their methods. It will
provide strong support for the field of biomolecular structure
prediction, design, and simulation.

Author & Mentors
----------------

[Michele Silva](User%3AMchelem "wikilink") michele.silva@gmail.com

**Mentors**

  
Thomas Hamelryck

Eric Talevich

Project's Code
--------------

Hosted at [the gSoC11 Mocapy
branch](http://mocapy.svn.sourceforge.net/viewvc/mocapy/branches/gSoC11/)

Project's Progress
------------------

### Options to create Python bindings to C++ code

#### Swig

There is already an effort to provide bindings for Mocapy++ using Swig.
However, Swig is not the best option if performance is to be required.
The Sage project aims at providing an open source alternative to
Mathematica or Maple. Cython was developed in conjunction with Sage (it
is an independent project, though), thus it is based on Sage's
requirements. They tried Swig, but declined it for performance issues.
According to the [Sage programming
guide](http://sage.math.washington.edu/tmp/sage-2.8.12.alpha0/doc/prog/node36.html)
"The idea was to write code in C++ for SAGE that needed to be fast, then
wrap it in SWIG. This ground to a halt, because the result was not
sufficiently fast. First, there is overhead when writing code in C++ in
the first place. Second, SWIG generates several layers of code between
Python and the code that does the actual work". This was written back in
2004, but it seems things didn't evolve much. The only reason I would
consider Swig is for future including Mocapy++ bindings on BioJava and
BioRuby projects.

#### Boost Python

Boost Python is comprehensive and well accepted by the Python community.
I would go for it for its extensive use and testing. I would decline it
for being hard to debug and having a complicated building system. I
don't think it would be worth including a boost dependency just for the
sake of creating the Python bindings, but since Mocapy++ already depends
on Boost, using it becomes a more attractive option. In my personal
experience, Boost Python is very mature and there are no limitations on
what one can do with it. When it comes to performance, Cython still
overcomes it. Have a look at the [Cython C++ wrapping
benchmarks](http://blog.behnel.de/index.php?p=38) and check the timings
of [Cython against Boost Python](http://www.behnel.de/cycppbench/).
There are also previous [benchmarks comparing Swig and Boost
Python](http://telecom.inescporto.pt/~gjc/pybindgen-benchmarks/).

#### Cython

It is incredibly faster than other options to create python bindings to
C++ code, according to several benchmarks available on the web. Check
the Simple benchmark between Cython and Boost.Python. It is also very
clean and simple, yet powerful. Python's doc on porting extension
modules mentions cython: "If you are writing a new extension module, you
might consider Cython." Cython has now support for efficient interaction
with numpy arrays. it is a young, but developing language and I would
definitely give it a try for its leanness and speed.

Since Boost is well supported and Mocapy++ already relies on it, we
decided to use Boost.Python for the bindings.

### Bindings Prototype

Bindings for a few Mocapy++ features and a couple of examples to find
possible implementation and performance issues.

**Procedure**

-   Implemented the examples hmm\_discrete and
    discrete\_hmm\_with\_prior in Python, assuming the interface
    Mocapy++ already provides.

<!-- -->

-   Implemented the bindings to provide a minimum subset of
    functionality, in order to run the implemented examples.

<!-- -->

-   Compared the performance of C++ and Python versions.

Mocapy++’s interface remained unchanged, so the tests look similar to
the ones in Mocapy/examples.

In the prototype the bindings were all implemented in a single module.
For the actual implementation, we could mirror the src packages
structure, having separated bindings for each package such as discrete,
inference, etc.

It was possible to implement all the functionality required to run the
examples. It was not possible to use the
[vector\_indexing\_suite](http://www.boost.org/doc/libs/1_42_0/libs/python/doc/v2/indexing.html)
when creating bindings for vectors of MDArrays. A few operators (in the
MDArray) must be implemented in order to export indexable C++ containers
to Python.

Two Mocapy++ examples that use discrete nodes were implemented in
Python. There was no problem in exposing Mocapy’s data structures and
algorithms. The performance of the Python version is very close to the
original Mocapy++.

External References
-------------------

[Mocapy++Biopython - Box of
ideas](https://docs.google.com/document/d/1E72Qysp3pMd69hSYfIXJKgBLdeSMbzSol9RYD2rKHlI/edit?hl=pt_BR&authkey=CPmFxK0H)

[Mocapy++ Bindings
Prototype](https://docs.google.com/document/d/1JPkCbvJ9Gk3b6LmQ68__UUt4Yz2P1-c286p6FVEqyXs/edit?hl=pt_BR&authkey=CJTCpZgL)