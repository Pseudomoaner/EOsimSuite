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

'rs' is a vector containing the current environmental state, specInd is an integer indicating the species index and EOparams contain parameters that influence the biotic components of the system (e.g. resource values and Monod constants). Value sfor EOparams are set in section (2).

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

Model parameters are split into specifications of biotic (EOparams) and abiotic (sysParams) components. Both are defined as dictionaries, with human-readable name:value pairings.

EOparams can be named arbitrarily, provided that assigned names match those referred to in the impact and sensitivity function definitions. They can also consist of numpy arrays as well as scalars. For example, for a simple one-species model of microbial growth with Monod parameters v (the maximal growth rate) and K (the half-maximal velocity constant) the dictionary could be constructed as:

```python
v = np.array([2.1])
K = np.array([0.3])
EOparams = {'v':v, 'K':K}
```

More complex models can be built by sampling random collections of parameters:

```python
#Functions that generate random C and D matrices
def genRandMats(noSpec,noRes):
    Cmax = 0.25 #Maximal value of the C matrix
    Cmin = 0 #Minimum resource preference
    Dmax = 0.18 #Maximal value of the D matrix
    Dmin = 0 #Minimum leakage from a given species (fraction of carbon lost to other metabolites)
    
    C = Cmin + np.random.rand(noSpec,noRes)*(Cmax-Cmin)
    for alphInd in range(min(noSpec,noRes)):
        C[alphInd,alphInd] = 1

    Dar = Dmin + np.random.rand(noSpec,noRes,noRes)*(Dmax-Dmin)
    
    for i in range(noSpec):
        np.fill_diagonal(Dar[i,:,:],0) #Ensures resources aren't leaked from their own pool
            
    return [C,Dar]
EOparams = {'C':C,'Dar':Dar}
```

sysParams should be defined using consistent syntax depending on the expermental setup (see next section). A list of variable names that should be used is provided below for reference, along with a list of system types for which each needs to be defined.

- **rIn**: The input values of environmental factors for open systems (e.g. chemical concentrations of input media). Required for: Chemostat, colony
- **D**: The system dilution rate. Required for: Chemostat, colony, serial transfer
- **dX**: Spatial separation between volume elements in 1D spatial model. Required for: Colony
- **specDiffConst**: Spatial diffusion constant for populations. Required for: Colony
- **resDiffConst**: Spatial diffusion constant for environmental factors. Required for: Colony
- **specNoise**: Standard deviation of normal distribution from which initial populations are sampled from. Required for: Colony
- **resNoise**: Standard deviation of normal distribution from which initial environmental factors are sampled from. Required for: Colony
- **transT**: Time between transfers. Required for: Serial transfer
- **Ks**: Carrying capacities for biotic resources. Required for: MacArthur
- **vs**: Maximal growth rates of biotic growth rates. Required for: MacArthur
- **ts**: Mortality rates of populations. Required for: MacArthur

### 3) Definition of experimental setup

A standard EOSimSuite model is run with the following syntax:

```python
thisSim = EOsimSuite(impactFunc,sensFunc,EOparams,sysParams,x0,r0)
thisSim.set_timeSettings(tSpan = [0,30], dt=0.01)
thisSim.simEOmodel('Chemostat')
thisSim.plotSolutionTimecourse()
```

The first line instantiates an EOSimSuite model with the functions and parameters defined in steps (1) and (2), as well as initial conditions (x0 for initial population abundances and r0 for environmental factors). the second line sets up the time domain over which the numerical integration should run. The third runs the numerical integration scheme with the specified experimental paradigm. Finally the fourth line generates a basic plot of the system dynamics with populations in blue and environmental factors in red.

The method set_dilSettings can be used to ensure that chemostat and serial transfer systems with equivalent long-term dilution rates can be directly compared: 

```python
thisSim.set_dilSettings(D=0.05,transT=0.2)

thisSim.simEOmodel('Chemostat')
thisSim.plotSolutionTimecourse()

thisSim.simEOmodel('SerialTransfer')
thisSim.plotSolutionTimecourse()
```

## Accessing the results

The results of the numerical integration can be accessed as attributes of the EOsimSuite object. Population dynamics are stored in specTimeseries, while environmental dynamics are stored in resTimeseries.

EOsimSuite also contains in-built methods to facilitate analysis of equilibrium states of models. Assuming that an equilibrium is reached, abiotic acceleration vectors can be assembled as:

```python
rStar = chemSim.resTimeseries[:,-1]
thisSim.set_intrinsicGR(intrinsicGR)
abioticAccel = thisSim.assembleGRvec(rStar)
```

while biotic acceleration matrices can be assembled as:

```python
rStar = chemSim.resTimeseries[:,-1]
thisSim.set_instInteract(instInteract)
bioticAccel = thisSim.assembleIntMat(rStar)
```

These can be treated as effective gLV parameters under certain scenarios. For more information, please see [Meacock & Mitri 2026](https://doi.org/10.32942/X2MD57).
