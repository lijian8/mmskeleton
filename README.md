# Spatial Temporal Graph Convolutional Networks (ST-GCN)
A graph convolutional network for skeleton based action recognition.

<div align="center">
    <img src="tools/.info/pipeline.png">
</div>

## Introduction
This repository holds the codebase, dataset and models for the paper

**Spatial Temporal Graph Convolutional Networks for Skeleton-Based Action Recognition** Sijie Yan, Yuanjun Xiong and Dahua Lin, AAAI 2018.

[Arxiv Preprint]

## Prerequisites
Our codebase is based on **Python 2.7**. There are a few dependencies to run the code. The major libraries we used are
- [PyTorch](http://pytorch.org/)
- NumPy

## Data Preparation
We experimented on two skeleton-based action recognition datasts: [NTU RGB+D](http://rose1.ntu.edu.sg/datasets/actionrecognition.asp) and [Kinetics-skeleton](https://s3-us-west-1.amazonaws.com/yysijie-data/kinetics-skeleton.zip). 
### NTU RGB+D
NTU RGB+D can be downloaded from [their website](http://rose1.ntu.edu.sg/datasets/actionrecognition.asp). Only the **3D skeletons**(5.8GB) modality is required in our experiments. After that, ```tools/ntu_gendata.py``` should be used to build the database for training or evaluation:
```
tools/ntu_gendata.py --data_path <path to nturgbd>
```
where the ```<path to nturgbd>``` points to the 3D skeletons modality of NTU RGB+D dataset you download, for example ```data/NTU-RGB-D/nturgbd+d_skeletons```.
### Kinetics-skeleton
[Kinetics](https://deepmind.com/research/open-source/open-source-datasets/kinetics/) is a video-based dataset for action recognition which only provide raw video clips without skeleton data. To obatin the joint locations, we first resized all videos to the resolution of 340x256 and converted the frame rate to 30 fps.  Then, we extracted skeletons from each frame in Kinetics by [Openpose](https://github.com/CMU-Perceptual-Computing-Lab/openpose). The extracted skeleton data we called **Kinetics-skeleton**(7.5GB) can be directly downloaded from [here](https://s3-us-west-1.amazonaws.com/yysijie-data/kinetics-skeleton.zip).

It is highly recommended storing data in the **SSD** rather than HDD for efficiency.


##  Testing Pretrained Models
### Get trained models
We provided the trained model weithts of our **ST-GCN** and the baseline model Temporal-Conv[1]. The model weights can be downloaded by running the script
```
bash tools/get_pretrained_models.sh
```
The downloaded models will be stored under the ```./model```.

### Evaluation
Once datasets and models ready, we can start the evaluation.
<!-- Use the folloing command to evaluate our **ST-GCN** model:
```
python main.py --config config/st_gcn/<dataset>/test.yaml
```
where the ```<dataset>``` can be  -->

For cross-view evaluation in NTU RGB+D, run
```
python main.py --config config/st_gcn/nturgbd-cross-view/test.yaml
```
For  cross-subject evaluation in NTU RGB+D, run
```
python main.py --config config/st_gcn/nturgbd-cross-subject/test.yaml
```
To evaluate ST-GCN model pretrained on Kinetcis-skeleton, run
```
python main.py --config config/st_gcn/kinetics-skeleton/test.yaml
```
Similary, the configuration file for testing baseline models can be found under the ```./config/baseline```.

To speed up evaluation by multi-gpu inference or modify batch size for reducing memory cost, use
```
python main.py --config <config file> --test-batch-size <batch size> --device <gpu0> <gpu1> <gpu2> ...
```


The expected **Top-1** **accuracy** of provided models are shown here:

| Model| Kinetics-<br>skeleton (%)|NTU RGB+D <br> Cross View (%) |NTU RGB+D <br> Cross Subject (%) |
| :------| :------: | :------: | :------: |
|Baseline| 20.3    | 83.1     |  74.3    |
|**ST-GCN** (Ours)| **30.6**| **88.9** | **80.7** | 

[1] Kim, T. S., and Reiter, A. 2017. Interpretable 3d human action analysis with temporal convolutional networks. In BNMW CVPRW. 

## Training
To train a new model, run 
```
python main.py --config config/st_gcn/<dataset>/train.yaml
```
where the ```<dataset>``` should be one of ```nturgbd-cross-view```, ```nturgbd-cross-subject``` and ```kinetics-skeleton```. The training results will be saved under the ```./work_dir``` by default, including weights, configurations and logging files.

You can modify the training parameters such as ```batch-size```, ```step```, ```base_lr``` and ```device``` in the command line or configuration files. The order of priority is:  command line > config file > default parameter. For more information, use ```main.py -h```.

Finally, custom model evaluation can be achieved by this command:
```
python main.py --phase test --config <path to training config> --weights <path to model weights>
```


