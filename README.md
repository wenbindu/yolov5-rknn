# 操作指南
> 本仓库主要为rk3588的yolov5模型训练，以及导出onnx，进而导出rknn，从而实现部署而构建。

## 基于Yolov5训练
> 主要基于 已修改版本的[yolov5仓库](https://github.com/airockchip/yolov5.git) 进行构建，其中主要包括了将silu函数修改为relu函数。本仓库将其仓库放置在了yolov5文件夹下。
1. 下载本仓库，基于Python3.8 创建venv环境，并基于固定某些包的版本的requirements文件进行依赖安装。
```
cd yolov5
pip install -r requirements-py38.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```
2. 采用coco数据集或者自定义数据集进行训练，关于 *data.yaml* 的修改，可以查看其他资料.
- 基于coco数据集合
```sh
python train.py --data coco.yaml --epochs 300 --weights '' --cfg yolov5n.yaml  --batch-size 128
                                                                  yolov5s                    64
                                                                 yolov5m                    40
                                                                 yolov5l                    24
                                                                 yolov5x                    16
```
- 基于特定数据集合
```sh
python train.py --img 640 --epochs 100 --data ds5/data.yaml --weights yolov5s.pt --batch-size 64
```
3. 采用训练好的pt模型进行检验测试。

```sh
python3 detect.py --source ./data/bus.jpg --weights runs/train/exp8/weights/best.pt
```

4. 导出模型，注意提供特定版本.

```
python export.py --weight runs/train/exp8/weights/best.pt --include onnx --rknpu rk3588 --img 640 --include onnx --opset 12
```

至此导出了可以进行rknn转换的onnx模型。


## 基于rknn_toolkit2 模型转换
> 此处基于docker镜像进行环境安装和部署[rknn_toolkit2_docker]。
#### 三个dockerfile 环境
1. *ubuntu_18_04_cp36*
2. **ubuntu_20_04_cp38**
3. *ubuntu_22_04_cp310*
#### 以ubuntu20.04 + python3.8 为例
1. 构建镜像并进入docker环境
```
docker build -f Dockerfile_ubuntu_20_04_for_cp38 -t rknn-tookit2:1.0.0-cp38 .  
docker images
docker run -it --rm --privileged -v /root/yolov5-rknn:/rknn-ws rknn-tookit2:1.0.0-cp38 /bin/bash
```
2. 进行模型转换
进入example 文件夹，同时将上一阶段训练导出的onnx拷贝到此处。本文主要在 `imgsz=640` 的基础上开发训练，如有不同，则需要自定义修改。
- 采用 [netron](https://github.com/lutzroeder/netron) 查看onnx结构。注意最后output前是否存在 **sigmoid**函数，会影响到后处理部分。
- 修改代码
> 根据代码中的注释, 修改 `test.py` 文件中部分代码。
- 执行转换和模拟推理。

```
python test.py
```

## 基于npu实现部署
因为部分操作尚未完结，等待后续补充...
