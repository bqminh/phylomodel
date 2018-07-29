A Simple and Flexible Format for Defining Phylogenetic Models
================================================================

Introduction
------------


Model-based phylogenetic analyses using maximum likelihood (ML) and Bayesian inference (BI) are commonly used nowadays to reconstruct evolutionary histories among organisms. With the advent of phylogenomic (e.g. multi-gene) data, simple models of sequence evolution are likely mis-specified. Such model misspecification may lead to systematic errors in phylogenetic inference. 

Many complex and realistic models of sequence evolution have been introduced, including partition, mixture and covarion models. However, many of them remain by and large not applicable due to difficulty of implementation in widely used phylogenetic software. There exist "complicated" language such as HyPhy and RevBayes, which allow to specify complex models but require significant amount of learning. 

This proposal aims to develop a specification format allowing users to easily define new phylogenetic models. Such format can be loaded and shared between different platforms and phylogenetic software. Thus, Thus, this take advantage of all machineries already available in ML/BI software such as model parameter estimation, tree search, bootstrapping. 

### Design principles

To achieve this aim our design principles for this format are:

* Simple: The format should be simple, human-readable and require little learning. 
* Portable: It should be easily exchangeable between different platform and software. 
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
dataTypes:
- name:         # data type name
  alias: [  ]   # vector of aliases for datatype name
  states: [ ]   # vector of ordered states for the alphabet
  missing: [ ]  # vector of states for missing character
  gap: [ - ]    # gap symbols
  equate:       # list of ambiguous characters
  - from:       # source state
    to: [  ]    # vector of mapped states
        
# next entry start with '- name: XXX'
```

Below is an example for defining DNA data:

```yaml
---
dataTypes:
#### definition for DNA data ###
- name: Nucleotide
  alias: [ NT, DNA, RNA ]
  states: [ A, C, G, T ]
  missing: [ N, ? ]
  gap: [ - ]
  equate:
  - from: U
    to: [ T ]
  - from: R
    to: [ A, G ]
    .....
  - from: V
    to: [ A, G, C ]
```

See [a specification file for basic data types](datatypes/basicdata.yml) (e.g., DNA, protein, codon).

### Substitution models

Next we define new models with the following basic syntax:

```yaml
# definition of models in YAML format
substitutionModels:
- name:         # model name string
  fullName: ""  # full model name string
  citation: ""  # citation string
  numStates:    # number of states
  reversible:   # boolean value (yes, true, no, false)

  parameters:   # list of all parameters
  - name:       # vector of parameter names
    range:      # vector of 2 elements for lower and upper bound
    initValue:  # initial values
    type:       # type of parameter (examples below)
  - name:       # another parameter names
    ....

  rateMatrix:   # specification for rate matrix Q
  - [ q11, q12, q13 ]
  - [ q21, q22, q23 ]
  - [ q31, q32, q33 ]
  stateFrequency: [ f1, f2, f3 ] # specification for state frequency vector
```

Examples for GTR model:

```yaml
substitutionModels:

### GTR model ###
- name: GTR
  fullName: “General time reversible”
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

```yaml
substitutionModels:

- name: JC+GTR
  parameters:
  - name: w[1, 2]
    type: weight # sum to 1.0
  mixture:
  - name: JC
    weight: w[1]
    scale: 1.0
  - name: GTR
    weight: w[2]
    scale: 1.0
```

### Covarion models

```yaml
substitutionModels:

- name: COV_GTR
  parameters:
  - name: [ s0, s1 ] # switching rate
    range: [ 0.0001, 100 ]
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
  - name: GTR
```
