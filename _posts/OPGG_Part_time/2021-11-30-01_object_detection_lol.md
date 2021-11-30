---
title: "[OPGG] 인턴 연계 과정 - 미니맵 챔피언 인식"
excerpt: "인턴 연계 과정 공부 내용"
categories: 
  - OPGG_Part
tags: 
    - Python
    - OPGG_Part
    - yolov5
toc: TRUE
toc_sticky: TRUE
use_math: TRUE
---

# 오피지지 인턴 과정

오피지지 데이터 분석가 과정 교육을 수료하고 우수 교육생으로 한 달간 인턴 생활을 하게 되었습니다.

회사의 실무를 보면서 개인적으로 진행했던 과정에 대해 정리하고자 합니다.

본 내용들은 실제 인턴 생활에서 맡았던 업무와 관련 있다고 생각한 부분을 개인적으로 진행한 것 입니다.

실제 맡은 업무와는 상이할 수 있으니 참고 바랍니다.

## 미니맵 챔피언 인식

여기서는 `yolov5` 모델을 이용해서 리그 오브 레전드 게임 영상에서 미니맵의 챔피언을 인식해보겠습니다.

모델의 성능은 고려하지 않고 전체 과정을 간단하게 살펴보았습니다.

본 코드는 코랩으로 진행하였으며 혹시 직접 해보고 싶다면 경로 등에 유의하세요.

(특히나 저 같은 경우는 디렉토리를 변경하거나 절대경로를 많이 사용했습니다.)

### 1.환경 설정


```python
# drive mount
from google.colab import drive
drive.mount('/content/drive')
```

    Mounted at /content/drive
    

- 드라이브 마운트


```python
import os
import glob
import zipfile

import cv2
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from PIL import Image

import requests

import warnings
warnings.filterwarnings("ignore")
```


