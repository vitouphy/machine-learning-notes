# Large Pretrained Model

Some models are pre-trained on large datasets, e.g., ImageNet, to perform a certain task like image classification or object detection. It appears that the models can capture different information at their layers. For example, the first layer captures the edge in the image, and the second captures the area, and so on. We can use the weights of these lower layers to further train for another task. This eliminates the need to train the model from scratch all the time.

In this section, we will look into popular pre-trained models.
- [ResNet](#residual-network-resnet)
- [MobileNet](#mobilenet)
- VGG
- Inception
- Xception
