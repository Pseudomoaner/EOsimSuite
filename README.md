# EOsimSuite
Python package for numerical simulation of EO models.

## Introduction
EO models are a class of dynamical system used to represent ecosystem dynamics. Perhaps the most well-known examples of EO models are Consumer-Resource models, but they can used to explictly represent a broad range of other biological interaction mechanisms such as antibiotic secretion and growth of auxotrophic microbes.

EOsimSuite is a Python package designed to facilitate consistent simulation of EO systems and rapidly translate equivalent biological components between different experimental settings (Batch culture, chemostats, serial transfer systems, range expansions...)

## Contents

This repository contains three folders:
 - **EOsimSuite:** contains the EOsimSuite package.
 - **DemoModels:** contains Jupyter notebooks demonstrating usage of the EOsimSuite package.
 - **ModelLibrary:** contains model specifications for a wide range of different EO systems.

## Specifying EO models

Model specification is a three step process: 1) definition of system dynamics, 2) definition of system parameters and starting conditions, 3) definition of experimental system to be used.

### 1) System dynamics

The differential equations defining EO systems are split into three components: sensitivity functions which define the growth rate of a biological population in the current environment, impact functions which define the effect of these populations on the state of the environment and allogenic functions which define external influences on the environment. Sensitivity and impact function are defined with function definitions, while allogenic functions are specified during selection of the experimental system.

Function definitions follow the following syntax:

```python
def sensFunc(rs,specInd,EOparams):
    EOParam1 = EOparams['EOParam1']
    EOParam2 = EOparams['EOParam2']
    
    return ([expression involving rs, specInd, and EOParams outputting scalar])

def impactFunc(rs,specInd,EOparams):
    EOParam1 = EOparams['EOParam1']
    EOParam2 = EOparams['EOParam2']
    
    impact = np.array([expression involving rs, specInd, and EOParams outputting vector])

    return impact
```

'rs' is a vector containing the current environmental state, specInd is an integer indicating the species index and EOParams contain parameters that influence the biotic components of the system (e.g. resource values and Monod constants). Value sfor EOParams are set in section (2).

In addition, quantities known as the abiotic and biotic accelerations can be calculated from these function definitions. If you wish to use these, specify the sensitivity gradients as:

```python
def gradSensFunc(rs,specInd,EOparams):
    EOParam1 = EOparams['EOParam1']
    EOParam2 = EOparams['EOParam2']

    sensGrad = np.array([expression involving rs, specInd, and EOParams outputting vector])

    return sensGrad
```

Biotic accelerations can be written using the generic definition

```python
def instInteract(rs,alphInd,betInd,EOparams):
    return(np.dot(impactFunc(rs,betInd,EOparams),gradSensFunc(rs,alphInd,EOparams)))
```

or else can be written explicitly based on the algebreic output of this dot product.

Abiotic accelerations depend on the experimental system in use. For example, chemostats can be represented using the generic definition:

```python
def intrinsicGR(rs,alphInd,EOparams,sysParams):
    return(np.dot(gradSensFunc(rs,alphInd,EOparams),sysParams['D']*(sysParams['rIn']-rs)))
```

Additional details on how the experimental systems are specified are presented in step (3).

### 2) Definition of system parameters

### 3) Definition of experimental setup

