A Simple and Flexible Format for Defining Phylogenetic Models
================================================================

Introduction
------------


Model-based phylogenetic analyses using maximum likelihood (ML) and Bayesian inference (BI) are commonly used nowadays to reconstruct evolutionary histories among organisms. With the advent of phylogenomic (e.g. multi-gene) data, simple models of sequence evolution are likely mis-specified. Such model misspecification may lead to systematic errors in phylogenetic inference. 

Many (cool) complex and realistic models of sequence evolution have been introduced, including partition, mixture and covarion models. However, many remain by and large not applicable due to difficulty of implementation in widely used phylogenetic software. There exist "complicated" language such as HyPhy and RevBayes, which allow to specify complex models but require significant amount of learning. 

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

### Data types


### Substitution models

### Mixture models


### Covarion models

