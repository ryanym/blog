+++
title = "Paper Study - Weight Agnostic Neural Networks"
date = "2021-02-11T23:04:04-05:00"
tags = [
    "paper",
    "deep learning"
]
categories = [
    "machine learning",
]

+++

Interactive version of this paper: [https://weightagnostic.github.io/](https://weightagnostic.github.io/)

## Summary

In a traditional deep learning framework we optimize weights of a fixed network to find the best model. In comparison, for Weight Agnostic Neural Networks (WANNs), we instead search for the best network architecture that perform well over a range of shared weights. The best network architecture usually has strong inductive biases which means importance of weights is minimized. This similar to precocial behaviours in nature.

In order to find the minimal description length WANN, the following steps are performed:

1. Initialize a minimal network and assign a single shared weight parameter to every network connection
2. Evaluate the network on a wide range of this single weight parameter
3. Rank the networks by performance and complexity
4. Create new population by mutate the best networks similar to genetic algorithms

Using this technique, WANNs have great representational power to model any function. Which means with enough training we can eventually find the network with the best inductive bias.

WANNs are well suited for RL tasks where the input dimension is low which allows WANNs to encode the relationships between the inputs and the internal states. The performance is on-par with fixed topology with trained weights.

Ensemble and tuned WANNs also perform well with classification problems compare to single layer network with trained weights. The resulting network still maintain the flexibility to allow weight training.

Individual weight in WANNs can also be further turned as offsets from the best shared weight.

## Critique

1. Authors mentioned WANNs are great for models with small input dimensions in some RL settings. However, the they didn’t address the performance of WANNs for more complexed RL problems.
2. I think the authors did not mention how to vary the network architecture from iteration to iteration. For example, when to add node and when to use a different activation function. What are the parameters to the “neighbour” function.
3. It seems the underlying assumption is that WANNs will find a network architecture with strong inductive bias with enough training. However, the authors did not benchmark how much training is needed to generate such network.