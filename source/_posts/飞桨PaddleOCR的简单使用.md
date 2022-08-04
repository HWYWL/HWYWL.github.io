---
title: 飞桨PaddleOCR的简单使用
date: 2022-08-04 18:02:30
tags: 
 - 其他
categories: 
 - 其他
keywords: "其他"
description: 飞桨PaddleOCR的简单使用
---

公司目前有一个需求是自动识别用户上传的银行卡和身份证，为了信息安全考虑没有使用第三方的API，而是自己搭建了一套，搭建使用如下。

# 环境配置
```shell
# 我使用是AnolisOS 8.5版本，一个Centos 8的衍生版本，完全兼容Centos 8。
# 安装开发者工具
dnf groupinstall 'development tools'

# 可通过查找不同版本安装
yum search python | grep -i devel

# 安装python-devel(注意python版本)
yum install -y python39-devel.x86_64

# 创建软连接
ln -s /usr/bin/python3.9 /usr/bin/python3
ln -s /usr/bin/python3.9 /usr/bin/python
ln -s /usr/bin/pip3.9 /usr/bin/pip3
ln -s /usr/bin/pip3.9 /usr/bin/pip

# 升级pip到最新版本，不然安装库会报错
python -m pip install –upgrade pip

# 安装libGL库
yum -y install mesa-libGL.x86_64

# 安装jdk1.8(可选)
yum install -y java-1.8.0-openjdk.x86_64

# 更新系统
yum update

# 安装paddlepaddle
python -m pip install paddlepaddle -i https://mirror.baidu.com/pypi/simple

# 安装PaddleOCR
pip install "paddleocr>=2.5.0"
```

# 测试
```python
from paddleocr import PaddleOCR, draw_ocr

# Paddleocr目前支持的多语言语种可以通过修改lang参数进行切换
# 例如`ch`, `en`, `fr`, `german`, `korean`, `japan`
ocr = PaddleOCR(use_angle_cls=True, lang="ch")
img_path = '/home/test.jpg'
result = ocr.ocr(img_path, cls=True)
for line in result:
    print(line)

# 显示结果
from PIL import Image

image = Image.open(img_path).convert('RGB')
boxes = [line[0] for line in result]
txts = [line[1][0] for line in result]
scores = [line[1][1] for line in result]
im_show = draw_ocr(image, boxes, txts, scores, font_path='/home/simfang.ttf')
im_show = Image.fromarray(im_show)
im_show.save('result.jpg')
```

说明：**simfang.ttf** 文件其实就是字库，我随便下载一个来用，下面是我下载的地址：

```
http://y.downyagt.com:7658/soft/simfang_downyi.com.zip
```