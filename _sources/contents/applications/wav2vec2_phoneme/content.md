# Wav2Vec2 for Phoneme


```{note}
- Use Wav2Vec2 and train to recognize phonemes that are automatically generated.
- Perform quite well in both seen and unseen language.
```

## Background

This is a summary of the paper Simple and Effective Zero-shot Cross-lingual Phoneme Recognition ([Arxiv](https://arxiv.org/abs/2109.11680)). The goal of this work is to apply the Wav2Vec2 model to recognize phonemes from raw audio.

### Application of Wav2Vec2 

A Transformer-based model, called [Wav2Vec2](https://arxiv.org/pdf/2109.11680.pdf) outperforms many of the existing work, particularly in low-resource language. This work explores how well this model behaves for the task of translating from audio to phonemes. Note that a phoneme (/ˈfoʊniːm/) is a unit of sound that can distinguish one word from another in a particular language [(Wikipedia)](https://en.wikipedia.org/wiki/Phoneme). It represents a proper way to pronounce a word.

### Procedure

Briefly, how Wav2Vec2 works is that it pretrains using only raw audio via self-supervision. No preprocessing or label data is needed. 
- It maps every 25ms of audio strided by 20ms into a latent speech representation $z_t$
- For each time step, $z_t$ goes into the Transformer model and produces the contextual representation $c_t$ (like regular Transformer, which also utilizes $z$ at other time steps).
- $z_t$ is also discretized into $q_t$. In other words, It gets converted into a label integer.
- The model learns to determine which $q_i$ is corresponded with $c_t$.

### Phoneme Recogntion

After pretraining the Wav2Vec2, a [Connectionist Temporal Classification (CTC)](https://en.wikipedia.org/wiki/Connectionist_temporal_classification) layer is added on top of the transformer model so that it can predict the phoneme for each timestep. Note that for this step, the model no longer learns to find the corresponding $q_i$ and $c_t$. It focuses only on getting the right phoneme prediction.


## Dataset and Setup 

The dataset used for this work is BABEL and CommonVoice. These datasets, however, do not contain any label data for phonemes. Instead, separate tools are used to automatically generate those labels, i.e., ESpeak or Phonetisaurus. Note that only one tool should be used for consistent labeling. 

## Result 

### Zero Shot Performance 
```{figure} ./figures/zero-shot.png
---
name: zero-shot performance
width: 400px
align: center
---
For each column language, (for example **de**), each model is trained on all languages beside **de** and is tested on that language. The performance of this zero-shot is quite astounding. 
```

### Effect of Pretraining 
```{figure} ./figures/pretrain.png
---
name: zero-shot performance
width: 360px
align: center
---
Zero-Shot performance of different pre-trained models.
```
XLSR-53 is a multilingual pretrained model (the one we have been discussing so far). w2v LV-60K is the same model architecture but it's pretrained on English + language specified by each row. No pretrained refers to the original model that pretrained only using the English language. The score is phoneme error rate (PER), the lower the better.

Having no pretrained achieves the worst performance. We simply can't rely only on English speech and accent. It's not enough. At least, pretrained on the target language. It shows substantial improvements over the non-pretrained one. As for XLSR-53, it even shows improvements further. **Hence, why not just pretrained once on all languages, and just use that**. Then, it makes downstream tasks a lot easier.



## Conclusion 
- Wav2Vec2 is trained on CommonVoice and BABEL dataset using ESpeak / Phonetisaurus to create phoneme labels automatically.
- It can deliver substantially high performance on phoneme recognition tasks for both seen and unseen languages.
- Use a multilingual pretrained model if possible, particularly if the downstream tasks are on multiple unseen languages.


## References
- Xu, Qiantong, Alexei Baevski, and Michael Auli. "Simple and Effective Zero-shot Cross-lingual Phoneme Recognition." arXiv e-prints (2021): arXiv-2109.