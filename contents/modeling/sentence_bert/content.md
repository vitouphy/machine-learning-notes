# Sentence-BERT
[Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks](https://arxiv.org/abs/1908.10084)

## Problem

BERT is a Transformer-based model that has been widely used in many applications in natural language processing, including sentence classification, named entity recognition, or machine reading. However, such a model is so big that even it has a hard time running on a typical personal computer, let alone on a mobile device. Generally, mobile devices with 2GB of GPU and 4GB of RAM resources, for example, are shared with many other tasks, which makes it even more challenging to have that big model running in the background. 

That's when MobileBERT comes into play. Its name is pretty self-explanatory. We try to reduce the size of BERT so that it can fit onto the phone system while maintaining its performance.

## MobileBERT Concept

In a nutshell, MobileBERT is like a distilled version of the BERT model. With the original trained BERT model, we try to transfer its knowledge and learned weights onto a smaller version of BERT. In other words, smaller BERT tries to mimic the output of the big BERT of every layer. 

Here are some key concepts of MobileBERT:
- **It is a deep and thin model**. Compress the BERT model not by reducing its depth but by reducing its width. The depth here refers to the number of layers; whereas width refers to the number of parameters in each layer. 
- **Distill knowledge from the big BERT**. You can train that small BERT from scratch as well, but throughout different experiments, it's concluded that distilling reaches better performance.
- **Fine-tunable for the downstream task**. With the trained mobileBERT, we can still fine-tune that further to your downstream tasks, for example, question and answering. That way we don't have to do the whole distilling from the big question and answering model.

## Architecture

Let's go over some architecture detail and key building blocks for MobileBERT. 

```{figure} ./figures/architecture.png
---
name: architecture
width: 480px
align: center
---
MobileBERT Architecture
```

### Inverted-Bottleneck BERT
Let me add some detail that MobileBERT isn't distilled directly from the original BERT. It's distilled from Inverted-Bottleneck BERT, which is quite similar to BERT. The major difference is introducing the bottleneck concept to the beginning and end of each encoder block. Suppose that the input is a 100-dimension vector. The encoder block expands it into 512, does some processing, and squashes it down back to 100-dimension as output. By doing so, we can control the encoder block of IB-BERT and the MobileBERT to be having the same input and output size. This **helps the knowledge transferring between the two models** that we will discuss later on. The main difference would be that IB-BERT's encoder block internally expands into much higher parameters.

### Knowledge Transfer Function
How do they transfer the knowledge from IB-BERT to MobileBERT? It's done at one layer (encoder block) at a time. For each encoder block, there is feature map transfer and attention transfer. 
- **Feature map transfer** is done via mean squared error between the output of an encoder block between both models. Thanks to the design choice of making the input and output of the encoder block the same for IB-BERT and MobileBERT, it makes the transfer easier.
- Attention Transfer is done **per attention head** using KL-divergence. Again, this is also possible because the parameters of each attention have the same size. For more information on how MHA behaves, please read this awesome [blog post](https://jalammar.github.io/illustrated-transformer/).

### Progressive Knowledge Transfer

```{figure} ./figures/progressive_knowledge_transfer.png
---
name: progressive_knowledge_transfer
width: 480px
align: center
---
Progresive Knowledge Transfer. Small dot is copy operation. Dash is knowldge transfer, and dash+dot is knowledge distillation.
```

Two things to keep in mind here. (1) some layers are copied right over, i.e., embedding and classifier layers, and (2) for the other layer, the knowledge transfer is done one layer at a time and by freezing the transferred layer. Finally, the final knowledge distillation is performed, meaning the final output vector of IB-BERT model is compared against that of MobileBERT via KL-divergence. 


### Other Details
**MHA : FFN (1:2)**: One of the key attributes of the BERT model is the number of parameters in the fully-forward network (FFN) has double the size of that in the multi-headed attention module (MHA). Empirical analysis shows that it helps with the training. In the MobileBERT architecture, the parameters of MHA are almost the same as FFN. To solve this, multiple FFNs are stacked on top of one another to maintain the ratio.

**Replacing layer normalization**: Latency is very crucial when it comes to running on mobile. However, LayerNorm is an expensive operation when it comes to processing time. So, it is replaced with a simple element-wise linear transformation. Apparently, that gets the job done. It's faster while still maintaining the performance

## Results

For all downstream tasks, MobileBERT is fine-tuned directly just like we regularly fine-tune the BERT model, except that it's better to use a large learning rate and more epochs.

```{figure} ./figures/result.png
---
name: result
width: 720px
align: center
---
result
```

As the table suggests, MobileBERT performs relatively well compared to the BERT base model while having fewer parameters and latency. Similarly, it also performs between than the other variants of the BERT model.

## Conclusion

MobileBERT has drastically reduced the parameters and latency to the point that it can be run on mobile. More importantly, it can be done so while maintaining performance comparable to the original BERT model, and it can easily be trained on a downstream task just like the regular BERT. That opens so much possibility for bringing BERT capability to the palm of your hands.

## References
- [MobileBERT: a Compact Task-Agnostic BERT for Resource-Limited Devices](https://arxiv.org/abs/2004.02984)