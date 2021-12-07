# EfficientNet 
[EfficientNet: Rethinking Model Scaling for Convolutional Neural Networks (Tan. et al, 2019)](https://arxiv.org/abs/1905.11946)

```{note}
- Scale depth, width, and resolution all together achieve better performance than scaling only one of them.
- It allows information to flow better. Bigger resolution requires more channels to capture more features, and it requires more layers to process them properly.
- Mobile Inverted Bottleneck is one of the building blocks.
- Use neural architecture search (NAS) to help with searching for the right parameters.

```

## Problem

There are two problems that this work is addressing:
- The effect of the number of layers, channels, and resolutions
- Finding the next small-scale model that performs well

For the first problem, there are many papers that improve the model by scaling the number of layers, numbers of channels, or the resolution of the image. Some studies on one of the values. Others study on two together. However, there was not any that looked all three of them.

For the second problem, obviously, if we can achieve better performance on smaller models, it would help with computation on low-power devices (e.g., mobiles) or on edge devices.

## Concept
### EfficientNet Block
- **Mobile inverted bottleneck** is the building block for this network. It utilizes the concept of skip-connection, a squeeze-and-excitement mechanism, with depth-wise separable convolution for small parameters.
- Neural architecture search (NAS) is used to find the optimal network for this model.

```{figure} ./figures/MB_block.png
---
name: MB_block
width: 300px
align: center
---
Mobile Inverted Bottleneck. The first Conv1x1 (from input) is for scaling up and the second is for compressing. (Figure from MobileNet-v2's paper)
```

### Compound Scaling

```{note}
It's critical to balance all three aspects to pursue better performance.
```

This work focuses on three aspects: 
- depth $d$: Number of layers.
- width $w$: Number of channels.
- resolution $r$: Image size (Refer to the image size of every layer).

The experiment shows that scaling one of the aspects will improve the performance. However, it quickly reaches its limit and starts to flatten out. **Scaling on all three** yields the best performance.

```{figure} ./figures/effect_compound_scaling.png
---
name: effect_compound_scaling
width: 480px
align: center
---
Scaling Up EfficientNet-B0
```

This compound scaling effect is not only applicable to EfficientNet but also to other convolutional models, for instance, ResNet-50 and MobileNet-v2.


### Usage

First, a base model is needed. Using that base model, we create another one by scaling up and set the width, depth, and resolution by multiplying with the following:
- depth: $d = \alpha ^ \phi$
- width: $w = \beta ^ \phi$
- resolution: $r = \gamma ^ \phi$

For simplicity, it is assumed that the three will be scaled by a certain rate. $\alpha, \beta, \gamma$ are constants. Only $\phi$ is changed. 

```{note}
The number of depths here does not refer to the total number of layers in the network. Instead, this coefficient refers to the frequency that a block (or a stage) should repeat itself, as shown in the table below. 
```

Let's apply this to the EfficientNet. First, EfficientNet-B0 is obtained as the base model. Then, they set ($\phi = 1$) and use neural architecture search (NAS) to search for the $\alpha, \beta, \gamma$  values that best optimize the performance for EfficientNet-B1. Once the three values are obtained, we only need to change $\phi$ to $2, 3, ... N$ to scale the model and get EfficientNet-B$\phi$.  

```{figure} ./figures/efficientnet_staging.png
---
name: efficientnet_staging
width: 480px
align: center
---
Detail on EfficientNet-B0
```

Let's attempt to create EfficientNet-B7. From the paper, the NAS suggests $\alpha=1.2$, $\beta=1.1$, and $\gamma=1.1$. With $\phi=7$:
- $d=(1.2)^7=3.58$
- $w=r=(1.1)^7=1.95$

If we look at stage 4, the resolution should now be $109 \times 109$. The number of channels should be $78$, and the number of layers should be $7$. Viola, that's how we obtain EfficientNet-B7.