Official implementation of [Explicit Shape Encoding for Real-time Instance Segmentation](https://arxiv.org/abs/1908.04067). 

<p align='center'>
    <img src="quality.png", width="640">
</p>


Contents

[Requirements](#requirements)

[Features](#features)

[Preparation](#preparation)

[Training](#training)

[Evaluation](#evaluation)

[Benchmarking the speed of network](#benchmarking-the-speed-of-network)

---

### Requirements
- python 3.6
- mxnet 1.5.0
- shapely 1.6.4

Other common package version is not very crucial.

### Features
- Include both ESE-Seg(Yolov3-darknet53) and ESE-Seg(Yolov3-tiny).
- Good performance

  |416x416 |VOC2012 Test(mAP)| SBD Test(mAP)| Time per forward<br/>(batch size = 1)|
  | :-: | :-:| :-:|:-:|
  | ESE-Seg(Darknet53) | 69.3% | 64.1% |26 ms|
  | ESE-Seg(Darknet-tiny)| 53.2% | 48.1% |8.0 ms|
  
  The models are trained from bbox-pretrained weights on coco and then trained in SBD.

- Some efficient backbones on hand

  Like yolov3-tiny, yolov3. 

  Check folder `/gluon-cv/gluoncv/model_zoo/yolo/darknet.py` for details.
---

### Preparation
##### 1) Code
`git clone https://github.com/WenqiangX/ese_seg.git`

`git clone https://github.com/dmlc/gluon-cv.git`

`cd gluon-cv`

`python setup.py develop --user`

`cd ..`

`cp -r ./ese_seg/gluon-cv gluon-cv`

`cd ese_seg`

##### 2) Data
- SBD (Semantic Boundaries Dataset and Benchmark)

`mkdir data`

`cd data`

Firstly, you can download our preprocessed dataset on [google drive](https://drive.google.com/file/d/1Yr2ggBOBhXR8gn5FZDlMy2CUWjf-64NU/view?usp=sharing).

`tar -zxvf VOCsbdche.tar.gz`
- Custom Dataset

Our dataset format is equal to pascal voc 2012, you can check it in our preprocessed dataset.

JPEGImages( image data )

ImageSets ( dataname list )

cheby_fit ( cheby_coefficient train label )

label_polygon_360_xml ( polygon val label )

We provide preprocessing code in ./labelutils, you can just run your.bash simply according to sbd_dataset.bash
For example, in SBD, preprocess label into (SegementationObject/   SegmentationClass/) as PascalVOC format

    ./sbd
        ./SegementationObject
        ./SegementationClass
make the ./labelutils and ./sbd in the same directory, such ./ESE-SEG/sbd

`bash sbd_dataset.bash`

Another important thing is that there will be very few invalid labels, so you can change your train/val labelname list according to ./coef_8_success.txt. We have provided the sbd vaild train and val txt.

##### 3) weights
To Do: 
We will release two pretrained weight based on darknet53 and tinydarknet in some weeks.


### Training

##### 1) Direct training

1.1) run

`cd ese_seg`

`python sbd_train_che_8.py --syncbn --network darknet53 --batch-size 20 --dataset voc --gpus 0 --warmup-epochs 10 --save-prefix ./darknet53_result`

or

`python sbd_train_che_8.py --syncbn --network tiny_darknet --batch-size 64 --dataset voc --gpus 0 --warmup-epochs 10 --save-prefix ./darknet53_result`

you can also change the val dataset to voc2012 by --val_2012.

The result are both very well by direct training. Your can pretrain only bbox in coco, make its bbox prediction more accurate.

##### 2) Pretrain bbox

1.1) Preprocess the coco dataset as custom dataset. And coco dataset is train dataset, voc/sbd is val dataset. You can check the path in ./gluon-cv/gluoncv/data/pascal_voc/detection.py 

1.2) run

`python sbd_train_che_8.py --syncbn --network darknet53 --only_bbox True --batch-size 20 --dataset cocopretrain --gpus 0 --warmup-epochs 10 --save-prefix ./darknet53_pretrain_coco_result`

or
    
`python sbd_train_che_8.py --syncbn --network tiny_darknet --only_bbox True --batch-size 20 --dataset cocopretrain --gpus 0 --warmup-epochs 10 --save-prefix ./darknet53_pretrain_coco_result`

##### 3) Results
To Do:


---

### Evaluation
`cd ese_seg`

##### 1) ESE-SEG Yolov3
1.1) Your should have a model weight \$weight_path\$

1.2) run

`python sbd_eval_che_8.py --network darknet53 --val_voc2012 True --resume weight_path --save-prefix ./eval_voc2012_tinydarknet`

or 

`python sbd_eval_che_8.py --network darknet53 --resume weight_path --save-prefix ./eval_voc2012_tinydarknet`

##### 2) ESE-SEG yolov3-tiny
2.1) Your should have a model weight $weight_path$

2.2) run

`python sbd_eval_che_8.py --network tiny_darknet --val_voc2012 True --resume weight_path --save-prefix ./eval_voc2012_tinydarknet`

or 

`python sbd_eval_che_8.py --network tiny_darknet --resume weight_path --save-prefix ./eval_voc2012_tinydarknet`


### Demo
`cd ese-seg`

##### 1) ESE-SEG Yolov3
1.1) Your should have a model weight \$weight_path\$ and \$image_dir\$

1.2) run

`python demo_yolo.py --network yolo3_darknet53_voc --images $image_dir$ --save_dir ./demo_darknet53 --pretrained $weight_path$`

##### 2) ESE-SEG Yolov3-tiny
2.1) Your should have a model weight $weight_path$ and \$image_dir\$

2.2) run

`python demo_yolo.py --network yolo3_darknet53_voc --images $image_dir$ --save_dir ./demo_darknet53 --pretrained $weight_path$`

---

### Benchmarking the speed of network
We test the speed based on forward process and yolo haed predicted.

Tiny-darknet: backbone ~2ms two-head ~3+3 ms


---

### Citation
If you find our work is useful to your research, feel free to cite
```
@inproceedings{xu2019ese,
  title={Explicit Shape Encoding for Real-Time Instance Segmentation},
  author={Xu, Wenqiang and Wang, Haiyang and Qi, Fubo and Lu, Cewu},
  booktitle={ICCV},
  year={2019}
}
```
### Credits
I got a lot of code from [gluon-cv](https://github.com/dmlc/gluon-cv.git), thanks to Gluoncv.

### Comments
YOLOv3 in GluonCV is the best reimplemented YOLO so far, hence we build this system upon it. However, GluonCV is more a production code than research code, so even all we have to do is changing **dataloader and network prediction head**, it still took a lot of modification from original GluonCV. For research use, it is encouraged to use more light-weighted or more flexible detection framework, should make the implementation of our core idea easier.

### Other Implementation
[ESE-SEG](https://github.com/Haiyang-W/ESE-SEG.git)
