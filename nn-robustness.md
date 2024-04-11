---
title: Neural network robustness verification
filename: nn-robustness.md
---

# Neural network robustness verification
> *Released* 27.03.2024, *edited* 11.4.2024

## What is neural network robustness verification?

### Local robustness verification
* Answers the question of how much a test example (input) has to be changed in order to cause the network to misclassify it.
* A changed test example that is misclassified is called an adversarial example.
* The more robust the network, the more change has to be made to the test example to cause a misclassification.
* There are multiple methods of training the network in order to achieve more robustness.
* Different metrics can be used to measure the difference in the original and the perturbed (changed) test example. 
* Using different metrics as the objective functions to produce adversarial examples leads to different adversarial examples.
* Both a lower bound and an upper bound can be found. The lower bound is the adversarial example found to be furthest from the original test example. The upper bound is a distance from the test example over which no adversarial example can be found. Given enough time, a complete verifier (MIP or SMT -based) will have these two numbers converge.
* Methods for finding adversarial examples are often called attacks. White-box attacks have access to the underlying model while black-box attacks do not. Because of this, hiding the model behind an API will not protect from black-box attacks.

### Global robustness verification (as in Kabana and Cohen, 2024)
* Tries to find the minimal globally robust bound, i.e, the smallest difference in the activations of the first and the second most active classes which guarantees that the given perturbation cannot cause a misclassification for any arbitrary input.

## What are the best current methods for robustness verification?

### Local robustness
> robustness relating to perturbations of individual examples - robustness in local neighborhoods

#### White-box attacks

##### MIP-based approaches - able to find certificates of robustness and adversarial examples (upper and lower bounds)
* MIPVerify - Tjeng et al. 2019
* BaB attack - Zhang et al. 2022 (only able to find adversarial examples efficiently)
    
##### SMT-based approaches - able to find certificates of robustness and adversarial examples (upper and lower bounds)
* Reluplex - Katz et al. 2017
* Marabou - Katz et al. 2019

##### LP-based approaches - only able to find adversarial examples (lower bounds)
* rLR-QP - Croce and Hein, 2018
* LiPPA - Ismaila et al. 2021

##### Gradient-based approaches - only able to find adversarial examples (lower bounds)
* Fast Gradient Sign Method - Goodfellow et al. 2015
* CW attack - Carlini and Wagner 2017
* PGD attack - Madry et al. 2018
* FAB attack - Croce and Hein 2020
* AutoAttack - Croce and Hein 2020

#### Black-box attacks
* Bandits
* SimBA
* SignHunter
* Square attack - Andriushchenko et al. 2020

### Global robustness
> local robustness to an arbitrary input far enough from the decision boundaries (difference between classification scores has to be high enough)

#### MIP and gradient-based approaches
* VHAGaR - Kabana and Cohen 2024

#### SMT-based approaches
* Marabou - Katz et al. 2019

## Literature

*The Fourth International Verification of Neural Networks Competition (VNN-COMP 2023): Summary and Results (Brix et al., 2023)*

*Reluplex: An efficient SMT solver for verifying deep neural networks (Katz et al., 2017)*
* Reluplex verifier

*A randomized gradient-free attack on ReLU networks. (Croce and Hein, 2018)*
* rLR-QP

*Reliable Evaluation of Adversarial Robustness with an Ensemble of Diverse Parameter-free Attacks (Croce and Hein, 2020)*
* AutoAttack

*Minimally distorted Adversarial Examples with a Fast Adaptive Boundary Attack (Croce and Hein, 2020)*
* FAB attack

*Beta-CROWN: Efficient Bound Propagation with Per-neuron Split Constraints for Neural Network Robustness Verification (Wang et al., 2021)*
* GPU bound tightening
* complete verification

*Towards Evaluating the Robustness of Neural Networks (Carlini and Wagner, 2017)*
* C&W attack

*EXPLAINING AND HARNESSING ADVERSARIAL EXAMPLES (Goodfellow et al. 2015)*
* FGSM attack

*The Marabou Framework for Verification and Analysis of Deep Neural Networks (Katz et al. 2019)*
* Marabou verifier

*Square Attack: a query-efficient black-box adversarial attack via random search (Andriushchenko et al. 2020)*
* Sqaure attack

*Towards Deep Learning Models Resistant to Adversarial Attacks (Madry et al. 2019)*
* PGD attack

*Evaluating robustness of neural networks with mixed integer programming (Tjeng et al., 2019)*
* MIPVerify
* proposes multiple improvements to the standard MIP formulation to achieve better performance when generating adversarial examples
    * tight formulations for non-linearities
    * progressive bound tightening (first interval, then LP-based bounds)
    * neuron pruning (CNNs and maxpools as well)
* shows that their new MIP approach can handle networks several orders of magnitude larger than any previous complete verifier

*Linear Program Powered Attack (Ismaila et al., 2021)*
* LiPPA attack
* proposes a novel attack method which consist of iteratively solving the local hyperplanes (linear regions) with gradient exploration steps in between to find new linear regions (LiPPA)
    * finds better adversarial examples in some cases than other methods and is comp
* shows that the number of linear regions near a test example is much smaller than the theoretical maximum
* shows that the MIP approach is wasting time on branching on infeasible binary combinations
* shows that no approach is utilizing the full information contained in the binary variables to find interesting (and feasible) linear regions

