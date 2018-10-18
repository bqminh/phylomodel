A Simple and Flexible Format for Defining Phylogenetic Models
================================================================

Introduction
------------


Model-based phylogenetic analyses using maximum likelihood (ML) and Bayesian inference (BI) are commonly used nowadays to reconstruct evolutionary histories among organisms. With the advent of phylogenomic (e.g. multi-gene) data, simple models of sequence evolution are likely mis-specified. Such model misspecification may lead to systematic errors in phylogenetic inference. 

Many complex and realistic models of sequence evolution have been introduced, including partition, mixture and covarion models. However, many of them remain by and large not applicable due to difficulty of implementation in widely used phylogenetic software. There exist "complicated" languages such as HyPhy and RevBayes, which allow to specify complex models but require significant amount of learning. 

This proposal aims to develop a specification format allowing users to easily define new phylogenetic models. Such format can be loaded and shared between different platforms and phylogenetic software. Thus, this take advantage of all machineries already available in ML/BI software such as model parameter estimation, tree search, bootstrapping. 

### Design principles

To achieve this aim our design principles for this format are:

* Simple: The format should be simple, human-readable and require little learning. 
* Portable: It should be easily exchangeable between different platforms and software. 
* Sufficiently flexible: It allows to specify complex models without much sacrifice for simplicity. 

In a short-term we would like to achieve the following goals:

* Allowing to define all popular models to date.
* Developing a stand-alone C/C++ libary to process the model file, to be easily integrated into phylogenetic software.
* Supporting this specification in IQ-TREE.

### Existing formats

All existing file formats have pros and cons:

| Name     | Pros             |        Cons |
|----------|---------------------|--------------------------|
| Nexus format | Definition for state space   | None for defining models |
| HyPhy language | Quite powerful | Too complicated |
| NeXML format  | Very flexible | Not so human-readable. None for models yet, but in principle can be extended to include models |
| RevBayes language | Perhaps most flexible for Bayesian inference | Substantial learning effort |


A new model file format
------------------------

Due to aforementioned limitations, we want to propose a new format. Based on the design principles, the following two formats seem suitable, from which the model file format will extend:

* NeXML: an extension of XML (eXtensible Markup Language) and is already supported in various phylogenetic software.
* YAML (Yet Another Markup Language): Very human-readable and yet powerful enough.

We choose to use both formats as they can be converted between each other. For the purpose of illustrations below, we will only use YAML due to its advantage of being human-readable. If a software only supports NeXML, the YAML model file can be converted to this format before usage.

> **NOTE**: The specification is currently under development. Syntax changes may be introduced until further notice.


### Data types

Inspired by the Nexus format, the syntax for defining new data types looks like:

```yaml
---
# definition of datatypes in YAML format
- datatype:     # data type name
  state: [ ]    # vector of ordered states for the alphabet
  missing: [ ]  # vector of states for missing character
  gap: [ - ]    # gap symbols
  equate:       # list of ambiguous characters
  - X: [ ]      # map from a state to list of states
        
# next entry start with '- datatype: XXX'
```

Below is an example for defining DNA data:

```yaml
---
#### definition for DNA data ###
- datatype: DNA
  state: [ A, C, G, T ]
  missing: [ N, "?" ]
  gap: "-"
  equate:
    U: T      # T and U are the same
    R: [A, G] # R is interpreted as A or G
    Y: [C, T]
    W: [A, T]
    S: [G, C]
    M: [A, C]
    K: [G, T]
    B: [C, G, T]
    H: [A, C, T]
    D: [A, G, T]
    V: [A, G, C]
```

See [a specification file for basic data types](datatypes/basicdata.yml) (e.g., DNA, protein, codon).

### Substitution models

Next we define new models with the following basic syntax:

```yaml
# definition of models in YAML format
substitutionModels:
- name:         # model name string
  description:  # model description
  citation:     # citation string
  DOI:          # DOI for the publication (optional)
  forData:      # for which kind of data type? (optional)
  numStates:    # number of states (optional)
  reversible:   # boolean value (yes, true, no, false)

  parameters:   # list of all parameters
  - name:       # vector of parameter names
    range:      # vector of 2 elements for lower and upper bound
    initValue:  # initial values
    type:       # type of parameter (examples below)
  - name: x[1..3] # parameters intepreted as x[1], x[2] and x[3]
    ....

  constraints:  # defining constraints for parameters
    

  rateMatrix:   # specification for rate matrix Q
  - [ q11, q12, q13 ]
  - [ q21, q22, q23 ]
  - [ q31, q32, q33 ]
  stateFrequency: [ f1, f2, f3 ] # specification for state frequency
```

Examples for GTR model:

```yaml
substitutionModels:

### GTR model ###
- name: GTR
  description: “General time reversible”
  citation: “Tavare, 1986”
  reversible: true
  parameters:
  - name: r[1..5]  # rate parameter vector r[1], ..., r[5]
    range: [ 0.0001, 100 ]
    initValue: 1.0
  - name: f[1..4]  # frequency vector f[1],..., f[4]
    type: frequency  # make parameters in range [0,1] and sum to 1.0

  rateMatrix:
  - [         -, r[1]*f[2], r[2]*f[3], r[3]*f[4] ]
  - [ r[1]*f[1],         -, r[4]*f[3], r[5]*f[4] ]
  - [ r[2]*f[1], r[4]*f[2],         -,      f[4] ]
  - [ r[3]*f[1], r[5]*f[2],      f[3],         - ]

  stateFrequency: [ f ]
```

See [sub-folder `substmodels`](substmodels) for definition of substitution models for DNA, protein, binary, morphological and codon.

### Mixture models

For defining mixture models, one should add a `mixture: ` section as follows:

```yaml
substitutionModels:

- name: JC+GTR
  description: "Mixture of JC and GTR"
  parameters:
  - name: w[1, 2]
    type: weight   # type weight implies that w[1]+w[2]=1.0
  mixture:         # mixture components defined in this section
  - fromModel: JC  # 1st component from JC model
    weight: w[1]   # weight parameter
    scale: 1.0     # scaling factor of Q matrix for this component
  - fromModel: GTR # 2nd component from GTR model
    weight: w[2]
    scale: 1.0
```

### Covarion models

For defining covarion models, one should add a `covarion: ` section as the following example:

```yaml
substitutionModels:

- name: COV_GTR
  description: "Covarion model switching between invariant and GTR"
  parameters:
  - name: [ s0, s1 ] # switching rate
    range: [ 0.0001, 100 ]
    
  # definition for a covarion model [ [off,off2on], [on2off,GTR] ]  
  covarion:
  - name: off
    rateMatrix:
    - [ -, 0, 0, 0 ]
    - [ 0, -, 0, 0 ]
    - [ 0, 0, -, 0 ]
    - [ 0, 0, 0, - ]
  - name: off2on
    rateMatrix:
    - [ s0,  0,  0,  0 ]
    - [  0, s0,  0,  0 ]
    - [  0,  0, s0,  0 ]
    - [  0,  0,  0, s0 ]
  - name: on2off
    rateMatrix:
    - [ s1,  0,  0,  0 ]
    - [  0, s1,  0,  0 ]
    - [  0,  0, s1,  0 ]
    - [  0,  0,  0, s1 ]
  - fromModel: GTR  # last part copied from GTR model
```

See [more advanced covarion codon models by Bielawski](substmodels/bielawski.yml).

## Remarks

* How to link parameters between mixture components? Default: linked.
* Binning of states?
* For expression of rate matrices, which operators should be supported: multiplication, addition, subtraction, division. What about brackets?

