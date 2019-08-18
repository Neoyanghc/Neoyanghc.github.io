---
layout:     post
title:      "jacktf_tensorflow个人深度学习框架1"
subtitle:   " \"开始编写对应的data_process and data_provide 函数接口\""
date:       2019-8-17 17:00:00
author:     "jack"
header-img: "img/post-bg-tf.jpg"
catalog: true
tags:
    - CNN
    - 学习笔记
    - 深度学习
---

### TFRecord，Tensorflow Data API 与 数据处理

![](https://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/20190817220236.png)

#### 1. 数据处理

深度学习，大数据技术等等第一步都离不开数据处理，我们以部分数据集cifar，mnist图片数据进行对应的处理

这部分代码存放在 `data_provide.py` 文件中

先来介绍一下其中几个比较重要的函数

+ 传入图片.jpg存放的地址，然后将其分为,train_dict 和 val_dict，dict为[path,label]

  `def split_train_val_sets(images_dir, val_ratio=0.02)`

+ 获取图片对应的label，并返回dict字典

  `def _get_labling_dict(image_files=None)`

+ 将dict写入json文件中

  `def write_annotation_json(images_dir, train_json_output_path, val_json_output_path)`

##### 1.1 Kaggle cat vs dog 数据集处理

数据集和数据集之间的区别，其实就是在于怎么读入数据，然后怎么获取label,生成json文件

例如 **cat vs dog** 的数据集，其实就是**cat0002.jpg** 的格式存储的

那其实我们利用前缀去匹配就可以了

但是**cifar**就是提供一个**label.txt**，我们需要打开label.txt进行读取，进行映射，不过也很简单

有些甚至数据不是以**.jpg格式**存储的，只要是图片我们就可以存储成其形式进行处理

```python
# 传入图片.jpg存放的地址，然后将其分为,train_dict 和 val_dict，dict为[path,label]
def split_train_val_sets(images_dir, val_ratio=0.02):
    if not os.path.exists(images_dir):
        raise ValueError('`images_dir` does not exist.')
    # image_files 为寻找到的所有的图片路径名
    #  image_files = glob.glob(os.path.join(images_dir, '*.png'))
    image_files = glob.glob(os.path.join(images_dir, '*.jpg'))

    # 进行分割
    num_val_samples = int(len(image_files) * val_ratio)
    val_files = image_files[:num_val_samples]
    train_files = image_files[num_val_samples:]
    
    train_dict = _get_labling_dict(train_files)
    val_dict = _get_labling_dict(val_files)
    return train_dict, val_dict

# 获取图片对应的label，并返回dict字典
def _get_labling_dict(image_files=None):
    if image_files is None:
        return None
    # kagger，猫vs狗数据集的处理方式
    # 其他数据集，主要修改这里
    labling_dict = {}
    for image_file in image_files:
        image_name = image_file.split('/')[-1]
        if image_name.startswith('cat'):
            labling_dict[image_file] = 0
        elif image_name.startswith('dog'):
            labling_dict[image_file] = 1
    return labling_dict

```

##### 1.2 cifar-10 数据集处理

```python
# 解压缩，返回解压后的字典
def unpickle(file):
    fo = open(file, 'rb')
    dict = pickle.load(fo, encoding='latin1')
    fo.close()
    return dict

def save_to_jpg(images_dir)
    # 生成训练集图片，如果需要png格式，只需要改图片后缀名即可。
    if not os.path.exists(images_dir):
        raise ValueError('`images_dir` does not exist.')
     
    for j in range(1, 6):
        dataName = "data_batch_" + str(j) 
        Xtr = unpickle(dataName)
        print(dataName + " is loading...")
        for i in range(0, 10000):
            img = np.reshape(Xtr['data'][i], (3, 32, 32))  
            # Xtr['data']为图片二进制数据
            img = img.transpose(1, 2, 0)  # 读取image
            picName = images_dir+'/train/'+str(Xtr['labels'][i])
            \							+ '_' + str(i + (j - 1)*10000) + '.jpg'  
            imsave(picName, img)
        print(dataName + " loaded.")
    print("test_batch is loading...")
    
    testXtr = unpickle("test_batch")
    for i in range(0, 10000):
        img = np.reshape(testXtr['data'][i], (3, 32, 32))
        img = img.transpose(1, 2, 0)
        picName = images_dir+'/test/' + str(testXtr['labels'][i]) + '_'+ str(i)'.jpg'
        imsave(picName, img)
    print("test_batch loaded.")
    
def _get_labling_dict(image_files=None):
    if image_files is None:
        return None
    # cifar 数据的打开方式
    labling_dict = {}
    for image_file in image_files:
        image_name = image_file.split('/')[-1]
        labling_dict[image_file] = image_name[0]
    return labling_dict
```

#### 2. TFrecord 生成

##### 2.1 TFrecord的优势

正常情况下我们训练文件夹经常会生成 train, test 或者val文件夹，这些文件夹内部往往会存着成千上万的图片或文本等文件，**这些文件被散列存着**，这样不仅占用磁盘空间，并且再被一个个读取的时候会非常慢，繁琐。占用大量内存空间（有的大型数据不足以一次性加载）。

此时我们TFRecord格式的文件存储形式会很合理的帮我们存储数据。TFRecord内部使用了“Protocol Buffer”二进制数据编码方案，它只占用一个内存块，只需要一次性加载一个二进制文件的方式即可，简单，快速，尤其对大型训练数据很友好。

而且当我们的训练数据量比较大的时候，可以将数据分成多个TFRecord文件，来提高处理效率。

##### 2.2 generate_tfrecord.py

利用data_provide.py 传递出来的参数进行将images.jpg 变成 .record 文件

利用 tf.dataset API 可以进行处理

指定图片的路径，然后指定图片resize的固定大小

**关键函数**

+ 定义协议和对一个图片数据进行 创建

  `def create_tf_example(image_path, label, resize_size=None)`

+ 利用输入的json文件中的dict进行tfrecord文件的写出

  `def generate_tfrecord(annotation_dict, output_path, resize_size=None)`

1. 首先利用 tf.gfile 对 图片文件进行打开，然后将其读取到内存中

```python
def create_tf_example(image_path, label, resize_size=None):
    # 打开图片文件 python open 函数也可以，但是tf.gfile 可以打开tf的指定文件
    with tf.gfile.GFile(image_path, 'rb') as fid:
        encoded_jpg = fid.read()
    
    # 在内存中读取IO文件
    encoded_jpg_io = io.BytesIO(encoded_jpg)
    image = Image.open(encoded_jpg_io)
    width, height = image.size
```

2. 是否需要进行resize

```python
    if resize_size is not None:
        if width > height:
            width = int(width * resize_size / height)
            height = resize_size
        else:
            width = resize_size
            height = int(height * resize_size / width)
        image = image.resize((width, height), Image.ANTIALIAS)
        bytes_io = io.BytesIO()
        image.save(bytes_io, format='JPEG')
        encoded_jpg = bytes_io.getvalue()
```

3. **定义tfrecord的协议**

   这部分刚接触的时候，也是一脸懵逼不知道是什么意思

   但是实际意义上就是在定义，这个数据的类型，包括整形，float等等

   key：要保存数据的名字    value：要保存的数据，但是格式必须符合tf.train.Feature实例要求。

```python
tf_example = tf.train.Example(
        features=tf.train.Features(feature={
            'image/encoded': bytes_feature(encoded_jpg),
            'image/format': bytes_feature('jpg'.encode()),
            'image/class/label': int64_feature(label),
            'image/height': int64_feature(height),
            'image/width': int64_feature(width)}))

def int64_feature(value):
    return tf.train.Feature(int64_list=tf.train.Int64List(value=[value]))
def int64_list_feature(value):
    return tf.train.Feature(int64_list=tf.train.Int64List(value=value))
def bytes_feature(value):
    return tf.train.Feature(bytes_list=tf.train.BytesList(value=[value]))
def bytes_list_feature(value):
    return tf.train.Feature(bytes_list=tf.train.BytesList(value=value))
def float_list_feature(value):
    return tf.train.Feature(float_list=tf.train.FloatLst(value=value))
```

4. 利用write进行写入

   先创建一个writer实例，告诉它的存储路径

   `writer = tf.python_io.TFRecordWriter(record_path)`

   再一个个写入

   `writer.write(tf_example.SerializeToString())`

   最后将它关闭

   `writer.close()`

   完整代码如下：

```python
def generate_tfrecord(annotation_dict, output_path, resize_size=None):
    num_valid_tf_example = 0
    # 创建一个write的实例对象
    writer = tf.python_io.TFRecordWriter(output_path)
    for image_path, label in annotation_dict.items():
        if not tf.gfile.GFile(image_path):
            print('%s does not exist.' % image_path)
            continue
        tf_example = create_tf_example(image_path, label, resize_size)
        writer.write(tf_example.SerializeToString())
        num_valid_tf_example += 1
        
        # 输出创建信息进行显示
        if num_valid_tf_example % 1000 == 0:
            print('Create %d TF_Example.' % num_valid_tf_example)
    writer.close()
    print('Total create TF_Example: %d' % num_valid_tf_example)
```

#### 3. TFrecord 读入

这部分到model.py 在介绍

#### 4. 数据处理与增强

这部分就是对数据进行多种操作，主要借鉴官方实例[Tensorflow/model](  https://github.com/tensorflow/models/blob/master/research/slim/preprocessing/vgg_preprocessing.py)

对图片文件进行预处理，包括中心化，去边缘，边缘扩展化 

```python
def preprocess_images(images, output_height, output_width, is_training=False,
                     resize_side_min=_RESIZE_SIDE_MIN,
                     resize_side_max=_RESIZE_SIDE_MAX,
                     border_expand=False, normalize=False,
                     preserving_aspect_ratio_resize=True):
    """Preprocesses the given image.

    Args:
        images: A `Tensor` representing a batch of images of arbitrary size.
        output_height: The height of the image after preprocessing.
        output_width: The width of the image after preprocessing.
        is_training: `True` if we're preprocessing the image for training and
            `False` otherwise.
        resize_side_min: The lower bound for the smallest side of the image 
            for aspect-preserving resizing. If `is_training` is `False`, then 
            this value is used for rescaling.
        resize_side_max: The upper bound for the smallest side of the image 
            for aspect-preserving resizing. If `is_training` is `False`, this 
            value is ignored. Otherwise, the resize side is sampled from
            [resize_size_min, resize_size_max].

    Returns:
        A  batch of preprocessed images.
    """
    images = tf.cast(images, tf.float32)
    def _preprocess_image(image):
        return preprocess_image(image, output_height, output_width,
                                is_training, resize_side_min,
                                resize_side_max, border_expand, normalize,
                                preserving_aspect_ratio_resize)
    return tf.map_fn(_preprocess_image, elems=images)
```

当然也可以自己进行实现