*A Branch and Bound Framework for Stronger Adversarial Attacks of ReLU Networks (Zhang et al., 2022)*
* BaB attack

*Verification of Neural Networks' Global Robustness (Kabaha and Cohen, 2024)*
* VhAGAR
* presents a framework for determining the global robustness of a neural network model
    * MIP formulation at the core
    * pruning search space by computing dependencies
    * hyper-adversarial attack to provide lower bounds and optimization bounds
* shows that their approach is significantly faster and more accurate than existing global robustness verifiers
* this method is very time-consuming, sometimes taking multiple hours for networks with only at most thousands of neurons
* results show that class confidences often must be (exceptionally?) high in order to guarantee robustness to a perturbation

**and many more...**

### Summary

The field of neural network robustness verification is mostly focused on local robustness, i.e., finding adversarial examples, as this is the computationally most feasible approach.  Multiple different attacks have been proposed for finding adversarial examples, many gradient-based and some LP-based. The types of attacks can be further classified into white-box and black-box attacks. White-box attacks have access to the neural network model structure while black-box attacks only see the model output or the scores for each of the categories. The robustness of neural network models is evaluated based on the amount of test inputs for which an adversarial example can be found using these attacks. This approach is problematic, however. The attacks that aim to find adversarial examples usually have no guarantee of finding an adversarial example even if one exists. Therefore, with a better attack, the robust accuracy of a classifier can dramatically decrease if adversarial examples are found for more of the test set inputs.

Methods for certifying the robustness of individual inputs also exist. They are guaranteed to find an adversarial example within a given perturbation if such an example exists. These methods are based on solving MIP of SMT problems, usually with a general solver, but heuristic improvement algorithms have also been proposed. The problem with certified robustness is that the MIP or SMT formulations are computationally expensive for large network sizes and therefore limited in their practicability. Nevertheless, the solution time can be reduced in some cases as, at least with MIP formulations, the solution can be prematurely terminated and both a lower and an upper bound can be obtained. The lower bound is the adversarial example with the largest perturbation and the upper bound is the perturbation size larger than which no adversarial example can be found.

Finally, in addition to local robustness, some research has also been focused on global robustness verification. The problem of global robustness is much harder than that of local robustness, and even defining global robustness mathematically can be challenging. Still, MIP and SMT -based methods for tackling this problem have been proposed. The problem with them, however, is the same as with certified robustness but even worse. Due to computational high cost, only very small neural networks can be globally verified. Also, the classifications which can be considered robust seem to have to be exceptionally certain, i.e., have a large difference between the scores of the highest predicted label and the second highest label.

### Current state-of-the-art methods

The international verification of neural networks competition (VNN-COMP) has been held annually since 2020 to fairly and objectively compare neural network verification methods. The last year's competition (2023) determined **α,β-CROWN** to be the winner. This verifier was the fastest and able to verify the largest number of instances in most of the benchmarks used. It is based in GPU bound propagation, a branch-and-bound algorithm and mixed-integer programming. 

The VNN-COMP is focused on complete neural network verifiers, i.e., algorithms that should be able to find certificates of robustness. The problem of finding just adversarial examples is easier, though. I'm not aware whether there is a clear winner among the methods for adversarial attacks. In terms of white-box attacks **AutoAttack** seems to perform the best, at least at the time of the publication its paper (2020). However, its performance against **LiPPA**, for instance, is unknown. **Square attack** is the best black-box attack I'm currently aware of but its performance compared to LiPPA and AutoAttack is unknown as well.

For global robustness verification, **VhAGAR** seems to perform the best but its capabilities are underwhelming when comparing the size of the networks it is able to verify compared to the size of networks used in real-world applications.

## Ideas to implement in Gogeta.jl

### From the papers
* compression (for CNNs as well) - Tjeng et al. 2019
    * pruning maxpool layers
* slackless ReLU formulation - Tjeng et al. 2019
* progressive tightening - Tjeng et al. 2019
* norm-based input restriction (in addition to bounds)
* residual layer support
* GPU-based bound tightening - Zhang et al. 2022
* MIP decomposition - Zhang et al. 2022

### Novel ideas
* relaxing walk algorithm for adversarial input generation
* sampling and enhanced sampling for adversarial inputs
* lossless compression (linear dependencies) in CNNs for adversarial attacks

## Research questions
* What is the best way to sample interesting linear regions?
    * How to use the information contained in the binary variables to choose the best regions?

* How to train neural networks to see like humans?
    * adversarial examples show that inperceptible changes (to the human eye) can cause misclassification by the network

## Master's thesis idea?
Implementing, adapting and improving relaxing walk for adversarial attacks and comparing it to the current methods, especially for other specific adversarial example generation methods (LP and gradient methods)
* hypothesis: relaxing walk -based attack will be at competitive with state-of-the-art approaches and better than LiPPA for finding adversarial examples

## Relevant links

* [Verification of neural networks competition (VNN-COMP)](https://sites.google.com/view/vnn2023)
* [Visualizing neural networks (TensorSpace.js)](https://tensorspace.org)
* [ResNet-50 model](https://huggingface.co/microsoft/resnet-50)