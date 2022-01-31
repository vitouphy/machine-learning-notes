# Vision Transformer
[An Image Is Worth 16x16 Words: Transformers For Image Recognition At Scale (Dosovitskiy. et al, 2021)](https://arxiv.org/abs/2010.11929)

```{note}
- It's like a regular Transformer but for computer vision.
- Swapping word embedding with image embedding.
- Split an image into N patches. Get embedding for each patch and feed to Transformer.
```

## Motivation

Transform has become the go-to model in natural language processing ever since its initial appearance in 2017. However, there aren't many works that studies its capability in computer vision yet. 

The goal of this work is to make use of the Transformer model in computer vision.

## Concept
### Transformer Block
- We won't go into depth on the main architecture of the Transformer. 
- Briefly in machine translation, given a sequence of words, we get the vector representation (embedding) for each word and pass them to Transformer.
- Additional positional embedding is also added on top of the word embedding so that the model is aware of where that word is.
- Transformer takes in a sequence of vectors and computes each timestep (word) independently from the previous timestep, unlike recurrent neural network (RNN). This allows the model to be trainable in parallel across multiple machines. 
- To be aware of the surrounding words, Transformer uses a self-attention mechanism. It may associate a word at timestep $t$ with $n$ words before and after it.


### Vision Transformer
- For computer vision, it's a little different since we are not working with a sequence of words. The only thing we need to change is its input.
- Hence, we split an image into $N$  patches of size $m \times m$. Then, we rearrange those $N$ patches into an array (similar to an array of words).
- Reshape each patch and apply a linear transformation layer to get its corresponding embedding. Note that this layer will be shared across all timesteps.
- Add position embedding and feed them into Transformer.

```{figure} ./figures/vit_model.png
---
name: Vision Transformer Architecture
width: 700px
align: center
---
Vision Transformer Architecture
```

## Strengths and Limitations
- Pretrained on a large dataset (at least 100M samples) yields better performance compared to ResNet architecture (non-Transformer SOTA in computer vision).
- Experimenting with a dataset of size 10M and 30M shows that the performance is somewhat limited.
- Such limitation is not only for vision but also in the NLP domain as well.

```{figure} ./figures/performance.png
---
name: Vision Transformer Architecture
width: 700px
align: center
---
Left Figure: Transfer to ImageNet. Right Figure: Few-shots on ImageNet. Both suggests that big dataset leads to performance improvement on Vision Transformer (ViT)
```

## Conclusion
- Now we can input an image into a Transformer by modifying the input to the model.
- Pretraining on big data of more than 100M shows the potential of Transformer.
- With lower data, ResNet is more promising in terms of accuracy.
