+++
title = "Paper Study - AutoDropout: Learning Dropout Patterns to Regularize Deep Networks"
date = "2021-02-13T22:04:04-05:00"
tags = [
    "paper",
    "deep learning"
]
categories = [
    "machine learning",
]

+++

Original paper can be found here: https://arxiv.org/abs/2101.01761

## **Summary**

Neural Networks uses regularization methods like Dropout and weight decay to counter over-fitting issues. Recent researches shown that structures of the network can be leveraged to design dropout patterns that achieves better results than randomly dropout.

In practice, dropout patterns are difficult to generalize since they are often task or domain specific.Thus the authors proposed AutoDropout to automate the design of dropout patterns which can be generalized for different applications. AutoDropout leverages reinforcement learning to find the best dropout pattern over a structured search space, where the reward for the RL is the validation performance of the dropout pattern.

Search space for dropout patterns in ConvNets is based on a contiguous, tiled, rectangle, the hyper-parameters includes height, width, stride and repeats. In addition, geometric transformations including rotating and shearing are also added to the search space. Search space for dropout patterns in Transformers is similar. Dropout pattern is realized by four hyper-parameters: size, stride, type of noise mask, how many token does a pattern affects. Unlike ConvNets, dropout pattern can be applied to multiple sublayers in Transformer.

Experiments showed that for supervised and semi-supervised image classification tasks, AutoDropout outperforms SOTA methods, including methods that requires manual dropout pattern design.For Transformers, AutoDropout outperforms Transformer-XL which leverages Variational Dropout.

## **Critique**

1. For both ConvNets and Transformers the authors proposed a fixed set of hyper-parameters as the search space. However I don’t think they addressed how these hyper-parameter was selected. If they are selected manually based from empirical experiences, then AutoDropout really isn’t much different from applying optimization on a set of manually selected features.
2. Authors mentioned that the training cost of AutoDropout could be high because it’s based on RL, however, they did not benchmark any training cost compared to the existing methods.
3. Authors only briefly mentioned RL is used as the search method even though the core component of AutoDropout is RL. I hope they can explain in detail what are the actions, states and why RL is best suited for this task.