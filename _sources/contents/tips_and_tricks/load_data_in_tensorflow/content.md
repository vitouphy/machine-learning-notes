# Load Data in TF.Data

:::{note} 
Benefits of TF-Data on the loading-preprocessing pipeline:
- **Small Dataset**: If all your data can reside in memory, then TF-Data does not help much. Well, at least, it makes the code cleaner.
- **Large Dataset**: Take advantage of multi-core CPU and speed up the pipeline.
:::

## Problem
There are two steps to train a model. One is to load data from the disk onto the memory (RAM). The second is to use that data and train the model. Without careful design, loading the data can take more time than the actual training itself, which is inefficient and costly. 

**Working with Small Dataset:** We can simply tackle this problem by just loading all data into memory before starting the training. Hence, the data will always be available for training. This is completely fine if the data can fit inside the memory. However, that is not always the case.

**Working with Large Dataset:** Imagine you have to train a classification model on 1 million images. Each image is around 100KB in size, which would amount to 100GB. Unless you have that much amount of RAM, loading all of them is not an option. 

That is when TF-Data comes into play.

## TF-Data
In a nutshell, TF-Data loads the data into memory only when they are needed. To avoid the delay when loading the data, TF-Data preloads them in the background during the training. It also makes use of multi-core on your machine to speed up the loading process.

On top of them, the TF-Data pipeline makes data pre-processing and data augmentation a lot more convenient with the function `.map()`. If we add another argument `num_parallel_calls=tf.data.AUTOTUNE`, we can have those functions perform in parallel as well. Note that `.prefetch()` is important to preload the data in the background.

```python
dataset = (tf.data.Dataset.from_tensor_slices((image_path, label))
           .map(load_image, num_parallel_calls=tf.data.AUTOTUNE)
           .map(normalization, num_parallel_calls=tf.data.AUTOTUNE)
           .batch(32)
           .prefetch(tf.data.AUTOTUNE))
```


## TF Records
TF.Data is quite efficient in itself for loading data, but it won't be efficient enough if the data locates online, e.g., AWS S3 or Google Cloud Storage. Then, there is an additional step, which is to download the data. As you may already know, copying or downloading 1 million of 100KB files is slower than one 100GB file. If your bandwidth supports up to 10MB per second and you only download 100KB at a time, then we are not using the bandwidth optimally. 

In this case, we can group 1024 images into a file of size around 100MB. In other words, we can fetch 1024 images in 10 seconds. That sounds like a good deal, but it seems like a ton of work.

Luckily, there is TF-Records, which informally, is like JSON but for storing tensors.

Let's define a function for serializing tensors into string. Each record (`tf.train.Example`) consists of multiple `tf.train.Feature`.
```python
def serialize_example(image_binary, label):
  feature = {
      'image_binary': _bytes_feature(image_binary),
      'label': _int64_feature(label)
  }

  proto = tf.train.Example(features=tf.train.Features(feature=feature))
  return proto.SerializeToString()
```

The above function may not work properly with the `.map()`. The following wrapper may help with that.
```python 
def tf_serialize_example(image_binary, label):
  tf_string = tf.py_function(serialize_example, (image_binary, label), tf.string)     
  return tf.reshape(tf_string, ()) 
```

Here are the helper functions for converting each data type to `tf.train.Feature`.
```python
def _bytes_feature(value):
  """Returns a bytes_list from a string / byte."""
  if isinstance(value, type(tf.constant(0))):
    value = value.numpy() # BytesList won't unpack a string from an EagerTensor.
  return tf.train.Feature(bytes_list=tf.train.BytesList(value=[value]))

def _float_feature(value):
  """Returns a float_list from a float / double."""
  return tf.train.Feature(float_list=tf.train.FloatList(value=[value]))

def _float_feature_array(value):
  """Returns a float_list from a float / double."""
  return tf.train.Feature(float_list=tf.train.FloatList(value=value))

def _int64_feature(value):
  """Returns an int64_list from a bool / enum / int / uint."""
  return tf.train.Feature(int64_list=tf.train.Int64List(value=[value]))
```

Let's rebuild dataset with this new serialize function.
```python
dataset = (tf.data.Dataset.from_tensor_slices((image_path, label))
           .map(load_image, num_parallel_calls=tf.data.AUTOTUNE)
           .map(normalization, num_parallel_calls=tf.data.AUTOTUNE)
           .map(tf_serialize_example, num_parallel_calls=tf.data.AUTOTUNE)
           .prefetch(tf.data.AUTOTUNE))
```

With the dataset ready, we can save that to a file. Note that if we want to group 1024 images into one file, a simple loop and if-else would do the trick.
```python
writer = tf.data.experimental.TFRecordWriter('file.tfrecords')
writer.write(dataset)
```

### Deserialize
Use the following code to convert back to tensor objects.
```python
def _parse_serialize_function(proto):
    feature = {
        'image_binary': tf.io.FixedLenFeature([], tf.string),
        'label': tf.io.FixedLenFeature([], tf.int64),
    }
    return tf.io.parse_single_example(proto, feature)
```


## Conclusion
We have discussed use cases for both tf.Data and tf-records. **tf.Data** simplifies the pipeline for loading the data as well as speeds up the process by utilizing the multi-processing capabilities. **TF-Records** helps make data transport and data loading even faster, especially if you need to load your data from a remote data storage.


## References
- [tf.data.Dataset](https://www.tensorflow.org/api_docs/python/tf/data/Dataset)
- [TFRecord and tf.train.Example](https://www.tensorflow.org/tutorials/load_data/tfrecord)
- [A hands-on guide to TFRecords](https://towardsdatascience.com/a-practical-guide-to-tfrecords-584536bc786c)
- [Creating TFRecords](https://keras.io/examples/keras_recipes/creating_tfrecords/)
- [Storage efficient TFRecord for images](https://medium.com/coinmonks/storage-efficient-tfrecord-for-images-6dc322b81db4)