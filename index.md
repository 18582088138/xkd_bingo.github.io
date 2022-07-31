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

```markdown
##### 1.	Specify Output Node of 80*80
We can search the keyword “Transpose” in Netron, and then the convolution node will be found, marked as rectangle shown in Figure 1. After double clicking it, we can read the name “Conv_245” on the right panel of properties marked as oval. Figure 1 shows the output node with size of 1x3x80x80x85, which is used to detect small objects. We need to apply “Conv_245” of convolution node to specify the model optimizer parameters.
![avatar](https://github.com/18582088138/xkd_bingo.github.io/blob/gh-pages/yolov5%20output%20node%20of%208x%20down%20sampling.png "test")
```




Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [Basic writing and formatting syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/18582088138/xkd_bingo.github.io/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
