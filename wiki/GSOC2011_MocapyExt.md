---
title: GSOC2011 MocapyExt
permalink: wiki/GSOC2011_MocapyExt
layout: wiki
---

BioPython is a very popular library in Bioinformatics and Computational
Biology. Mocapy++ is a machine learning toolkit for training and using
Bayesian networks. However, Mocapy++ is implemented in C++ and its
monolithic architecture does not provide any mechanism to plug-in new
node types (probability distributions/densities) without prior source
code conversion to C++ and the recompilation of the library. The goal of
the MocapyExt project is to develop an easy-to-use plug-in system for
Mocapy++, which would allow to load and test probability distributions
on the fly. If a user is working in C++, the user could load/embed
arbitrary probability distributions in Mocapy++ and then proceed to use
them in a familiar manner.

Introduction
------------

Mocapy++ is an open source toolkit that is freely available under the
GNU General Public Licence (GPL) from
[SourceForge](http://sourceforge.net/projects/mocapy). Mocapy++ is used
for parameter learning and inference in dynamic Bayesian networks
(DBNs), which encode probabilistic relationships among random variables
in a domain. The library supports a wide spectrum of DBN architectures
and probability distributions, including distributions from directional
statistics; notably, Kent distribution on the sphere and the bivariate
von Mises distribution on the torus, which have proven to be useful in
formulating probabilistic models of protein and RNA structure. This
makes Mocapy++ especially suitable for biomolecular structure
prediction.

The goal of the MocapyExt project is to develop a plugin interface for
Mocapy++. This plugin interface will allow to quickly implement and test
new probability distributions and thus save development time and effort
as it then becomes possible to extend Mocapy++ without undue
modifications and subsequent recompilations.

Author & Mentors
----------------

[Justinas V. Daugmaudis](User%3AJustinas_Daugmaudis "wikilink")
vygis.d@gmail.com

**Mentors**

  
Thomas Hamelryck

Eric Talevich

Work Plan
---------

'''Gain understanding of S-EM and directional statistics '''

-   Review the course material in SCI-B2-1011-Structural bioinformatics;

<!-- -->

-   Study the theory for Markov chain Monte Carlo methods and dynamic
    Bayesian networks;

<!-- -->

-   Supplement the base knowledge by reading relevant papers.

'''Study of Mocapy++ use cases '''

-   Study the Mocapy++ implementation of Stochastic Expectation
    Maximization (S-EM) and parameter learning of Bayesian networks;

<!-- -->

-   Review examples distributed with Mocapy++ library.

'''Study of Mocapy++ internals and code '''

-   Distill the relevant concepts for nodes and densities. Currently,
    there are no such concepts defined within the Mocapy++ framework;

<!-- -->

-   Compare with other examples that implement
    probability distributions.

'''Design of Mocapy++ plugin interface '''

-   Plugin module has to implement the model of
    NodeConcept/DensityConcept;

<!-- -->

-   All the models have to be uniformly accessible through the
    plugin system.

'''Implement Mocapy++ plugin module '''

-   Implement test cases in C++ using the new plugin interface;

<!-- -->

-   Provide Python bindings for the defined interface;

<!-- -->

-   Implement test cases in Python. Verify that this plugin system can
    be used transparently in Python.

'''Experiment with the modular Mocapy++ architecture '''

-   Create sample applications that showcase the advantages of
    modularity that is provided by the plugin system in Mocapy++;

<!-- -->

-   Measure performance.

Schedule
--------

'''Week 1-2 \[May 23th -- June 5th\] '''

Design of interface strategy: it is hard to overstate the importance of
the plugin API feeling "natural" to a Mocapy++ user. So much of the time
will be devoted to the design of the interface.

'''Week 3-5 \[June 6th -- June 19th\] '''

Implement the plugin module.

'''Week 6-7 \[June 20th -- June 30th\] '''

Implement some examples that would showcase Mocapy++ plugin system in
action; for example, how to use externally implemented logistic
distribution.

'''Week 7-8 (Midterm) \[July 1st -- July 10th\] '''

-   Thorough testing and consolidating the features. Writing the
    documentation for each feature.

<!-- -->

-   Mid-term Evaluations. Discuss with the mentor the state of the
    project and adjust the schedule to comply with project's needs.

'''Week 9-10 \[July 11th -- July 24th\] '''

Example applications. Should focus on the type reflection capabilities
of the plugin module.

'''Week 11-12 \[July 25th -- August 7th\] '''

Update the documentation to reflect the new functions. Also document the
examples, do any scrub work of code, etc. Planned 'pencils down' date.

'''Week 13-14 \[August 8th -- August 21st\] '''

Introduce the bindings to the wider audiences, gather the opinions and
the reviews in community.

'''Week 15 \[August 22nd\] '''

The end of the project.

Source Code
-----------

Hosted at [the gSoC11 Mocapy
branch](http://mocapy.svn.sourceforge.net/viewvc/mocapy/branches/gSoC11/)

Progress
--------

### Prototype implementation

#### Embedding the Python interpreter

For a client program to be able to execute Python code, it is necessary
to initialize the scripting environment. It is done by invoking the
Py\_Initialize() function. The interpreter is released by invoking the
Py\_Finalize() function.

It is important to note, however, that any statement between
Py\_Initialize() and Py\_Finalize() calls could throw an exception. If
an exception is thrown, it must be either handled in a try/catch block,
or the program must be terminated. Having this in mind, an previous
example program could be made more robust by enveloping the statements
between Py\_Initialize() and Py\_Finalize() in a try/catch block.
Py\_Finalize() safety could not be overstated. Currently Boost.Python
has several global (or function-static) objects whose existence keeps
reference counts from dropping to zero until the Boost.Python shared
object is unloaded. This can cause a crash because when the reference
counts do go to zero, there is no interpreter. This poses a question if
such a method of initializing the Python interpreter could be considered
to be "easy to use", "safe" or even "non-intrusive", which are the major
MocapyEXT design tenets.

The solution to this issue that is easy to use, safe and non-intrusive
is surprisingly elegant and showcases the RAII (Resource Acquisition Is
Initialization) idiom.

#### The lifetime of the Python interpreter

The Python interpreter is initialized before the entry to the main
function, and is released after the exit from the main function. No
additional end-user effort is required, except to include the headers
that define the necessary Python plug-in wrappers.

#### Compile/link dependencies

The Python interpreter shall be linked to the client program (not to the
Mocapy++ library). The end-user of the MocapyEXT shall be responsible
for additional include and/or library paths to the Python headers and
libraries that are necessary to compile and link the client program
successfully.

MocapyEXT also instantiates its static data members in such a manner
that they are instantiated only once and in the library's compilation
unit. In the C++ standard it is explicitly stated that

"... in particular, the initialization (and any associated side-effects)
of a static data member does not occur unless the static data member is
itself used in a way that requires the definition of the static data
member to exist."\[cpp\_std2003, temp.inst\]

The user shall not need to manage the instantiated static data members.

#### Thread safety guarantees

The MocapyEXT plug-ins have the following thread-safety guarantees:

-   it is safe to call plug-in member functions from multiple threads;

<!-- -->

-   it is safe to perform concurrent read-only accesses of the passed
    parameter containers;

<!-- -->

-   it is safe to perform concurrent read/write accesses, if there is
    only one read or write access to the passed factual parameters at
    a time.

Concurrent mutable access of the passed parameter containers requires
synchronization, for example via reader-writer lock. Mutable access
includes altering values within the container, invoking member functions
that invalidate iterators, moving the container via the move
constructor.

#### The module search path

When a module foo that contains ESS or Densities definitions is being
imported, the embedded Python interpreter searches for a file named
foo.py in the list of directories specified by the environment variable
PYTHONPATH and the current path.

Before proceeding to import a module, sys.path list is explicitly
appended with the current path. A user that relies on scripts being in
some other directory than a client program should list the paths in the
PYTHONPATH environment variable.

Note that because the interpreter also searches in an
installation-dependent default path, it is important that the file
containing ESS or Densities definitions not have the same name as a
standard module.

#### Plugin registration

The minimal interface import example shows the usage of three plug-in
classes: densities\_plugin, ess\_plugin and aggregate\_plugin. The
intended usage pattern for densities\_plugin and ess\_plugin are cases
when types ESS and Densities are implemented in separate files --
one-class-per-file. Each class is loaded in their own interpreter
context. The purpose of aggregate\_plugin is to simplify the management
of the the loaded types and to optimize the resources allocated for the
embedded Python interpreter as aggregate\_plugin creates only one
instance of the embedded interpreter for both types ESS and Densities.

`#include <mocapy/plugin/ess_plugin.hpp>`  
`#include <mocapy/plugin/densities_plugin.hpp>`  
`#include <mocapy/plugin/aggregate_plugin.hpp>`

`int main()`  
`{`  
`  using namespace mocapy::ext;`  
`  densities_plugin dens_pl("plugin_tests", "DensitiesPython");`  
`  ess_plugin ess_pl("plugin_tests", "ESSPython");`  
`  aggregate_plugin aggr_pl("plugin_tests", "ESSPython", "DensitiesPython");`  
`  return 0;`  
`}`

Basically, a user provides a name of a python library, a name of class
and a list of factual arguments necessary for the construction of the
model of the class. The MocapyExt library in turn then returns an
instance of the new class.

`densities_plugin p("test_module", "DensitiesClassName");`  
`densities_adapter n = p.densities(`*`argument` `list`*`);`

The given *argument list* (up to 6 arguments are supported) would
parametrize the initialization of Densities object. Parameters are
forwarded to the Python API and converted using the type conversion
facilities provided by the Boost.Python library. The mechanism is
general and the user can convert any arbitrary type T to its equivalent
in Python.

The parameters are forwarded via const-references. This method accepts
and forwards arbitrary typed arguments, at the cost of always treating
the argument as const. This solution is typically used for constructor
arguments. An esoteric problem with this approach is that it is not
possible to form a const reference to a function type, but this
deficiency, which is being addressed in Core Issue
[295](http://std.dkuug.dk/jtc1/sc22/wg21/docs/cwg_active.html#295),
actually works in our favor, because this prevents ill-intended attempts
to pass a function to the Python interpreter.

Any further invocation of ess and dens member functions automatically
calls a corresponding class method in Python.

#### The lifetime of a class instance

The lifetime of ESS computer and Densities objects (respectively, ess
and dens) may exceed the lifetime of their respective factory plugin(s).
Objects ess and dens preserve reference counted pointers to the Python
interpreter, thus a valid Python interpreter instance exists until the
destruction of objects ess and dens.

#### Creation of a node

This is a simple example that showcases two different methods to
initialize a plugin node. In this particular case, the module
plugin\_tests that has been used in the program implements a dummy
discrete node of fixed length 1, and always returns \[0,\] as sampled
value.

`#include <mocapy/plugin/ess_plugin.hpp>`  
`#include <mocapy/plugin/densities_plugin.hpp>`  
`#include <mocapy/plugin/aggregate_plugin.hpp>`  
`#include <mocapy/plugin/plugin_node.hpp>`

`int main()`  
`{`  
`  using namespace mocapy::ext;`  
`  // Node initialization from two separately loaded modules`  
`  {`  
`    ess_plugin ess("plugin_tests", "ESSPython");`  
`    densities_plugin dens("plugin_tests", "DensitiesPython");`  
`    plugin_node_type node(ess.ess(`*`argument`
`list`*`), dens.densities(`*`argument` `list`*`));`  
`  }`  
`  // Node initialization from a single loaded module`  
`  {`  
`    aggregate_plugin pl("plugin_tests", "ESSPython", "DensitiesPython");`  
`    plugin_node_type node(pl.ess(`*`argument`
`list`*`), pl.densities(`*`argument` `list`*`));`  
`  }`  
`  return 0;`  
`}`

plugin\_node\_type is a typedef of the ChildNode class template. It is
important to reiterate that the lifetime of a node must not exceed the
lifetime of plug-in factories. Upon the destruction of a node, its
member variables ess and densities will try to decrement their reference
count (thus effectively freeing up the resources allocated by the Python
interpreter) and the valid Python interpreter instance must exist.
Failure to ensure the proper destruction order will result in undefined
behavior.

### Measuring performance

It is interesting to measure how much plugin interface costs compared to
the implementation of ESS and Densities computers themselves.