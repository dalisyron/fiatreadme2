% Fiat − Deductive Synthesis of Abstract Data Types in a Proof Assistant<br />
  POPL 2015 Artifact evaluation

**Fiat refines declarative specifications into functional programs
  that can be run with good performance, and it all happens inside Coq
  without modifications. Our artifact includes three examples
  utilizing a library for specifying ADTs with SQL-like operations and
  automatically deriving implementations using tactics. It also
  includes the manual derivation of an implementation of one such ADT
  that automatically caches queries.**

## Conflicts

* Gregory Malecha
* Jean Yang

## Getting started

Thanks for evaluating this artifact! Here is how to get started:

1. Download the [paper](fiat.pdf).

2. Download the artifact: choose one of these three options

    * [OVA virtual machine](fiat-gui.ova) (~ 1.8GB; Linux Mint)
    * [OVA virtual machine, no GUI](fiat-nogui.ova) (~ 1GB; Ubuntu server)
    * [Source code tarball](fiat.tar.gz) (~ 130kB; see the dependencies section below)

    In the virtual machines, use username `fiat` and password
    `fiat`. The source code is in `/home/fiat/popl2015/`

3. Compile the source code (not necessary in the VM):

    `$ make sources`

4. Start exploring the examples!

    The best way to get a feeling of how the library works is to step
    through the semi-automated refinement of the bookstore example
    from the paper found in `examples/BookstoreSemiAutomatic.v`.

    The `examples` folder contains three further fully-automated
    example refinements of ADTs with SQL-like operations:

    + `Bookstore.v` uses the same specification as
      `BookstoreSemiAutomatic.v`, but uses the `plan` tactic found in
      `src/QueryStructure/Refinements/AutoDB.v` to automatically
      derive an implementation.

    + `Stocks.v` and `Weather.v` contain the two additional examples
      from Section 5.2 of the paper, featuring more complex indexing
      schemes, and more diverse aggregation operators.

    `CacheADT/LRUCache.v` demonstrates the manual derivation of the
    least recently used cache discussed in Section 3.1 of the paper.

    `BookstoreCache.v` utilizes the LRUCache derived in
    `CacheADT/LRUCache.v` to build the version of the bookstore
    example from Section 4.7 which caches queries using finite
    differencing.

## Technical details

### Dependencies

* To build the library:          `Coq 8.4pl2`
* To step through the examples:  `GNU Emacs 23.4.1, Proof General 4.3`
* To extract and run OCaml code: `OCaml 3.12.1`

### Compiling

* To build the library: `make sources`
* To extract the bookstore example to OCaml: `make repl` (rather slow)
* To build all the examples: `make examples` (slow; *not* required to
  step through the examples in emacs!). Note that compiling all examples
  requires a significant amount of memory (> 2GB).

## Going further

### Extracting and running OCaml code

Running `make repl` extracts the bookstore example code and produces a
`repl` binary in the `examples/` directory, which supports the following
commands:

    reset
    add_book    [author title isbn]
    place_order [isbn]
    get_titles  [author]
    num_orders  [author]
    benchmark   [nb_authors nb_books nb_orders nb_titles_queries nb_orders_queries]

To replicate the results highlighted in the paper, run `examples/repl`,
and type

    benchmark 1000 10000 100000 100000 100000

### Exploring the code base

#### Top-level directory organization

* `src` - Source files for the ADT synthesis library
* `examples` - Example ADT derivations using the library

#### Detailed sub-directory organization:

##### `src/Computation`

Definitions of computations and refinement that form the core of ADT refinement.

* `Core.v`: core definitions of computation and refinement
* `Monad.v`: proofs that computations satisfy the monad laws
* `SetoidMorphisms.v`: setoid rewriting machinery support for refinement
* `ReflectiveMonad.v`: reflective tactic for simplifying using the monad laws
* `ApplyMonad.v`: applicative tactic for simplifying using the monad laws
* `Refinements/General.v`: core set of refinement facts used for synthesis


##### `src/ADT/`

Definition of abstract data types (ADTs).

* `ADTSig.d`: definition of ADT interfaces (signatures)
* `Core.v`: core definitions of ADT
* `ADTHide.v`: definitions for hiding part of an ADT interface


##### `src/ADTNotation/`

More pleasant notations for ADTs.

* `ilist.v`: definitions of the indexed lists used in ADT notations
* `StringBound.v`: typeclass definition of the bounded strings our
                   notations use for method indices
* `BuildADTSig.v`: notation for ADT signatures
* `BuildADT.v`: notation for ADT definitions
* `BuildADTReplaceMethods.v`: functions for notation-friendly method
                              replacement

##### `src/ADTRefinement/`

Definitions of and machinery for refining ADTs from specification to implementation.

* `Core.v`: definition of ADT refinement
* `SetoidMorphisms.v`: setoid rewriting machinery for ADT refinement
* `GeneralRefinements.v`: core tactics for a 'single step' of ADT
                          refinement through either method or constructor refinement
* `BuildADTSetoidMorphims.v`: Notation-friendly proofs of switching
                              representation and method refinement
* `GeneralBuildADTRefinements.v`: Notation-friendly tactics for refining
                                  a single ADT method or constructor


##### `src/ADTRefinement/Refinements/`

Definitions and tactics for more specialized refinements.

* `HoneRepresentation.v`: switching the representation type of an ADT
* `SimplifyRep.v`: simplifying the representation type of an ADT
* `RefineHideADT.v`: refining ADTs with interfaces hidden using the
                     definitions in ADTHide.v
* `ADTCache.v`: adding a cached value to the representation of an ADT
* `ADTRepInv.v`: refining an ADT under a representation invariant
* `DelegateMethods.v`: delegating functionality to another ADT


##### `src/ADTRefinement/BuildADTRefinements/`

Definitions and tactics for notation-friendly versions of some of the specialized refinements in ADTRefinement/Refinements

##### `src/QueryStructure/`

Library for specifying and refining ADTs with SQL-like operations

* `Heading.v`: definitions of signatures for labeled data (headings)
* `Tuple.v`: definitions for labeled data (tuples)
* `Schema.v`: definitions for specifying sets of tuples with data-integrity
              constraints (relation schema)
* `Relation.v`: definitions for sets of tuples (relations)
* `QueryStructureSchema.v`: definitions for specifying sets of relations with
                            cross-relation data-integrity constraints
* `QueryStructure.v`: definitions for reference representation of ADTs
                      with SQL-like operations

##### `src/QueryStructure/QuerySpecs`

Definitions and notations for specifying SQL-like operations on QueryStructures

* `EmptyQSSpecs.v`: empty QueryStructure constructor
* `InsertQSSpecs.v`: QueryStructure insertion operation + basic refinement for
                     for performing data-integrity checks
* `QueryQSSpecs.v`: query operations for QueryStructures


##### `src/QueryStructure/Refinements`

Implementations of QueryStructures

* `ListImplementation`: list-based representation
* `FMapImplementation`: extra lemmas about Coq's Fmaps
* `Bags`: Implementations of the Bag interface described in the paper:
  + `ListBags.v`: List-based bag implementation
  + `TreeBags.v`: FMap-based bag implementation
  + `CachingBags.v`: Bags augmenting existing bags with a caching layer
  + `CountingBags`: Specific kind of ListBags keeping track of their
                    number of elements
  + `BagsOfTuples.v`: Nested TreeBags containing tuples
