# PULC Recognition Model of Vehicle Attribute

------

## Catalogue

- [1. Introduction](#1)
- [2. Quick Start](#2)
    - [2.1 PaddlePaddle Installation](#2.1)
    - [2.2 PaddleClas Installation](#2.2)
    - [2.3 Prediction](#2.3)
- [3. Training, Evaluation and Inference](#3)
    - [3.1 Installation](#3.1)
    - [3.2 Dataset](#3.2)
      - [3.2.1 Dataset Introduction](#3.2.1)
      - [3.2.2 Getting Dataset](#3.2.2)
    - [3.3 Training](#3.3)
    - [3.4 Evaluation](#3.4)
    - [3.5 Inference](#3.5)
- [4. Model Compression](#4)
  - [4.1 SKL-UGI Knowledge Distillation](#4.1)
    - [4.1.1 Teacher Model Training](#4.1.1)
    - [4.1.2 Knowledge Distillation Training](#4.1.2)
- [5. SHAS](#5)
- [6. Inference Deployment](#6)
  - [6.1 Getting Paddle Inference Model](#6.1)
    - [6.1.1 Exporting Paddle Inference Model](#6.1.1)
    - [6.1.2 Downloading Inference Model](#6.1.2)
  - [6.2 Prediction with Python](#6.2)
    - [6.2.1 Image Prediction](#6.2.1)
    - [6.2.2 Images Prediction](#6.2.2)
  - [6.3 Deployment with C++](#6.3)
  - [6.4 Deployment as Service](#6.4)
  - [6.5 Deployment on Mobile](#6.5)
  - [6.6 Converting To ONNX and Deployment](#6.6)

<a name="1"></a>

## 1. Introduction

This case provides a way for users to quickly build a lightweight, high-precision and practical classification model of vehicle attribute using PaddleClas PULC (Practical Ultra Lightweight image Classification). The model can be widely used in Vehicle identification, road monitoring and other scenarios.

The following table lists the relevant indicators of the model. The first three lines means that using Res2Net200_vd_26w_4s, ResNet50 and MobileNetV3_small_x0_35 as the backbone to training. The fourth to seventh lines means that the backbone is replaced by PPLCNet, additional use of EDA strategy and additional use of EDA strategy and SKL-UGI knowledge distillation strategy.


| Backbone | mA（%） | Latency(ms) | Size(M) | Training Strategy |
|-------|-----------|----------|---------------|---------------|
| Res2Net200_vd_26w_4s  | 91.36 | 79.46  | 293 | using ImageNet pretrained  |
| ResNet50  | 89.98 | 12.83  | 92 | using ImageNet pretrained |
| MobileNetV3_small_x0_35  | 87.41 | 2.91  | 2.8 | using ImageNet pretrained |
| PPLCNet_x1_0  | 89.57 | 2.36  | 7.2 | using ImageNet pretrained |
| PPLCNet_x1_0  | 90.07 | 2.36  | 7.2 | using SSLD pretrained  |
| PPLCNet_x1_0  | 90.59 | 2.36  | 7.2 | using SSLD pretrained + EDA strategy|
| <b>PPLCNet_x1_0<b>  | <b>90.81<b> | <b>2.36<b>  | <b>7.2<b> | using SSLD pretrained + EDA strategy + SKL-UGI knowledge distillation strategy|


It can be seen from the table that the ma metric is higher when the backbone is Res2Net200_vd_26w_4s, but the inference speed is slower. After replacing the backbone with the lightweight model MobileNetV3_small_x0_35, the speed can be greatly improved, but the ma metric drops significantly. When the backbone is replaced by PPLCNet_x1_0, the ma metric is increased by 2 percentage points, and the speed is also increased by about 23%. On this basis, after using the SSLD pre-training model, the ma metric can be improved by about 0.5 percentage points without changing the inference speed. Further, when the EDA strategy is integrated, the ma metric can be improved by another 0.52 percentage points. Finally, using After SKL-UGI knowledge distillation, the ma metric can continue to improve by 0.23 percentage points. At this time, the ma metric of PPLCNet_x1_0 is only 0.55 percentage points away from Res2Net200_vd_26w_4s, but it is 32 times faster. The training method and deployment instructions of PULC will be introduced in detail below.


**Note**:

* The Latency is tested on Intel(R) Xeon(R) Gold 6148 CPU @ 2.40GHz. The MKLDNN is enabled and the number of threads is 10.
* About PP-LCNet, please refer to [PP-LCNet Introduction](../models/PP-LCNet_en.md) and [PP-LCNet Paper](https://arxiv.org/abs/2109.15099).

<a name="2"></a>

## 2. Quick Start

<a name="2.1"></a>  

### 2.1 PaddlePaddle Installation

- Run the following command to install if CUDA9 or CUDA10 is available.

```bash
python3 -m pip install paddlepaddle-gpu -i https://mirror.baidu.com/pypi/simple
```

- Run the following command to install if GPU device is unavailable.

```bash
python3 -m pip install paddlepaddle -i https://mirror.baidu.com/pypi/simple
```

Please refer to [PaddlePaddle Installation](https://www.paddlepaddle.org.cn/install/quick?docurl=/documentation/docs/en/install/pip/linux-pip_en.html) for more information about installation, for examples other versions.

<a name="2.2"></a>  

### 2.2 PaddleClas wheel Installation

The command of PaddleClas installation as bellow:

```bash
pip3 install paddleclas
```

<a name="2.3"></a>

### 2.3 Prediction

First, please click [here](https://paddleclas.bj.bcebos.com/data/PULC/pulc_demo_imgs.zip) to download and unzip to get the test demo images.


* Prediction with CLI

```bash
paddleclas --model_name=vehicle_attribute --infer_imgs=pulc_demo_imgs/vehicle_attribute/0002_c002_00030670_0.jpg
```

Results:
```
>>> result
attributes: Color: (yellow, prob: 0.9893476963043213), Type: (hatchback, prob: 0.9734097719192505), output: [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0], filename: pulc_demo_imgs/vehicle_attribute/0002_c002_00030670_0.jpg
Predict complete!
```

**Note**: If you want to test other images, only need to specify the `--infer_imgs` argument, and the directory containing images is also supported.

* Prediction in Python

```python
import paddleclas
model = paddleclas.PaddleClas(model_name="vehicle_attribute")
result = model.predict(input_data="pulc_demo_imgs/vehicle_attribute/0002_c002_00030670_0.jpg")
print(next(result))
```

**Note**: The `result` returned by `model.predict()` is a generator, so you need to use the `next()` function to call it or `for` loop to loop it. And it will predict with `batch_size` size batch and return the prediction results when called. The default `batch_size` is 1, and you also specify the `batch_size` when instantiating, such as `model = paddleclas.PaddleClas(model_name="vehicle_attribute",  batch_size=2)`. The result of demo above:

```
>>> result
[{'attributes': 'Color: (yellow, prob: 0.9893476963043213), Type: (hatchback, prob: 0.9734097719192505)', 'output': [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0], 'filename': 'pulc_demo_imgs/vehicle_attribute/0002_c002_00030670_0.jpg'}]
```

**Note**: The value index of `output` is 0-9 to indicate the color attribute, and the corresponding colors are: yellow, orange, green, gray, red, blue, white,  golden, brown, black; the index is 10-18 to represent the model attributes, the corresponding models are sedan, suv, van, hatchback, mpv, pickup, bus, truck, estate.

<a name="3"></a>

## 3. Training, Evaluation and Inference

<a name="3.1"></a>  

### 3.1 Installation

Please refer to [Installation](../installation/install_paddleclas_en.md) to get the description about installation.

<a name="3.2"></a>

### 3.2 Dataset

<a name="3.2.1"></a>

#### 3.2.1 Dataset Introduction

The data used in this case is the [pa100k dataset](https://www.v7labs.com/open-datasets/pa-100k).

<a name="3.2.2"></a>  

#### 3.2.2 Getting Dataset


Part of the data visualization is shown below.

<div align="center">
<img src="../../images/PULC/docs/vehicle_attribute_data_demo.png"  width = "500" />
</div>

First, apply for and download data from [VeRi dataset official website](https://www.v7labs.com/open-datasets/veri-dataset), put it in the `dataset` directory of PaddleClas, the dataset directory name is `VeRi `, use the following command to enter the folder.


```shell
cd PaddleClas/dataset/VeRi/
```

Then use the following code to convert the label (you can execute the following command in the python terminal, or you can write it to a file and run the file using `python3 convert.py`).

```python
import os
from xml.dom.minidom import parse

vehicleids = []

def convert_annotation(input_fp, output_fp):
    in_file = open(input_fp)
    list_file = open(output_fp, 'w')
    tree = parse(in_file)

    root = tree.documentElement

    for item in root.getElementsByTagName("Item"):  
        label = ['0', '0', '0', '0', '0', '0', '0', '0', '0', '0', '0', '0', '0', '0', '0', '0', '0', '0', '0']
        if item.hasAttribute("imageName"):
            name = item.getAttribute("imageName")
        if item.hasAttribute("vehicleID"):
            vehicleid = item.getAttribute("vehicleID")
            if vehicleid not in vehicleids :
                vehicleids.append(vehicleid)
            vid = vehicleids.index(vehicleid)
        if item.hasAttribute("colorID"):
            colorid = int (item.getAttribute("colorID"))
            label[colorid-1] = '1'
        if item.hasAttribute("typeID"):
            typeid = int (item.getAttribute("typeID"))
            label[typeid+9] = '1'
        label = ','.join(label)
        list_file.write(os.path.join('image_train', name)  + "\t" + label + "\n")

    list_file.close()

convert_annotation('train_label.xml', 'train_list.txt')  #imagename vehiclenum colorid typeid
convert_annotation('test_label.xml', 'test_list.txt')
```


After executing the above command, the `VeRi` directory has the following data:

```
VeRi
├── image_train
│   ├── 0001_c001_00016450_0.jpg
│   ├── 0001_c001_00016460_0.jpg
│   ├── 0001_c001_00016470_0.jpg
...
├── image_test
│   ├── 0002_c002_00030600_0.jpg
│   ├── 0002_c002_00030605_1.jpg
│   ├── 0002_c002_00030615_1.jpg
...
...
├── train_list.txt
├── test_list.txt
├── train_label.xml
├── test_label.xml
```

where `train/` and `test/` are the training set and validation set, respectively. `train_list.txt` and `test_list.txt` are the converted label files for training and validation sets, respectively.


<a name="3.3"></a>

### 3.3 Training

The details of training config in `./ppcls/configs/PULC/vehicle_attribute/PPLCNet_x1_0.yaml`. The command about training as follows:

```shell
export CUDA_VISIBLE_DEVICES=0,1,2,3
python3 -m paddle.distributed.launch \
    --gpus="0,1,2,3" \
    tools/train.py \
        -c ./ppcls/configs/PULC/vehicle_attribute/PPLCNet_x1_0.yaml
```

The best metric for the validation set is around `90.59%` (the dataset is small and generally fluctuates around 0.3%).


<a name="3.4"></a>

### 3.4 Evaluation

After training, you can use the following commands to evaluate the model.

```bash
python3 tools/eval.py \
    -c ./ppcls/configs/PULC/vehicle_attribute/PPLCNet_x1_0.yaml \
    -o Global.pretrained_model="output/PPLCNet_x1_0/best_model"
```

Among the above command, the argument `-o Global.pretrained_model="output/PPLCNet_x1_0/best_model"` specify the path of the best model weight file. You can specify other path if needed.

<a name="3.5"></a>

### 3.5 Inference

After training, you can use the model that trained to infer. Command is as follow:

```python
python3 tools/infer.py \
    -c ./ppcls/configs/PULC/vehicle_attribute/PPLCNet_x1_0.yaml \
    -o Global.pretrained_model=output/DistillationModel/best_model
```

The results:

```
[{'attr': 'Color: (yellow, prob: 0.9893478155136108), Type: (hatchback, prob: 0.9734100103378296)', 'pred': [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0], 'file_name': './deploy/images/PULC/vehicle_attribute/0002_c002_00030670_0.jpg'}]
```

**Note**:

* Among the above command, argument `-o Global.pretrained_model="output/PPLCNet_x1_0/best_model"` specify the path of the best model weight file. You can specify other path if needed.
* The default test image is `./deploy/images/PULC/vehicle_attribute/0002_c002_00030670_0.jpg`. And you can test other image, only need to specify the argument `-o Infer.infer_imgs=path_to_test_image`.

<a name="4"></a>

## 4. Model Compression

<a name="4.1"></a>

### 4.1 SKL-UGI Knowledge Distillation

SKL-UGI is a simple but effective knowledge distillation algrithem proposed by PaddleClas.

<!-- todo -->
<!-- Please refer to [SKL-UGI](../advanced_tutorials/distillation/distillation_en.md) for more details. -->

<a name="4.1.1"></a>

#### 4.1.1 Teacher Model Training

Training the teacher model with hyperparameters specified in `ppcls/configs/PULC/vehicle_attribute/PPLCNet_x1_0.yaml`. The command is as follow:

```shell
export CUDA_VISIBLE_DEVICES=0,1,2,3
python3 -m paddle.distributed.launch \
    --gpus="0,1,2,3" \
    tools/train.py \
        -c ./ppcls/configs/PULC/vehicle_attribute/PPLCNet_x1_0.yaml \
        -o Arch.name=ResNet101_vd
```

The best metric for the validation set is around `91.60%`. The best teacher model weight would be saved in file `output/ResNet101_vd/best_model.pdparams`.

<a name="4.1.2"></a>

#### 4.1.2 Knowledge Distillation Training

The training strategy, specified in training config file `ppcls/configs/PULC/vehicle_attribute/PPLCNet_x1_0_distillation.yaml`, the teacher model is `ResNet101_vd`, the student model is `PPLCNet_x1_0`. The command is as follow:

```shell
export CUDA_VISIBLE_DEVICES=0,1,2,3
python3 -m paddle.distributed.launch \
    --gpus="0,1,2,3" \
    tools/train.py \
        -c ./ppcls/configs/PULC/vehicle_attribute/PPLCNet_x1_0_distillation.yaml \
        -o Arch.models.0.Teacher.pretrained=output/ResNet101_vd/best_model
```

The best metric for the validation set is around `90.81%`. The best student model weight would be saved in file `output/DistillationModel/best_model_student.pdparams`.

<a name="5"></a>

## 5. Hyperparameters Searching

The hyperparameters used by [3.2 section](#3.2) and [4.1 section](#4.1) are according by `Hyperparameters Searching` in PaddleClas. If you want to get better results on your own dataset, you can refer to [Hyperparameters Searching](PULC_train_en.md#4) to get better hyperparameters.

**Note**: This section is optional. Because the search process will take a long time, you can selectively run according to your specific. If not replace the dataset, you can ignore this section.

<a name="6"></a>

## 6. Inference Deployment

<a name="6.1"></a>

### 6.1 Getting Paddle Inference Model

Paddle Inference is the original Inference Library of the PaddlePaddle, provides high-performance inference for server deployment. And compared with  directly based on the pretrained model, Paddle Inference can use tools to accelerate prediction, so as to achieve better inference performance. Please refer to [Paddle Inference](https://www.paddlepaddle.org.cn/documentation/docs/zh/guides/infer/inference/inference_cn.html) for more information.

Paddle Inference need Paddle Inference Model to predict. Two process provided to get Paddle Inference Model. If want to use the provided by PaddleClas, you can download directly, click [Downloading Inference Model](#6.1.2).

<a name="6.1.1"></a>

### 6.1.1 Exporting Paddle Inference Model

The command about exporting Paddle Inference Model is as follow:

```bash
python3 tools/export_model.py \
    -c ./ppcls/configs/PULC/vehicle_attribute/PPLCNet_x1_0.yaml \
    -o Global.pretrained_model=output/DistillationModel/best_model_student \
    -o Global.save_inference_dir=deploy/models/PPLCNet_x1_0_vehicle_attribute_infer
```

After running above command, the inference model files would be saved in `PPLCNet_x1_0_vehicle_attribute_infer`, as shown below:

```
└── PPLCNet_x1_0_vehicle_attribute_infer
    ├── inference.pdiparams
    ├── inference.pdiparams.info
    └── inference.pdmodel
```

**Note**: The best model is from knowledge distillation training. If knowledge distillation training is not used, the best model would be saved in `output/PPLCNet_x1_0/best_model.pdparams`.

<a name="6.1.2"></a>

### 6.1.2 Downloading Inference Model

You can also download directly.

```
cd deploy/models
# download the inference model and decompression
wget https://paddleclas.bj.bcebos.com/models/PULC/vehicle_attribute_infer.tar && tar -xf vehicle_attribute_infer.tar
```

After decompression, the directory `models` should be shown below.

```
├── vehicle_attribute_infer
│   ├── inference.pdiparams
│   ├── inference.pdiparams.info
│   └── inference.pdmodel
```

<a name="6.2"></a>

### 6.2 Prediction with Python

<a name="6.2.1"></a>  

#### 6.2.1 Image Prediction

Return the directory `deploy`:

```
cd ../
```

Run the following command to classify whether there are human in the image `../images/PULC/vehicle_attribute/0002_c002_00030670_0.jpg`.

```shell
# Use the following command to predict with GPU.
python3.7 python/predict_cls.py -c configs/PULC/vehicle_attribute/inference_vehicle_attribute.yaml -o Global.use_gpu=True
# Use the following command to predict with CPU.
python3.7 python/predict_cls.py -c configs/PULC/vehicle_attribute/inference_vehicle_attribute.yaml -o Global.use_gpu=False
```

The prediction results:

```
0002_c002_00030670_0.jpg:     {'attributes': 'Color: (yellow, prob: 0.9893478155136108), Type: (hatchback, prob: 0.9734099507331848)', 'output': [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0]}
```


<a name="6.2.2"></a>  

#### 6.2.2 Images Prediction

If you want to predict images in directory, please specify the argument `Global.infer_imgs` as directory path by `-o Global.infer_imgs`. The command is as follow.

```shell
# Use the following command to predict with GPU. If want to replace with CPU, you can add argument -o Global.use_gpu=False
python3.7 python/predict_cls.py -c configs/PULC/vehicle_attribute/inference_vehicle_attribute.yaml -o Global.infer_imgs="./images/PULC/vehicle_attribute/"
```

All prediction results will be printed, as shown below.

```
0002_c002_00030670_0.jpg:     {'attributes': 'Color: (yellow, prob: 0.9893476963043213), Type: (hatchback, prob: 0.9734097719192505)', 'output': [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0]}
0014_c012_00040750_0.jpg:     {'attributes': 'Color: (red, prob: 0.999872088432312), Type: (sedan, prob: 0.999976634979248)', 'output': [0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0]}
```

Among the prediction results above, `someone` means that there is a human in the image, `nobody` means that there is no human in the image.

<a name="6.3"></a>

### 6.3 Deployment with C++

PaddleClas provides an example about how to deploy with C++. Please refer to [Deployment with C++](../inference_deployment/cpp_deploy_en.md).

<a name="6.4"></a>

### 6.4 Deployment as Service

Paddle Serving is a flexible, high-performance carrier for machine learning models, and supports different protocol, such as RESTful, gRPC, bRPC and so on, which provides different deployment solutions for a variety of heterogeneous hardware and operating system environments. Please refer [Paddle Serving](https://github.com/PaddlePaddle/Serving) for more information.

PaddleClas provides an example about how to deploy as service by Paddle Serving. Please refer to [Paddle Serving Deployment](../inference_deployment/classification_serving_deploy_en.md).

<a name="6.5"></a>

### 6.5 Deployment on Mobile

Paddle-Lite is an open source deep learning framework that designed to make easy to perform inference on mobile, embeded, and IoT devices. Please refer to [Paddle-Lite](https://github.com/PaddlePaddle/Paddle-Lite) for more information.

PaddleClas provides an example of how to deploy on mobile by Paddle-Lite. Please refer to [Paddle-Lite deployment](../inference_deployment/paddle_lite_deploy_en.md).

<a name="6.6"></a>

### 6.6 Converting To ONNX and Deployment

Paddle2ONNX support convert Paddle Inference model to ONNX model. And you can deploy with ONNX model on different inference engine, such as TensorRT, OpenVINO, MNN/TNN, NCNN and so on. About Paddle2ONNX details, please refer to [Paddle2ONNX](https://github.com/PaddlePaddle/Paddle2ONNX).

PaddleClas provides an example of how to convert Paddle Inference model to ONNX model by paddle2onnx toolkit and predict by ONNX model. You can refer to [paddle2onnx](../../../deploy/paddle2onnx/readme_en.md) for deployment details.
