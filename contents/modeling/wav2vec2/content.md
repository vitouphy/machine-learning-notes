# Wav2Vec 2.0
[Wav2vec 2.0: A Framework for Self-Supervised Learning of Speech Representations (Baevski. et al, 2020)](https://arxiv.org/abs/2006.11477)

```{note}
- Based on the Transformer model, Wav2Vec 2.0 is trained using a self-supervised method to predict masked audio segments.
- Perform well in low-resource multilingual languages.
```

## Motivation

There are two main motivations for this paper:

**1. Can there be a better way to train the wave-to-vec model without any label data?**

If we compare this to training text-based language model (LM), working with text is much easier to do self-supervised learning. Commonly, a token is masked and the model is used to guess what that masked word is, e.g., the BERT model. Alternatively, given $n$ tokens, the model is trained to predict the next one. However, in audio, things are slightly more complicated, specifically, it's no longer about predicting a token but predicting a sequence of the audio segment.

**2. Can the model benefit from training on multilingual data?** 

If we have the same model architecture but are trained on two different types of data (monolingual and multilingual), which one is more beneficial for the downstream task?



## Concept

The core of Wav2Vec2 is based on the standard Transformer model. It's one of the main building blocks of this architecture. The main difference is its input and the loss function.

```{figure} ./figures/model.png
---
name: Wav2Vec 2.0 Transformer Architecture
width: 700px
align: center
---
Wav2Vec 2.0 Transformer Architecture
```

- **Preprocessing:** This model takes in raw audio. No audio preprocessor (or Fourier transform) is required. 
- **Latent Representation:** Raw audio $X$ is split into chunks (with some overlappings) and fed into CNN $Z$ to extract its latent representation. For instance at chunk $i$, $x_i \rightarrow z_i$.
- **Contextual Representation:** This latent representation $z_i$ along with $z_j$ at other time steps are fed into the standard Transformer to obtain contextual representation $c_i$. What this means is that the vector $c_i$ captures and is aware of nearby audio chunks.
- **Product Quantization:** The same $z_i$ is also get discretized into $l_i$. In other words, we try to put a label onto $z_i$. 
    - $q$ has two modules where each element has a vocab size of 320.
    - This corresponds to 102.4K combinations of code words.
    - This discretize module uses **Gumbel-Softmax** to allow the flow of differentiation because the regular **argmax** function is indifferentiable.
    - With the discretized $l_i$, it is projected back into a vector and concatenated (since we are having two modules for each $l$) into a vector $q_i$ of the same size as $c_i$.
- **Masked Prediction:** Remember that the model does not see $z_i$ because $z_i$ is masked and instead obtains $c_i$ from its surrounding context? Now model needs to pick which $q_i$ is the most relevant to the embedding $c_i$? 
    - The comparison between $c_i$ and $q_i$ is done using a simple cosine similarity function. 

```{figure} ./figures/contrastive_loss.png
---
name: Contrastive Loss Function
width: 500px
align: center
---
```

- **Diverse Discretize**: There can be cases where the model does not utilize all the vocabulary in the $Q$ module. It might just pick a subset of $Q$ again and again. To combat this problem, they include a diversity loss function. Essentially, it penalizes when the model is very certain of a particular code label. The higher the entropy, the bigger loss value is.

```{figure} ./figures/diversity_func.png
---
name: Diversity Loss Function
width: 500px
align: center
---
```

- **Extra 💬**: If the model ends up comparing vector $c_i$ with another vector $q_i$, why need to go through all those quantization processes? Why don't we just use $z_i$ instead? 
    - If we compare with $z_i$, that would mean we want the model to output exactly the same as the latent representation module ($Z$). This would just turn this Transformer module into an identity function. 




## Dataset 
The model was experimented with and trained on Librispeech (960 hours of audio) or LibriVox (53.2K hours of audio). Both datasets are not merged. Based on [this blog post](https://ai.facebook.com/blog/wav2vec-20-learning-the-structure-of-speech-from-raw-audio/), it seems that the publicly released version of Wav2Vec 2.0 is pre-trained on **LibriVox** dataset. 


## Strengths and Limitations
### Strengths
- This self-supervised learning approach is successful. Fine-tuning this on the benchmark dataset (ASR) outperforms the existing models. 
- The fine-tuning does not require that much data. This enables the downstream task on low-resource language to be efficient. It can go down to 10 hours or even 1 hour. 
    - 💬 We try to fine-tune this on the Khmer language recognition task using only around 3 hours. And the performance is mindblowing, given using low resource language.

### Limitations
- The limitation of this model is not properly discussed in the paper.



## Conclusion
- We can now apply self-supervised learning to raw audio data.
- Fine-tuned Wav2Vec 2.0 performs well on both high and low-resource languages (down to 10 hours). It can also go down to 1 hour. Obviously, it wouldn't be that good compared to having more data, but it's feasible.

<br/>
<hr/>
<br/>

💬 denotes **opinion** and is not from the paper.