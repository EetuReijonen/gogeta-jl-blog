---
title: Neural network robustness verification
filename: nn-robustness.md
---

# Neural network robustness verification
> *Released* 27.03.2024

## What is neural network robustness verification?

### Local robustness verification
* Answers the question of how much a test example (input) has to be changed in order to cause the network to misclassify it.
* A changed test example that is misclassified is called an adversarial example.
* There are multiple methods of training the network in order to achieve more robustness.
* The more robust the network, the more change has to be made to the test example to cause a misclassification.
* Different metrics can be used to measure the difference in the original and the perturbed (changed) test example. 
* Using different metrics as the objective functions to produce adversarial examples leads to different adversarial examples.
* Both a lower bound and an upper bound can be found. The lower bound is the adversarial example found to be furthest from the original test example. The upper bound is a distance from the test example over which no adversarial example can be found. Given enough time, a complete verifier (MIP or SMT -based) will have these two numbers converge.

### Global robustness verification (as in Kabana and Cohen, 2024)
* Tries to find the minimal globally robust bound, i.e, the smallest difference in the activations of the first and the second most active classes, which guarantees that the given perturbation cannot cause a misclassification for any arbitrary input.

## What are the best current methods for robustness verification?

### Local robustness
> robustness relating to perturbations of individual examples - robustness in local neighborhoods

#### MIP-based approaches - able to find certificates of robustness and adversarial examples (upper and lower bounds)
* MIPVerify - Tjeng et al. 2019
    
#### SMT-based approaches - able to find certificates of robustness and adversarial examples (upper and lower bounds)
* Reluplex - Katz et al. 2017
* Marabou - Katz et al. 2019

#### LP-based approaches - only able to find adversarial examples (lower bounds)
* LiPPA - Ismaila et al. 2021

#### Gradient-based approaches - only able to find adversarial examples (lower bounds)
* Fast Gradient Sign Method - Goodfellow et al. 2015
* PGD attack - Madry et al. 2018
* CW attack - Carlini and Wagner 2017

### Global robustness
> local robustness to an arbitrary input far enough from the decision boundaries (difference between classification scores has to be high enough)

#### MIP and gradient-based approaches
* VHAGaR - Kabana and Cohen 2024

#### SMT-based approaches
* Marabou - Katz et al. 2019

## Literature

*Reluplex: An efficient SMT solver for verifying deep neural networks (Katz et al., 2017)*

*Evaluating robustness of neural networks with mixed integer programming (Tjeng et al., 2019)*
* proposes multiple improvements to the standard MIP formulation to achieve better performance when generating adversarial examples
    * tight formulations for non-linearities
    * progressive bound tightening (first interval, then LP-based bounds)
    * neuron pruning (CNNs and maxpools as well)
* shows that their new MIP approach can handle networks several orders of magnitude larger than any previous complete verifier

*Linear Program Powered Attack (Ismaila et al., 2021)*
* proposes a novel attack method which consist of iteratively solving the local hyperplanes (linear regions) with gradient exploration steps in between to find new linear regions (LiPPA)
    * finds better adversarial examples in some cases than other methods and is comp
* shows that the number of linear regions near a test example is much smaller than the theoretical maximum
* shows that the MIP approach is wasting time on branching on infeasible binary combinations
* shows that no approach is utilizing the full information contained in the binary variables to find interesting (and feasible) linear regions

*A Branch and Bound Framework for Stronger Adversarial Attacks of ReLU Networks (Zhang et al., 2022)*

*Verification of Neural Networks' Global Robustness (Kabaha and Cohen, 2024)*
* presents a framework for determining the global robustness of a neural network model
    * MIP formulation at the center
    * pruning search space by computing dependencies
    * hyper-adversarial attack to provide lower bounds and optimization bounds
* shows that their approach is significantly faster and more accurate than existing global robustness verifiers

**and many more...**

## Ideas to implement in Gogeta.jl

### From the papers
* compression (for CNNs as well)
    * pruning maxpool layers
* LiPPA algorithm
* slackless ReLU formulation
* progressive tightening
* norm-based input restriction (in addition to bounds)
* residual layer support

### Novel ideas
* relaxing walk algorithm for adversarial input generation
* sampling and enhanced sampling for adversarial inputs

## Research questions
* What is the best way to sample interesting linear regions?
    * How to use the information contained in the binary variables to choose the best regions?

## Master's thesis idea?
Implementing, adapting and improving relaxing walk for adversarial attacks and comparing it to the current methods, especially for other specific adversarial example generation methods (LP and gradient methods)
* hypothesis: relaxing walk -based attack will be at competitive with state-of-the-art approaches and better than LiPPA for finding adversarial examples