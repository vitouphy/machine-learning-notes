# NaN on Training

:::{note} 
Encounter NaN during training? Try:
- Clip gradient before updating the parameters.
- Add L1/L2 regularization.
- Adjust hyper-parameter (batch size, learning rate)
- Make sure all operations is performing as expected.
:::

## Problem
You might have experience getting NaN as loss value during training. This implies that loss value is so big that it can not store in float anymore. This is also commonly related with `Exploding Gradient`. 

## Exploding Gradient
Gradient, informally, is a value informing which direction and how much we should update each parameter. Every parameter has its own gradient to update during each training step. Often, the gradient can get very large. 

For example, assuming the gradient is 1.05, learning rate is 1,0, its corresponding parameter $w=1$. At the training step 1,000, we will have $w=1.54*10^{21}$. That is huge. Large-value parameters, obviously, will result in large loss value. That is why NaN occurs during training.

## Solutions

There are some solutions available to handle this problem.

### 1 - Clip Gradient
We can control the value of gradient by $min(\text{gradient}, y=10)$, where y is any positive real number. For the gradient of maximum 10, the learning rate, of course, should not be 1.0 but lower.

### 2 - Weight Regularization
Gradient clipping may help dealing with the NaN issue, but it is not guaranteed because it does not set any constraints on the size of each parameter. It can still be large. To decrease the likelihood of its occurence, weight regularization come into play. Essentially, weight regularization (e.g., L1 or L2) penalizes the model if its parameters get too big. 

### 3 - Check Your Model
Running wrong operations, for example matrix multiplication instead of element-wise multiplication, may also be the cause. So, always check and make sure your implementaion is doing what it is supposed to be doing. 

## References
- [A Gentle Introduction to Exploding Gradients in Neural Networks](https://machinelearningmastery.com/exploding-gradients-in-neural-networks/)
- [Common Causes of NANs During Training](https://sisyphus.gitbook.io/project/deep-learning-basics/deep-learning-debug/common-causes-of-nans-during-training)