---
title: Relaxing walk for adversarial attacks and future directions for research
filename: adv-attacks.md
---

# Relaxing walk for adversarial attacks and future directions for research
> *Released* 22.04.2024

## Relaxing walk for adversarial attacks

The relaxing walk algorithm is not a worthwhile approach for performing adversarial attacks on the large CNNs that are required for image recognition. In short, the problems are that the sampling is not applicable to CNNs and that the optimization formulations are computationally infeasible.

### The sampling problem

Converting MaxPool layers to MeanPool layers does not seem to improve the convex hull of a CNN. The optimal LP solutions are still zero solutions even with MaxPools replaced, at least with randomly initialized networks. Thus the LP relaxation -based sampling strategy is not suitable for use with CNNs containing MaxPool layers and as a result, the relaxing walk algorithm is reduced to only the local search (pseudo-gradient descent) part. There might exist some other ways to overcome this sampling issue, though.

### The time complexity

The sampling problem aside, the biggest issue with relaxing walk compared to gradient-based algorithms is the computational complexity. Even with small CNNs that produce LPs with tens of thousands of variables and constraints, the LP solutions take multiple seconds meaning that a single adversarial attack will take at least tens of seconds. As a comparison, in the same time AutoAttack can perform adversarial attacks for an entire test dataset of 10 000 examples - with a larger neural network. These larger networks impose LP formulations with millions to tens of millions of variables and constraints - problems that if not totally infeasible take at least tens of minutes to solve. The longer solution times might be acceptable if the LP approach provided additional benefits compared to the gradient-based attacks but this is not the case. Only MIP-based (or similar) approaches are able to conclusively verify the robustness of a test input.

### Pretrained models

Using pretrained CNN models with Julia is difficult. Sites such as the RobustBench repository contain many pretrained networks as Tensorflow models. However, converting those models to Flux models is not trivial. In fact, I could not find any out-of-the-box methods for it. Tensorflow can export models in the ONNX format and there exists ONNX.jl Julia package but the conversion to Flux models is not implemented. Luckily, the package Metalhead.jl containts a few pretrained image recognition models. One of the smallest of them, VGG-11, contains 12 convolutional/MaxPool layers and three fully connected layers, having a memory footprint of approximately 500 MB. Unfortunately, even formulating this network as a JuMP model is not feasible. After 10 minutes of computation, only the first two layers were formulated in the JuMP model and the memory footprint was already 10 GB. Extrapolating, the JuMP model might fit into a system memory of 128 GB. This neural network size also results in computationally infeasible LP formulations. Back-of-the-envelope calculations show that its MIP formulation would have 50 million variables, of which about 15 million binary. Even if possible to solve the LP relaxation in tens of minutes to hours, these times are unacceptable as they would only represent a single step in the relaxing walk algorithm.

## Deep learning models as mathematical optimization problems

In general, explicitly representing deep learning models as mathematical optimization problems is not feasible for now; the discrepancy between the size of the neural networks being used in real-world applications, such as convolutional networks, transformer networks or other types of large neural networks, and the capability of MIP/LP solvers is too large. The deep learning models have training processes which do not seek globally optimal solutions, and furthermore, they are trained using specialized hardware. These reasons make training large models possible but trying to find the global optimum of such large trained models is too difficult - many of the global optimization problems are provably NP-hard. Thus only optimization problems formulated from small neural networks of hundreds of neurons can be solved to global optimality.

### Future directions

Although the MIP formulation approach for optimizing neural networks is limited in its usefulness, I think there are many new interesting avenues for research: 1) finding applications where small networks of hundreds of neurons can be used, 2) finding faster solution methods for MIPs and LPs, and 3) using heuristic methods for optimizing trained neural networks.

#### 1. Applications for small neural networks

The computational feasiblity limit for optimizing trained neural networks globally using the MIP approach seems to be a network size of a few hundred neurons, although it depends on the input and output dimensions. This is a tight limitation but undoubtedly there are many applications where these network sizes are sufficient. Therefore one possible direction for research is finding the applications where small neural networks can be used as surrogates, for example, and their true global optimum has to be found with certainty.

#### 2. Faster solution methods for linear problems

A major reason for the discrepancy between the training performance of deep learning models and the optimization performance of LP and MIP solvers is the type of calculations required and the use of specialized hardware. Gradient descent -type algorithms used in deep learning rely on matrix-vector multiplication which is highly parallelizable and can be performed efficiently on GPUs. In contrast, linear programming solution methods rely on matrix decomposition/factorization and thus cannot be parallelized effectively. However, there has been recent progress in using first-order methods to solve LPs. These methods are parallelizable and can be performed on GPUs. They have shown competitive performance against state-of-the-art solvers such as Gurobi, especially in large problem instances. The advancements in the field are recent and there probably are further improvements to the algorithms and at least a lot of testing to be done.

#### 3. Heuristic optimization methods

Faster solution algorithms for the MIP formulation, such as the relaxing walk, are heuristic, i.e., they cannot guarantee global optimality. Since the biggest benefit of using the MIP formulation is lost this way, it might be worthwhile to try a different approach. A simple sample-and-gradient-descent -strategy might perform exceptionally well for optimizing a trained neural network, even of larger size. Similar methods have shown great success in, e.g., adversarial attacks, since the ability to run them on the GPU makes them very computationally efficient. Using gradient descent -type algorithms in the context of finding the optimum of a trained deep learning model could be another direction for future work.