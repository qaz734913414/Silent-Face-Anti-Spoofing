![静默活体检测](https://github.com/minivision-ai/Silent-Face-Anti-Spoofing/blob/master/images/logo.jpg)  
# 静默活体检测 (Silent-Face-Anti-Spoofing)   
该项目为[小视科技](https://www.minivision.cn/)的静默活体检测项目,您可以扫描下方的二维码获取APK(提取码为**phwd**),在安卓端安装APK,体验静默活体的检测效果.   
<img src="https://github.com/minivision-ai/Silent-Face-Anti-Spoofing/blob/master/images/静默活体APK.png" width="200" height="200" align=center />  
## 简介
在本工程中我们开源了活体模型训练架构，数据预处理方法，模型训练和测试脚本以及开源的APK供大家测试使用。  

活体检测技术主要是判别机器前出现的人脸是真实还是伪造的，其中借助其他媒介呈现的人脸都可以定义为虚假的人脸，包括打印的纸质照片、电子产品的显示屏幕、硅胶面具、立体的3D人像等。目前主流的活体解决方案分为配合式活体检测和非配合式活体检测（静默活体检测）。配合式活体检测需要用户根据提示完成指定的动作，然后再进行活体校验，静默活体则在用户无感的情况下直接进行活体校验。 

因傅里叶频谱图一定程度上能够反应真假脸在频域的差异,因此我们采用了一种基于傅里叶频谱图辅助监督的静默活体检测方法, 模型架构由分类主分支和傅里叶频谱图辅助监督分支构成，整体架构如下图所示：  
![整体架构图](https://github.com/minivision-ai/Silent-Face-Anti-Spoofing/blob/master/images/framework.png)  

使用自研的模型剪枝方法，将MobileFaceNet的Flops从0.224G降低待了0.081G，在精度损失不大的情况下,明显提升模型的性能(降低计算量与参数量).  

|Model|FLOPs|Params|
| :------:|:-----:|:-----:| 
|MobileFaceNet|0.224G|0.991M|
|MiniFASNetV1|0.081G|0.414M|
|MiniFASNetV2|0.081G|0.435M|

## APK

### Demo
<img src="https://github.com/minivision-ai/Silent-Face-Anti-Spoofing/blob/master/images/demo.gif" width="300" height="400"/>  
 
### 关键指标  
| Model|FLOPs|Speed| FPR | TPR |备注 |
| :------:|:-----:|:-----:| :----: | :----: | :----: |
|   APK模型 |84M| 20ms | 1e-5|97.8%| 开源|
| 高精度模型 |162M| 40ms| 1e-5 |99.7%| 未开源 |

### 测试方法  
- 显示信息:速度(ms), 置信度(0~1)以及活体检测结果(真脸or假脸)
- 点击右上角图标可设置阈值,如果置信度大于阈值,为真脸,否则为假脸  

### 测试须知 
- 所有测试图片必须通过摄像头采集得到，否则不符合正常场景使用规范，算法效果也无法保证。
- 因为RGB静默活体对摄像头型号和使用场景鲁棒性受限，所以实际使用体验会有一定差异。
- 测试时，应保证有完整的人脸出现在视图中，并且人脸旋转角与竖直方向小于30度（符合正常刷脸场景），否则影响体验。　　

**已测试手机型号**

|手机型号|麒麟990 5G|麒麟990 |骁龙845 |麒麟810 |RK3288 |
| :------:|:-----:|:-----:|:-----:|:-----:|:-----:|
|速度/ms|19|23|24|25|90|

## 工程
### 安装依赖库  
```
pip install -r requirements.txt
```
### Clone
```
git clone https://github.com/minivision-ai/Silent-Face-Anti-Spoofing  
cd Silent-Face-Anti-Spoofing
```  
### 数据预处理
1.将训练集分为3类,将相同类别的图片放入一个文件夹;  
2.因采用多尺度模型融合的方法,分别用原图和不同的patch训练模型,所以将数据分为原图和基于原图的patch;  
- 原图(org_1_height**x**width),直接将原图resize到固定尺寸(width, height),如图1所示;  
- 基于原图的patch(scale_height**x**width),采用人脸检测器人脸,获取人脸框,按照一定比例(scale)对人脸框进行扩边，为了保证模型的输入尺寸的一致性，将人脸框区域resize到固定尺寸(width, height),图2-4分别显示了scale为1,2.7和4的patch样例;  
![patch demo](https://github.com/minivision-ai/Silent-Face-Anti-Spoofing/blob/master/images/patch_demo.png)  

3.采用傅里叶频谱图作为辅助监督,训练集图片均需生成对应的傅里叶频谱图.
**数据集的目录结构如下所示**
```
├── datasets
    └── RGB_Images
        ├── org_1_80x60
            ├── 0
            ├── 1
            └── 2
        ├── 1_80x80
        └── ...
```  
### 训练
```
python train.py --device_ids 0  --patch_info your_patch
```  
### 测试
 ./resources/anti_spoof_models 活体检测的融合模型  
 ./resources/detection_model 检测器模型  
 ./images/sample 测试图片  
 **注意:测试图片需从宽高比为3:4的视频流中截取**   
 ```
 python test.py --image_name your_image_name
 ```  
## Q&A
**Q:**  数据集开源吗?  
A: 不开源  
**Q:** 为什么测试图片需从宽高比为3:4的视频流中截取?  
A: 因为活体的应用场景是实时视频流,该活体模型适配480x640的实时视频流  
 
## 参考  
- 检测器 [RetinaFace](https://github.com/deepinsight/insightface/tree/master/RetinaFace)
