# Angular Similarity

```{note}
- Angular similarity is similar to cosine similarity but better.
- Two small angles may result in very similar cosine similarity. Angular similarity makes the two values more distinguishable from one another.
```


## What's wrong with cosine similarity?

Cosine similarity is flawed when then angle between $u$ and $v$ is very small. As shown in the figure below (right), when the angle $\varphi$ is close to 0, the cosine similarity is very close to each other.

Such small difference makes comparison between two angles very difficult.

```{figure} ./figures/dot-product-curves.svg
---
name: angular_similarity
width: 480px
align: center
---
Behaviours of different similarity functions. (Source: [StackExchange](https://math.stackexchange.com/questions/2874940/cosine-similarity-vs-angular-distance) )
```


## Angular Similarity

The formula for this similarity is as follows:

$$ 
f(x)=1-\frac{\text{arccos}(x)}{\pi} \Leftrightarrow f_\varphi (\varphi) = f(cos(\varphi))=1-\frac{\varphi}{180^o}
$$

where $\varphi$ is the angle in degree between two vectors, $f$ is a function to map angle into similarity.

Angular similarity linearly maps the value. It makes the change in $\varphi$ linearly correlates with its similarity. This makes the comparison more consistent. The other functions, i.e., cubic function and `square root function`, are just there for comparison. Below is the square root function.

$$
f(x)=1-\sqrt{\frac{1-x}{2}}
$$

## Food for Thoughts
When working with high-dimensional data, it is very likely to experience `curse of dimensionality`. Meaning the higher dimension we are working with, the closer the value moves toward 90 degrees.

There was one blog illustrating that if we randomize two vectors of high dimensions, in average the angle between two randomly selected vectors is very close to 90 degrees, which makes things super difficult when it comes to comparison.

**Does this mean that each small difference in value drastically impact on the similarity?**

If that is really the case, wouldn't that make `square-root function` more suitable when it comes to dealing with high-dimensionality data?


## References
- [https://math.stackexchange.com/questions/2874940/cosine-similarity-vs-angular-distance](https://math.stackexchange.com/questions/2874940/cosine-similarity-vs-angular-distance)
- [https://en.wikipedia.org/wiki/Cosine_similarity#Angular_distance_and_similarity](https://en.wikipedia.org/wiki/Cosine_similarity#Angular_distance_and_similarity)
- [http://www.uco.es/users/ma1fegan/Comunes/asignaturas/vision/Encyclopedia-of-distances-2009.pdf](http://www.uco.es/users/ma1fegan/Comunes/asignaturas/vision/Encyclopedia-of-distances-2009.pdf)