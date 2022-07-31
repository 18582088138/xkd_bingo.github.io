## How to optimizer Yolov5 by OpenVINO™

We use the [OpenVINO™](https://www.intel.com/content/www/us/en/developer/tools/openvino-toolkit/overview.html) to optimizer public models such as Yolov5, Resnet50.

The OpenVINO toolkit makes it simple to adopt and maintain your code. [Model Optimizer API](https://docs.openvino.ai/latest/openvino_docs_MO_DG_Deep_Learning_Model_Optimizer_DevGuide.html) parameters make it easier to convert your model and prepare it for inferencing. The [runtime (inference engine)](https://docs.openvino.ai/latest/openvino_docs_OV_UG_OV_Runtime_User_Guide.html) allows you to tune for performance by compiling the optimized network and managing inference operations on specific devices. 


### Project Introduction

This series of document is divided in 4 chapters introduces a BKM on how to convert Yolov5 from PyTorch to IR files using model optimizer tool , how to quantize the Yolov5 from FP32 to INT8 with minimum drop on precision and also provides method to accelerate the model development on CPU device and in the last section, an inference demo is showed for the deployment of YOLOv5. 
- Convert Yolov5 from PyTorch to IR by OpenVINO™
- Quantize Yolov5 from FP32 to INT8 
- Accuracy check the Yolov5 INT8 precision 
- Development Yolov5 on CPU device

### Pre-Requisite Environment 

The following are some verified version in environment requisite, not all supported versions.

| Pre-Requisite | Description |
| :-----------  | :---------  |
|OpenVINO™      |OpenVINO™ 2021.3, 2021.4, 2022.1, 2022.2 |
|Python         |Python 3.6.9 |
|ONNX           |ONNX 1.9.0   |
|PyTorch        |PyTorch 1.9.0|
|OS             |Ubuntu 18.04.2 LTS, 20.04|
|Kernel version |5.4.0-74-generic|
|Netron         |Netron 4.9.8 |

#### Convert Yolov5 from PyTorch to IR by OpenVINO™

##### Setup Yolov5 Environment
Run git command to clone Yolov5 repository, here we use release v5.0 and install requirement.
```markdown
$ git clone https://github.com/ultralytics/yolov5 -b v5.0
$ pip install -r requirements.txt
$ pip install onnx
```

#####  Convert PyTorch weights to ONNX weights
There are YOLOv5s, YOLOv5m, YOLOv5l, and YOLOv5x four models supported for YOLOv5, we use   YOLOv5s as example here, and other models are same. Run the below command to get **yolov5s.pt**
And convert it to ONNX weights.
```markdown
$ wget https://github.com/ultralytics/yolov5/releases/download/v5.0/yolov5s.pt
$ mkdir yolov5-v5
$ mv yolov5s.pt yolov5-v5
$ python3 models/export.py --weights yolov5-v5/yolov5s.pt --img 640 --batch 1
```

#### Convert ONNX to IR files
We can start to convert it to IR files now after getting ONNX weights file from the 
last section. At first, the output nodes need to be specified. There are 3 output 
nodes in YOLOv5. We can use Netron to visualize the YOLOv5 ONNX weights.

##### Specify Output Node of 80*80
```markdown
  We can search the keyword “Transpose” in Netron, and then the convolution node will be found, marked as rectangle shown in Figure 1. 
  After double clicking it, we can read the name “Conv_245” on the right panel of properties marked as oval. 
  Figure 1 shows the output node with size of 1x3x80x80x85, which is used to detect small objects. 
  We need to apply “Conv_245” of convolution node to specify the model optimizer parameters.
  
  **Note** Here the dimension 85 is equal to (4+1)*80, which stands for 4 coordinates for BBX, 1 confidence and 80 classes.
```
<div align=left ><img src="https://github.com/18582088138/xkd_bingo.github.io/blob/gh-pages/yolov5%20output%20node%20of%208x%20down%20sampling.png"/>
  
**Figure 1**: The Output Node of YOLOv5s, P3 after 8x Down Sampling for Small Object Detection
  
  
##### Specify Output Node of 40*40
```markdown
  Similarly, we can get the convolution node “Conv_261”. 
  Figure 2 shows the output node with size of 1x3x40x40x85, which is used to detect medium objects. 
  We need to apply “Conv_261” of convolution node to specify the model optimizer parameters
```
<div align=left ><img src="https://github.com/18582088138/xkd_bingo.github.io/blob/gh-pages/yolov5%20output%20node%20of%2016x%20down%20sampling.png"/>

**Figure 2**: The Output Node of YOLOv5s, P4 after 16x Down Sampling for Small Object Detection

##### Specify Output Node of 20*20
```markdown
  At last, we can get the convolution node “Conv_277”. 
  Figure 3 shows the output node with size of 1x3x20x20x85, which is used to detect large objects. 
  We need to apply “Conv_277” of convolution node to specify the model optimizer parameters
```
<div align=left ><img src="https://github.com/18582088138/xkd_bingo.github.io/blob/gh-pages/yolov5%20output%20node%20of%2032x%20down%20sampling.png"/>

**Figure 3**: The Output Node of YOLOv5s, P5 after 32x Down Sampling for Small Object Detection

Using MO tool inside OpenVINOTM  to convert ONNX to IR files
```markdown
$ python3 /opt/intel/openvino/deployment_tools/model_optimizer/mo.py --input_model yolov5-v5/yolov5s.onnx --model_name yolov5-v5/yolov5s -s 255 --reverse_input_channels --output Conv_245,Conv_261,Conv_277

```

### IR Model Performance Verification
The benchmark application loads the Inference Engine (SW) at run time and executes inferences on the specified hardware inference engine, (CPU, GPU or VPU). The benchmark application measures the time spent on actual inferencing (excluding any pre or post processing) and then reports on the inferences per second (or Frames Per Second).

Use this data to help you decide which hardware is best for your applications and solutions, or to plan your AI workload on the Intel computing already included in your solutions.

This topic describes usage of Python implementation of the Benchmark Tool, before running the Benchmark tool, setup environment variable and install the requirements
```markdown
$ source /opt/intel/openvino/bin/setupvars.sh

$ cd /opt/intel/openvino/deployment_tools/tools/benchmark_tool
$ pip install -r requirements.txt

```
As example, we show how to evaluate the YOLOv5s model performance with benchmark_app script
```markdown
$ python3 /opt/intel/openvino/deployment_tools/tools/benchmark_tool/benchmark_app.py \
-m yolov5s.xml \
-api async \
-nstreams 12 \
-nthreads 48 \
-d CPU
  
``` 
<div align=left ><img src="https://github.com/18582088138/xkd_bingo.github.io/blob/gh-pages/yolov5_benchmark_performance.png"/>
**Figure 4** :Yolov5s performance detail  

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/18582088138/xkd_bingo.github.io/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