```python
# change directory
%cd /content

# yolov5 git clone
!git clone https://github.com/ultralytics/yolov5.git
```

    /content
    Cloning into 'yolov5'...
    remote: Enumerating objects: 10081, done.[K
    remote: Total 10081 (delta 0), reused 0 (delta 0), pack-reused 10081[K
    Receiving objects: 100% (10081/10081), 10.40 MiB | 24.54 MiB/s, done.
    Resolving deltas: 100% (6990/6990), done.
    


```python
# package for yolov5
%cd /content/yolov5/
!pip install -r requirements.txt
```

    /content/yolov5
    Requirement already satisfied: matplotlib>=3.2.2 in /usr/local/lib/python3.7/dist-packages (from -r requirements.txt (line 4)) (3.2.2)
    Requirement already satisfied: numpy>=1.18.5 in /usr/local/lib/python3.7/dist-packages (from -r requirements.txt (line 5)) (1.19.5)
    Requirement already satisfied: opencv-python>=4.1.2 in /usr/local/lib/python3.7/dist-packages (from -r requirements.txt (line 6)) (4.1.2.30)
    Requirement already satisfied: Pillow>=7.1.2 in /usr/local/lib/python3.7/dist-packages (from -r requirements.txt (line 7)) (7.1.2)
    Collecting PyYAML>=5.3.1
      Downloading PyYAML-6.0-cp37-cp37m-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_12_x86_64.manylinux2010_x86_64.whl (596 kB)
    [K     |████████████████████████████████| 596 kB 5.4 MB/s 
    [?25hRequirement already satisfied: requests>=2.23.0 in /usr/local/lib/python3.7/dist-packages (from -r requirements.txt (line 9)) (2.23.0)
    Requirement already satisfied: scipy>=1.4.1 in /usr/local/lib/python3.7/dist-packages (from -r requirements.txt (line 10)) (1.4.1)
    Requirement already satisfied: torch>=1.7.0 in /usr/local/lib/python3.7/dist-packages (from -r requirements.txt (line 11)) (1.10.0+cu111)
    Requirement already satisfied: torchvision>=0.8.1 in /usr/local/lib/python3.7/dist-packages (from -r requirements.txt (line 12)) (0.11.1+cu111)
    Requirement already satisfied: tqdm>=4.41.0 in /usr/local/lib/python3.7/dist-packages (from -r requirements.txt (line 13)) (4.62.3)
    Requirement already satisfied: tensorboard>=2.4.1 in /usr/local/lib/python3.7/dist-packages (from -r requirements.txt (line 16)) (2.7.0)
    Requirement already satisfied: pandas>=1.1.4 in /usr/local/lib/python3.7/dist-packages (from -r requirements.txt (line 20)) (1.1.5)
    Requirement already satisfied: seaborn>=0.11.0 in /usr/local/lib/python3.7/dist-packages (from -r requirements.txt (line 21)) (0.11.2)
    Collecting thop
      Downloading thop-0.0.31.post2005241907-py3-none-any.whl (8.7 kB)
    Requirement already satisfied: python-dateutil>=2.1 in /usr/local/lib/python3.7/dist-packages (from matplotlib>=3.2.2->-r requirements.txt (line 4)) (2.8.2)
    Requirement already satisfied: pyparsing!=2.0.4,!=2.1.2,!=2.1.6,>=2.0.1 in /usr/local/lib/python3.7/dist-packages (from matplotlib>=3.2.2->-r requirements.txt (line 4)) (3.0.6)
    Requirement already satisfied: cycler>=0.10 in /usr/local/lib/python3.7/dist-packages (from matplotlib>=3.2.2->-r requirements.txt (line 4)) (0.11.0)
    Requirement already satisfied: kiwisolver>=1.0.1 in /usr/local/lib/python3.7/dist-packages (from matplotlib>=3.2.2->-r requirements.txt (line 4)) (1.3.2)
    Requirement already satisfied: idna<3,>=2.5 in /usr/local/lib/python3.7/dist-packages (from requests>=2.23.0->-r requirements.txt (line 9)) (2.10)
    Requirement already satisfied: urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1 in /usr/local/lib/python3.7/dist-packages (from requests>=2.23.0->-r requirements.txt (line 9)) (1.24.3)
    Requirement already satisfied: chardet<4,>=3.0.2 in /usr/local/lib/python3.7/dist-packages (from requests>=2.23.0->-r requirements.txt (line 9)) (3.0.4)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.7/dist-packages (from requests>=2.23.0->-r requirements.txt (line 9)) (2021.10.8)
    Requirement already satisfied: typing-extensions in /usr/local/lib/python3.7/dist-packages (from torch>=1.7.0->-r requirements.txt (line 11)) (3.10.0.2)
    Requirement already satisfied: grpcio>=1.24.3 in /usr/local/lib/python3.7/dist-packages (from tensorboard>=2.4.1->-r requirements.txt (line 16)) (1.42.0)
    Requirement already satisfied: protobuf>=3.6.0 in /usr/local/lib/python3.7/dist-packages (from tensorboard>=2.4.1->-r requirements.txt (line 16)) (3.17.3)
    Requirement already satisfied: werkzeug>=0.11.15 in /usr/local/lib/python3.7/dist-packages (from tensorboard>=2.4.1->-r requirements.txt (line 16)) (1.0.1)
    Requirement already satisfied: absl-py>=0.4 in /usr/local/lib/python3.7/dist-packages (from tensorboard>=2.4.1->-r requirements.txt (line 16)) (0.12.0)
    Requirement already satisfied: google-auth<3,>=1.6.3 in /usr/local/lib/python3.7/dist-packages (from tensorboard>=2.4.1->-r requirements.txt (line 16)) (1.35.0)
    Requirement already satisfied: setuptools>=41.0.0 in /usr/local/lib/python3.7/dist-packages (from tensorboard>=2.4.1->-r requirements.txt (line 16)) (57.4.0)
    Requirement already satisfied: tensorboard-plugin-wit>=1.6.0 in /usr/local/lib/python3.7/dist-packages (from tensorboard>=2.4.1->-r requirements.txt (line 16)) (1.8.0)
    Requirement already satisfied: markdown>=2.6.8 in /usr/local/lib/python3.7/dist-packages (from tensorboard>=2.4.1->-r requirements.txt (line 16)) (3.3.6)
    Requirement already satisfied: wheel>=0.26 in /usr/local/lib/python3.7/dist-packages (from tensorboard>=2.4.1->-r requirements.txt (line 16)) (0.37.0)
    Requirement already satisfied: tensorboard-data-server<0.7.0,>=0.6.0 in /usr/local/lib/python3.7/dist-packages (from tensorboard>=2.4.1->-r requirements.txt (line 16)) (0.6.1)
    Requirement already satisfied: google-auth-oauthlib<0.5,>=0.4.1 in /usr/local/lib/python3.7/dist-packages (from tensorboard>=2.4.1->-r requirements.txt (line 16)) (0.4.6)
    Requirement already satisfied: pytz>=2017.2 in /usr/local/lib/python3.7/dist-packages (from pandas>=1.1.4->-r requirements.txt (line 20)) (2018.9)
    Requirement already satisfied: six in /usr/local/lib/python3.7/dist-packages (from absl-py>=0.4->tensorboard>=2.4.1->-r requirements.txt (line 16)) (1.15.0)
    Requirement already satisfied: rsa<5,>=3.1.4 in /usr/local/lib/python3.7/dist-packages (from google-auth<3,>=1.6.3->tensorboard>=2.4.1->-r requirements.txt (line 16)) (4.7.2)
    Requirement already satisfied: pyasn1-modules>=0.2.1 in /usr/local/lib/python3.7/dist-packages (from google-auth<3,>=1.6.3->tensorboard>=2.4.1->-r requirements.txt (line 16)) (0.2.8)
    Requirement already satisfied: cachetools<5.0,>=2.0.0 in /usr/local/lib/python3.7/dist-packages (from google-auth<3,>=1.6.3->tensorboard>=2.4.1->-r requirements.txt (line 16)) (4.2.4)
    Requirement already satisfied: requests-oauthlib>=0.7.0 in /usr/local/lib/python3.7/dist-packages (from google-auth-oauthlib<0.5,>=0.4.1->tensorboard>=2.4.1->-r requirements.txt (line 16)) (1.3.0)
    Requirement already satisfied: importlib-metadata>=4.4 in /usr/local/lib/python3.7/dist-packages (from markdown>=2.6.8->tensorboard>=2.4.1->-r requirements.txt (line 16)) (4.8.2)
    Requirement already satisfied: zipp>=0.5 in /usr/local/lib/python3.7/dist-packages (from importlib-metadata>=4.4->markdown>=2.6.8->tensorboard>=2.4.1->-r requirements.txt (line 16)) (3.6.0)
    Requirement already satisfied: pyasn1<0.5.0,>=0.4.6 in /usr/local/lib/python3.7/dist-packages (from pyasn1-modules>=0.2.1->google-auth<3,>=1.6.3->tensorboard>=2.4.1->-r requirements.txt (line 16)) (0.4.8)
    Requirement already satisfied: oauthlib>=3.0.0 in /usr/local/lib/python3.7/dist-packages (from requests-oauthlib>=0.7.0->google-auth-oauthlib<0.5,>=0.4.1->tensorboard>=2.4.1->-r requirements.txt (line 16)) (3.1.1)
    Installing collected packages: thop, PyYAML
      Attempting uninstall: PyYAML
        Found existing installation: PyYAML 3.13
        Uninstalling PyYAML-3.13:
          Successfully uninstalled PyYAML-3.13
    Successfully installed PyYAML-6.0 thop-0.0.31.post2005241907
    

- yolov5 git을 클론하고 필요한 패키지 등을 설치합니다.


```python
# create folder 
%cd /content/yolov5/
folder_lst = ["lol/images", "lol/labels", "lol/origin/crop"]

for folder in folder_lst:
    if not os.path.isdir(folder):
        os.makedirs(folder)

del folder_lst
```

    /content/yolov5
    

- 저는 yolov5 폴더 밑에 하위 폴더를 생성해두었습니다.


- 추후 학습에 사용할 이미지 파일을 넣을 것입니다.


```python
# image, labels 압축 해제
!unzip -uq '/content/drive/MyDrive/yolo_lol/images.zip' -d "/content/yolov5/lol/images"
!unzip -uq '/content/drive/MyDrive/yolo_lol/labels.zip' -d "/content/yolov5/lol/labels"
```

- 저는 미리 이미지, 라벨링 파일을 드라이브에 업로드 해두었습니다.


- 제 드라이브에 있는 zip파일을 앞서 만들 폴더에 압축해제 하였습니다.


- 이미지, 라벨링 파일 만드는 방법은 아래를 참고해주세요.

### 2.동영상 로드 및 편집

![](https://github.com/Romg2/Romg2.github.io/blob/master/_posts/OPGG_Part_time/2021-11-30-01/replay.PNG?raw=true)

- 저는 저의 게임 리플레이 영상(약5초)를 my_replay로 저장해두었습니다.


- 오른쪽 아래에 미니맵에 챔피언 초상화들이 확인됩니다.


- 이 챔피언들을 인식해보겠습니다.


```python
# 동영상에서 이미지 추출
vidcap = cv2.VideoCapture('/content/drive/MyDrive/yolo_lol/my_replay.mp4')

count = 0

while vidcap.isOpened():
    ret, image = vidcap.read()
    
    # 캡쳐된 이미지를 저장하는 함수
    try:
        cv2.imwrite(f"./lol/origin/frame_{str(count).zfill(3)}.jpg", image)
        print(f'Saved frame_{str(count).zfill(3)}.jpg')
    except:
        print("End Save image")
        break
    if count == 59:
        break
    count += 1
    
# 메모리 해제
vidcap.release()
```

    Saved frame_000.jpg
    Saved frame_001.jpg
    Saved frame_002.jpg
    Saved frame_003.jpg
    Saved frame_004.jpg
    Saved frame_005.jpg
    Saved frame_006.jpg
    Saved frame_007.jpg
    Saved frame_008.jpg
    Saved frame_009.jpg
    Saved frame_010.jpg
    Saved frame_011.jpg
    Saved frame_012.jpg
    Saved frame_013.jpg
    Saved frame_014.jpg
    Saved frame_015.jpg
    Saved frame_016.jpg
    Saved frame_017.jpg
    Saved frame_018.jpg
    Saved frame_019.jpg
    Saved frame_020.jpg
    Saved frame_021.jpg
    Saved frame_022.jpg
    Saved frame_023.jpg
    Saved frame_024.jpg
    Saved frame_025.jpg
    Saved frame_026.jpg
    Saved frame_027.jpg
    Saved frame_028.jpg
    Saved frame_029.jpg
    Saved frame_030.jpg
    Saved frame_031.jpg
    Saved frame_032.jpg
    Saved frame_033.jpg
    Saved frame_034.jpg
    Saved frame_035.jpg
    Saved frame_036.jpg
    Saved frame_037.jpg
    Saved frame_038.jpg
    Saved frame_039.jpg
    Saved frame_040.jpg
    Saved frame_041.jpg
    Saved frame_042.jpg
    Saved frame_043.jpg
    Saved frame_044.jpg
    Saved frame_045.jpg
    Saved frame_046.jpg
    Saved frame_047.jpg
    Saved frame_048.jpg
    Saved frame_049.jpg
    Saved frame_050.jpg
    Saved frame_051.jpg
    Saved frame_052.jpg
    Saved frame_053.jpg
    Saved frame_054.jpg
    Saved frame_055.jpg
    Saved frame_056.jpg
    Saved frame_057.jpg
    Saved frame_058.jpg
    Saved frame_059.jpg
    

- 우선 영상을 프레임 단위의 이미지로 저장하여야 합니다.


- 60fps로 5 x 60 = 300 이미지 중 60장만 저장해두겠습니다.


- 저는 이미지 파일들을 따로 저장하고 추후 이 코드는 실행하지 않았습니다.


```python
# 샘플로 1개 이미지 가져오기
img_basic = cv2.imread('./lol/origin/frame_000.jpg', cv2.IMREAD_COLOR)

plt.imshow(cv2.cvtColor(img_basic, cv2.COLOR_BGR2RGB))
plt.show()
```


    
![png](https://github.com/Romg2/Romg2.github.io/blob/master/_posts/OPGG_Part_time/2021-11-30-01/output_20_0.png?raw=true)
    


- 프레임 단위로 저장된 이미지 중 1개를 불러왔습니다.


- 잘 불러와지는 것이 확인 되네요.


```python
# 미니맵 자르기
img = img_basic[1010:,1874:,:]
plt.imshow(cv2.cvtColor(img, cv2.COLOR_BGR2RGB))
plt.show()
```


    
![png](https://github.com/Romg2/Romg2.github.io/blob/master/_posts/OPGG_Part_time/2021-11-30-01/output_22_0.png?raw=true)
    


- 여기선 수기로 잘라보면서 미니맵을 추출하였습니다.


- 실제로는 사용자별로 미니맵 크기가 다르므로 설정이 필요합니다.


- `cv2.matchTemplate()` 등을 사용해서 자동화 해야합니다.


- 저는 체험의 목적이 강하기 때문에 그냥 진행하겠습니다.


```python
# 원본 사진을 미니맵만 잘라 저장하기
origin_jpg_len = len(glob.glob('./lol/origin/*jpg'))

for i in range(origin_jpg_len-1):
    # 원본 사진 불러오기
    img_origin = cv2.imread(f'./lol/origin/frame_{str(i).zfill(3)}.jpg', cv2.IMREAD_COLOR)
    # 미니맵 자르기
    img = img_origin[1010:,1874:,:]
    # 이미지 크기 변형
    expand = cv2.resize(img, None, fx=512/430, fy=512/430, interpolation=cv2.INTER_CUBIC)
    
    cv2.imwrite(f"./lol/origin/crop/crop_frame_{str(i).zfill(3)}.jpg", expand)
```

- 앞서 미니맵을 자른 과정을 모든 사진에 대해 반복합니다.


- 자른 미니맵은 crop_frame이란 이름으로 저장하였습니다.


- 이 이미지들은 라벨링에 사용할 것이며 실제 학습에 사용될 이미지입니다.


- 저는 미리 저장을 해두어 앞서 1.환경 설정에서 처럼 다른 경로에 풀어두었습니다.


```python
# 이미지로 영상 만들기
paths = sorted(glob.glob('./lol/origin/crop/*.jpg'))
fps = 60

frame_array = []
for idx , path in enumerate(paths) : 
    img = cv2.imread(path)
    height, width, layers = img.shape
    size = (width, height)
    frame_array.append(img)
out = cv2.VideoWriter('my_replay2.mp4', cv2.VideoWriter_fourcc(*'DIVX'), fps, size)
for i in range(len(frame_array)):
    # writing to a image array
    out.write(frame_array[i])
out.release()
```

- 이제 자른 이미지들을 다시 영상으로 만들어 줍니다.


- 만들어진 영상은 detection에 사용할 것입니다.

![](https://github.com/Romg2/Romg2.github.io/blob/master/_posts/OPGG_Part_time/2021-11-30-01/minimap_replay.PNG?raw=true)

- 위와 같이 미니맵 부분만 영상으로 잘 만들어졌습니다.


- 사실 이 방법 말고도 `ffmpeg` 등을 사용하면 더 편리할 수도 있습니다.


- 궁금하신분은 구글에 crop video를 검색하세요.

### 3.라벨링

![](https://github.com/Romg2/Romg2.github.io/blob/master/_posts/OPGG_Part_time/2021-11-30-01/labelimg.PNG?raw=true)

- 라벨링은 `labelImg`를 설치하여 진행하였습니다.


- 저는 라벨링 소개가 아니므로 자세한 방법은 구글링을 부탁드립니다.


- 모든 이미지에서 10개의 챔피언에 대해 바운딩 박스를 정하고 라벨링을 진행합니다.


- 생각보다 정말 오래 걸립니다..

### 4.Yolov5 실행


```python
# yaml 만들기
%cd /content/yolov5/data

with open("lol.yaml", "w", encoding="utf8") as yaml:
    # 현재 train, val 구분하지 않으므로 모두 같은 경로를 설정하였음
    yaml.write("train: ./lol/images/")
    yaml.write("\nval: ./lol/images/")
    
    # number of classes
    yaml.write("\n\nnc: 10")
    # class names
    yaml.write("\nnames: ['Syndra','Thresh','Jhin','Graves','Irellia','Xayah','Taliyah','Yone','Pyke','Poppy']")
```

    /content/yolov5/data
    

- 모델을 실행하기 앞서 yaml 파일을 생성하여야 합니다.


- yaml 파일은 train, val 이미지 경로, 인식할 클래스 수, 클래스 명을 입력합니다.


- 저는 직접 yaml 파일을 작성하고 수정하는 방식을 위해 코드로 작성했습니다.


- 그리고 학습시킬 이미지가 적어 train, val 모두 같은 경로를 설정했습니다.


```python
# 학습시킬 이미지 파일 경로
%cd /content/yolov5/

# train
img_lst = sorted(glob.glob('/content/yolov5/lol/images/*.jpg'))
with open("train_list.txt", "w", encoding="utf8") as txt:
    for i, title in enumerate(img_lst):
        txt.write(title+"\n")

# val (현재 train과 동일)
img_lst = sorted(glob.glob('/content/yolov5/lol/images/*.jpg'))
with open("val_list.txt", "w", encoding="utf8") as txt:
    for i, title in enumerate(img_lst):
        txt.write(title+"\n")
```

    /content/yolov5
    

- yolov5폴더 내에 학습시킬 이미지들의 경로를 모두 적은 txt 파일을 생성합니다.


```python
# 학습 시작
!python train.py --img 512 --batch 10 --epochs 10 --data ./data/lol.yaml --cfg ./models/yolov5s.yaml --weights yolov5s.pt --name lol_yolov5s_results
```

    Downloading https://ultralytics.com/assets/Arial.ttf to /root/.config/Ultralytics/Arial.ttf...
    [34m[1mtrain: [0mweights=yolov5s.pt, cfg=./models/yolov5s.yaml, data=./data/lol.yaml, hyp=data/hyps/hyp.scratch.yaml, epochs=10, batch_size=10, imgsz=512, rect=False, resume=False, nosave=False, noval=False, noautoanchor=False, evolve=None, bucket=, cache=None, image_weights=False, device=, multi_scale=False, single_cls=False, adam=False, sync_bn=False, workers=8, project=runs/train, name=lol_yolov5s_results, exist_ok=False, quad=False, linear_lr=False, label_smoothing=0.0, patience=100, freeze=0, save_period=-1, local_rank=-1, entity=None, upload_dataset=False, bbox_interval=-1, artifact_alias=latest
    [34m[1mgithub: [0mup to date with https://github.com/ultralytics/yolov5 ✅
    YOLOv5 🚀 v6.0-113-g5ca5dd4 torch 1.10.0+cu111 CUDA:0 (Tesla K80, 11441MiB)
    
    [34m[1mhyperparameters: [0mlr0=0.01, lrf=0.1, momentum=0.937, weight_decay=0.0005, warmup_epochs=3.0, warmup_momentum=0.8, warmup_bias_lr=0.1, box=0.05, cls=0.5, cls_pw=1.0, obj=1.0, obj_pw=1.0, iou_t=0.2, anchor_t=4.0, fl_gamma=0.0, hsv_h=0.015, hsv_s=0.7, hsv_v=0.4, degrees=0.0, translate=0.1, scale=0.5, shear=0.0, perspective=0.0, flipud=0.0, fliplr=0.5, mosaic=1.0, mixup=0.0, copy_paste=0.0
    [34m[1mWeights & Biases: [0mrun 'pip install wandb' to automatically track and visualize YOLOv5 🚀 runs (RECOMMENDED)
    [34m[1mTensorBoard: [0mStart with 'tensorboard --logdir runs/train', view at http://localhost:6006/
    Downloading https://github.com/ultralytics/yolov5/releases/download/v6.0/yolov5s.pt to yolov5s.pt...
    100% 14.0M/14.0M [00:00<00:00, 98.6MB/s]
    
    Overriding model.yaml nc=80 with nc=10
    
                     from  n    params  module                                  arguments                     
      0                -1  1      3520  models.common.Conv                      [3, 32, 6, 2, 2]              
      1                -1  1     18560  models.common.Conv                      [32, 64, 3, 2]                
      2                -1  1     18816  models.common.C3                        [64, 64, 1]                   
      3                -1  1     73984  models.common.Conv                      [64, 128, 3, 2]               
      4                -1  2    115712  models.common.C3                        [128, 128, 2]                 
      5                -1  1    295424  models.common.Conv                      [128, 256, 3, 2]              
      6                -1  3    625152  models.common.C3                        [256, 256, 3]                 
      7                -1  1   1180672  models.common.Conv                      [256, 512, 3, 2]              
      8                -1  1   1182720  models.common.C3                        [512, 512, 1]                 
      9                -1  1    656896  models.common.SPPF                      [512, 512, 5]                 
     10                -1  1    131584  models.common.Conv                      [512, 256, 1, 1]              
     11                -1  1         0  torch.nn.modules.upsampling.Upsample    [None, 2, 'nearest']          
     12           [-1, 6]  1         0  models.common.Concat                    [1]                           
     13                -1  1    361984  models.common.C3                        [512, 256, 1, False]          
     14                -1  1     33024  models.common.Conv                      [256, 128, 1, 1]              
     15                -1  1         0  torch.nn.modules.upsampling.Upsample    [None, 2, 'nearest']          
     16           [-1, 4]  1         0  models.common.Concat                    [1]                           
     17                -1  1     90880  models.common.C3                        [256, 128, 1, False]          
     18                -1  1    147712  models.common.Conv                      [128, 128, 3, 2]              
     19          [-1, 14]  1         0  models.common.Concat                    [1]                           
     20                -1  1    296448  models.common.C3                        [256, 256, 1, False]          
     21                -1  1    590336  models.common.Conv                      [256, 256, 3, 2]              
     22          [-1, 10]  1         0  models.common.Concat                    [1]                           
     23                -1  1   1182720  models.common.C3                        [512, 512, 1, False]          
     24      [17, 20, 23]  1     40455  models.yolo.Detect                      [10, [[10, 13, 16, 30, 33, 23], [30, 61, 62, 45, 59, 119], [116, 90, 156, 198, 373, 326]], [128, 256, 512]]
    Model Summary: 270 layers, 7046599 parameters, 7046599 gradients, 15.9 GFLOPs
    
    Transferred 342/349 items from yolov5s.pt
    Scaled weight_decay = 0.00046875
    [34m[1moptimizer:[0m SGD with parameter groups 57 weight, 60 weight (no decay), 60 bias
    [34m[1malbumentations: [0mversion 1.0.3 required by YOLOv5, but version 0.1.12 is currently installed
    [34m[1mtrain: [0mScanning 'lol/labels' images and labels...60 found, 0 missing, 0 empty, 0 corrupted: 100% 60/60 [00:00<00:00, 1026.48it/s]
    [34m[1mtrain: [0mNew cache created: lol/labels.cache
    [34m[1mval: [0mScanning 'lol/labels.cache' images and labels... 60 found, 0 missing, 0 empty, 0 corrupted: 100% 60/60 [00:00<?, ?it/s]
    Plotting labels to runs/train/lol_yolov5s_results/labels.jpg... 
    
    [34m[1mAutoAnchor: [0m6.67 anchors/target, 1.000 Best Possible Recall (BPR). Current anchors are a good fit to dataset ✅
    Image sizes 512 train, 512 val
    Using 2 dataloader workers
    Logging results to [1mruns/train/lol_yolov5s_results[0m
    Starting training for 10 epochs...
    
         Epoch   gpu_mem       box       obj       cls    labels  img_size
           0/9     1.38G    0.1274   0.06466   0.06789       140       512: 100% 6/6 [00:06<00:00,  1.02s/it]
                   Class     Images     Labels          P          R     mAP@.5 mAP@.5:.95: 100% 3/3 [00:01<00:00,  1.99it/s]
                     all         60        600     0.0087       0.06    0.00505    0.00135
    
         Epoch   gpu_mem       box       obj       cls    labels  img_size
           1/9     1.57G    0.1265    0.0677   0.06738       113       512: 100% 6/6 [00:03<00:00,  1.73it/s]
                   Class     Images     Labels          P          R     mAP@.5 mAP@.5:.95: 100% 3/3 [00:01<00:00,  2.07it/s]
                     all         60        600     0.0209     0.0667     0.0129    0.00338
    
         Epoch   gpu_mem       box       obj       cls    labels  img_size
           2/9     1.57G    0.1247   0.07307   0.06695       151       512: 100% 6/6 [00:03<00:00,  1.73it/s]
                   Class     Images     Labels          P          R     mAP@.5 mAP@.5:.95: 100% 3/3 [00:01<00:00,  2.08it/s]
                     all         60        600     0.0235       0.09     0.0173    0.00492
    
         Epoch   gpu_mem       box       obj       cls    labels  img_size
           3/9     1.57G    0.1218   0.07233   0.06695       100       512: 100% 6/6 [00:03<00:00,  1.72it/s]
                   Class     Images     Labels          P          R     mAP@.5 mAP@.5:.95: 100% 3/3 [00:01<00:00,  2.12it/s]
                     all         60        600     0.0266     0.0932     0.0202    0.00721
    
         Epoch   gpu_mem       box       obj       cls    labels  img_size
           4/9     1.57G    0.1198   0.07748   0.06692       184       512: 100% 6/6 [00:03<00:00,  1.73it/s]
                   Class     Images     Labels          P          R     mAP@.5 mAP@.5:.95: 100% 3/3 [00:01<00:00,  2.22it/s]
                     all         60        600     0.0189     0.0983     0.0201    0.00775
    
         Epoch   gpu_mem       box       obj       cls    labels  img_size
           5/9     1.57G    0.1197   0.08971   0.06654       199       512: 100% 6/6 [00:03<00:00,  1.73it/s]
                   Class     Images     Labels          P          R     mAP@.5 mAP@.5:.95: 100% 3/3 [00:01<00:00,  2.22it/s]
                     all         60        600     0.0237      0.103     0.0212    0.00807
    
         Epoch   gpu_mem       box       obj       cls    labels  img_size
           6/9     1.57G    0.1167   0.08073   0.06652       156       512: 100% 6/6 [00:03<00:00,  1.74it/s]
                   Class     Images     Labels          P          R     mAP@.5 mAP@.5:.95: 100% 3/3 [00:01<00:00,  2.17it/s]
                     all         60        600     0.0209      0.112     0.0193    0.00695
    
         Epoch   gpu_mem       box       obj       cls    labels  img_size
           7/9     1.57G    0.1151   0.08292   0.06587       176       512: 100% 6/6 [00:03<00:00,  1.76it/s]
                   Class     Images     Labels          P          R     mAP@.5 mAP@.5:.95: 100% 3/3 [00:01<00:00,  2.11it/s]
                     all         60        600     0.0161      0.203     0.0201    0.00684
    
         Epoch   gpu_mem       box       obj       cls    labels  img_size
           8/9     1.57G    0.1135   0.08108   0.06632       159       512: 100% 6/6 [00:03<00:00,  1.75it/s]
                   Class     Images     Labels          P          R     mAP@.5 mAP@.5:.95: 100% 3/3 [00:01<00:00,  2.13it/s]
                     all         60        600     0.0174      0.158     0.0191    0.00634
    
         Epoch   gpu_mem       box       obj       cls    labels  img_size
           9/9     1.57G    0.1122   0.08377   0.06593       128       512: 100% 6/6 [00:03<00:00,  1.75it/s]
                   Class     Images     Labels          P          R     mAP@.5 mAP@.5:.95: 100% 3/3 [00:01<00:00,  2.12it/s]
                     all         60        600     0.0173      0.163     0.0184    0.00598
    
    10 epochs completed in 0.016 hours.
    Optimizer stripped from runs/train/lol_yolov5s_results/weights/last.pt, 14.4MB
    Optimizer stripped from runs/train/lol_yolov5s_results/weights/best.pt, 14.4MB
    
    Validating runs/train/lol_yolov5s_results/weights/best.pt...
    Fusing layers... 
    Model Summary: 213 layers, 7037095 parameters, 0 gradients, 15.9 GFLOPs
                   Class     Images     Labels          P          R     mAP@.5 mAP@.5:.95: 100% 3/3 [00:03<00:00,  1.20s/it]
                     all         60        600     0.0237      0.103     0.0213    0.00813
                  Syndra         60         60          0          0          0          0
                  Thresh         60         60          0          0          0          0
                    Jhin         60         60          0          0          0          0
                  Graves         60         60    0.00894       0.05    0.00964    0.00502
                 Irellia         60         60      0.141      0.117     0.0597     0.0162
                   Xayah         60         60          0          0          0          0
                 Taliyah         60         60     0.0867      0.867      0.143     0.0601
                    Yone         60         60          0          0          0          0
                    Pyke         60         60          0          0          0          0
                   Poppy         60         60          0          0          0          0
    Results saved to [1mruns/train/lol_yolov5s_results[0m
    

- 저는 빠른 실행을 위해 epoch도 적게 설정하고 모델도 s를 사용했습니다.


```python
# detection
!python detect.py --source /content/drive/MyDrive/yolo_lol/my_replay2.mp4 --weights ./runs/train/lol_yolov5s_results/weights/best.pt
```

    [34m[1mdetect: [0mweights=['./runs/train/lol_yolov5s_results/weights/best.pt'], source=/content/drive/MyDrive/yolo_lol/my_replay2.mp4, imgsz=[640, 640], conf_thres=0.25, iou_thres=0.45, max_det=1000, device=, view_img=False, save_txt=False, save_conf=False, save_crop=False, nosave=False, classes=None, agnostic_nms=False, augment=False, visualize=False, update=False, project=runs/detect, name=exp, exist_ok=False, line_thickness=3, hide_labels=False, hide_conf=False, half=False, dnn=False
    YOLOv5 🚀 v6.0-113-g5ca5dd4 torch 1.10.0+cu111 CUDA:0 (Tesla K80, 11441MiB)
    
    Fusing layers... 
    Model Summary: 213 layers, 7037095 parameters, 0 gradients, 15.9 GFLOPs
    video 1/1 (1/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.031s)
    video 1/1 (2/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.031s)
    video 1/1 (3/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.030s)
    video 1/1 (4/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.031s)
    video 1/1 (5/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.030s)
    video 1/1 (6/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.030s)
    video 1/1 (7/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.030s)
    video 1/1 (8/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.028s)
    video 1/1 (9/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.028s)
    video 1/1 (10/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.028s)
    video 1/1 (11/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.028s)
    video 1/1 (12/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.027s)
    video 1/1 (13/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.026s)
    video 1/1 (14/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.025s)
    video 1/1 (15/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.025s)
    video 1/1 (16/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.025s)
    video 1/1 (17/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.025s)
    video 1/1 (18/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (19/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (20/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (21/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (22/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (23/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (24/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (25/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (26/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (27/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (28/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (29/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (30/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (31/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (32/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (33/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (34/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (35/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (36/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (37/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (38/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (39/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (40/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (41/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (42/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (43/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (44/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (45/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (46/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (47/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (48/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (49/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (50/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (51/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (52/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (53/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (54/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (55/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (56/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (57/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (58/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (59/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (60/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (61/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (62/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (63/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (64/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (65/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (66/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (67/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (68/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (69/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (70/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (71/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (72/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (73/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (74/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (75/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (76/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (77/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (78/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (79/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (80/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (81/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (82/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (83/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (84/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (85/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (86/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (87/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (88/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (89/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (90/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (91/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (92/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (93/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (94/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (95/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (96/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (97/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (98/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (99/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (100/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (101/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (102/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (103/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (104/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (105/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (106/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (107/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (108/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (109/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (110/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (111/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (112/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (113/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (114/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (115/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (116/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (117/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (118/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (119/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (120/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (121/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (122/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (123/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (124/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (125/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (126/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (127/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (128/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (129/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (130/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (131/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (132/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (133/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (134/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (135/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (136/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (137/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (138/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (139/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (140/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (141/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (142/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (143/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (144/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (145/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (146/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (147/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (148/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (149/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (150/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (151/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (152/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (153/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (154/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (155/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (156/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (157/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (158/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (159/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (160/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (161/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (162/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (163/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (164/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (165/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (166/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (167/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (168/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (169/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (170/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (171/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (172/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (173/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (174/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (175/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (176/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (177/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (178/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (179/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (180/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (181/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (182/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (183/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (184/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (185/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (186/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (187/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (188/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (189/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (190/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (191/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (192/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (193/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (194/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (195/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (196/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (197/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (198/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (199/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (200/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (201/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (202/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (203/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (204/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (205/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (206/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (207/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (208/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (209/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (210/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (211/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (212/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (213/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (214/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (215/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (216/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (217/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (218/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (219/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (220/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (221/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (222/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (223/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (224/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (225/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (226/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (227/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (228/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (229/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (230/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (231/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (232/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (233/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (234/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (235/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (236/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (237/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (238/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (239/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (240/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (241/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (242/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (243/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (244/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (245/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (246/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (247/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (248/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (249/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (250/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (251/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (252/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (253/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (254/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (255/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (256/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (257/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (258/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (259/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (260/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (261/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (262/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (263/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (264/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (265/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (266/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (267/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (268/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (269/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (270/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (271/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (272/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (273/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (274/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (275/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (276/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (277/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (278/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (279/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (280/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (281/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (282/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (283/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (284/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (285/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (286/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (287/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (288/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (289/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (290/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (291/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (292/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (293/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (294/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (295/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.024s)
    video 1/1 (296/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (297/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (298/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (299/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    video 1/1 (300/300) /content/drive/MyDrive/yolo_lol/my_replay2.mp4: 640x640 Done. (0.023s)
    Speed: 0.5ms pre-process, 23.7ms inference, 0.3ms NMS per image at shape (1, 3, 640, 640)
    Results saved to [1mruns/detect/exp[0m
    

- 마지막으로 detction 과정입니다.


- 앞서 저장한 my_replay2를 사용하였습니다.


- runs/detect/exp 폴더에 detection이 된 영상이 저장됩니다.


- 학습 결과 등은 runs/train 폴더에 저장됩니다.

![](https://github.com/Romg2/Romg2.github.io/blob/master/_posts/OPGG_Part_time/2021-11-30-01/yolo_replay.PNG?raw=true)

- detection을 진행한 영상 캡처본입니다.


- 사실 이 영상은 위 학습에서 epoch를 200으로 설정했을 때 입니다.


- 정리하면서 다시 실행한다고 시간을 줄이려고 epoch을 10으로 해두었습니다.


- 챔피언 초상화를 어느정도 인식한 것으로 보이네요.

### 5.EOD

yolov5 모델을 이용해서 롤 게임영상에서 미니맵 초상화를 인식해보았습니다.

라벨링이나 yolov5 사용법에 대한 자세한 설명이 없기에 꼭 직접 찾아보시길 바랍니다.

여기선 모델 성능을 아예 고려하지 않았기 때문에 이런 식으로 작업이 가능하구나라고 봐주시면 감사하겠습니다.

그리고 여기선 detection으로 끝을 냈지만 옵션 설정에 따라 좌표도 얻을 수 있습니다.

이를 활용해서 다양한 분석이 가능할 것 같네요.
