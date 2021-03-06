# TransNet: Benchmark for Pedestrian Stop & Go Forecasting
[![Python 3.8](https://img.shields.io/badge/python-3.8-blue.svg)](https://www.python.org/downloads/release/python-380//)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) <br>
This repository contains the code of *TransNet*, our benchmark for studying the pedestrian stop and go behaviors in the context of self-driving. 
Walking to standing and standing to walking transitions are specific and relatively rare cases, hence samples are collected from several external datasets and 
integrated with unified interface.

![ex1](imgs/jaad_01.gif)

### Table of Contents
- [Installation](#installation)
- [Data preparation](#data-preparation)
- [Interface](#interface)
- [Statistics](#Statistics)
- [Further work](#further-work)
- [References](#references)


## Installation

```
# To clone the repository using HTTPS
git clone https://github.com/DongxuGuo1997/PSGF.git
```

The project is written and tested using python 3.8. The interface also require external libraries like PyTorch,
opencv-python, etc.  All required packages can be found in `requirements.txt` and to install dependencies run:
```
pip install -r requirements.txt
```
Please ensure all expected modules are properly installed before using the interface.


## Data preparation
#### Support datasets
At current stage we focus on the datasets related to the study of pedestrian actions/intentions from on-board vehicles.
The suitable datasets should provide RGB images captured from an egocentric view of a moving vehicle and accurately annotated.<br>

Currently TransNet support the following datasets:<br/>
* Joint Attention in Autonomous Driving([JAAD](http://data.nvision2.eecs.yorku.ca/JAAD_dataset/)) Dataset
* Pedestrian Intention Estimation([PIE](http://data.nvision2.eecs.yorku.ca/PIE_dataset/)) Dataset
* Trajectory Inference using Targeted Action priors Network([TITAN](https://usa.honda-ri.com/titan)) Dataset

#### Download
##### JAAD

- Download the videos and annotations from [official page](https://github.com/ykotseruba/JAAD). 
- Use the [scripts](https://github.com/ykotseruba/JAAD/blob/JAAD_2.0/split_clips_to_frames.sh) provided to extract images from videos.
- Use JAAD's [interface](https://github.com/ykotseruba/JAAD/blob/JAAD_2.0/jaad_data.py) to generate complete annotations in the form of python dictionary.
  More precisely use [`generate_database()`](https://github.com/ykotseruba/JAAD/blob/JAAD_2.0/jaad_data.py#L421) to obtain the all JAAD annotations in dictionary form as a pkl file.
  Please rename the .pkl file as `JAAD_DATA.pkl` and place it in `DATA/annotations/JAAD/anns`
- Train / val / test split: [link](https://github.com/ykotseruba/JAAD/tree/JAAD_2.0/split_ids).

##### PIE

- Download the videos and annotations from [offical page](https://github.com/aras62/PIE#interface). 
- Use the [scripts](https://github.com/aras62/PIE/blob/master/split_clips_to_frames.sh) provided to extract images from videos.
- Use PIE's [interface](https://github.com/aras62/PIE/blob/master/pie_data.py) to generate complete annotations in the form of python dictionary.
  In detail, please use [`generate_database()`](https://github.com/aras62/PIE/blob/master/pie_data.py#L441) to obtain the all JAAD annotations in dictionary form as a pkl file.
  Rename the .pkl file as `PIE_DATA.pkl` and place it in `DATA/annotations/PIE/anns`
- Train / val / test split: [link](https://github.com/aras62/PIE/blob/2256f96b8ab24d8407af34fb1f0b9a4714cd532e/pie_data.py#L84)

##### TITAN
To obtain the dataset, please refer to this [page]( https://usa.honda-ri.com/titan) and contact them directly.

After download the data, please place the images, annotations and video split ids in [DATA](https://github.com/DongxuGuo1997/TransNet/tree/main/DATA).
The expected result:
```
├── DATA/
│   ├── annotations/ 
│   │   ├── JAAD/ 
│   │   │   ├── anns/  
│   │   │   │   ├── JAAD_DATA.pkl # see Data-preparation-download-JAAD
│   │   │   └── splits/
│   │   │   │   ├── all_videos/
│   │   │   │   │   ├── train.txt
│   │   │   │   │   └── val.txt
│   │   │   │   │   └── test.txt
│   │   │   │   └── default/
│   │   │   │   │   ├── train.txt
│   │   │   │   │   └── val.txt
│   │   │   │   │   └── test.txt
│   │   │   │   └── high_visibility/
│   │   │   │   │   ├── train.txt
│   │   │   │   │   └── val.txt
│   │   │   │   │   └── test.txt
│   │   └── PIE/
│   │   │   ├── anns/ 
│   │   │   │   ├── PIE_DATA.pkl # see Data-preparation-download-PIE
│   │   └── TITAN/
│   │   │   ├── anns/
│   │   │   │   ├── clip_x.csv
│   │   │   └── splits/
│   │   │   │   │   ├── train_set.txt
│   │   │   │   │   └── val_set.txt
│   │   │   │   │   └── test_set.txt
│   └── images/
│   │   ├── JAAD/
│   │   │   ├── video_xxxx/ # 346 videos
│   │   └── PIE/
│   │   │   ├── set_0x/ # 6 sets
│   │   │   │   ├── video_xxxx/ 
│   │   └── TITAN/
│   │   │   ├── clip_xxx/  # 786 clips
└── (+ files and folders containing the raw data)
```
<b> Note </b>: No need to gather all datasets. The benchmark works normally with arbitrary subset of supported datasets when only the paths to desired datasets are specified.

## Interface  
At the heart of TransNet data blocks is the [`TransDataset`](https://github.com/DongxuGuo1997/TransNet/blob/main/src/dataset/trans/data.py) class.
`TransDataset` integrates the functions to collect transition samples from original annotations of JAAD, PIE and TITAN.
Using attributes of `TransDataset`, the user can conveniently extract the frame and history instances related to the stop & go of pedestrians:<br>
* `extract_trans_frame()`: extract the frame where stop or go transitions occur and the annotations of involved pedestrian
* `extract_trans_history()`: extract the whole history of a pedestrian up to the frame when transition happens <br>
The extracted samples each has an unique id specifying the source dataset(`J`,`P`,`T`), transition type(`S`,`G`),data split(`trian`,`val`,`test`) 
and sample index,ie. `TG_003_train`. The data loading is done by customized PyTorch `torch.utils.data.Dataset`, namely [`SequenceDataset`](https://github.com/DongxuGuo1997/TransNet/blob/main/src/dataset/loader.py#L76) and 
[`FrameDataset`](https://github.com/DongxuGuo1997/TransNet/blob/main/src/dataset/loader.py#L29). We also privede [`BaseVisualizer`](https://github.com/DongxuGuo1997/TransNet/blob/main/src/visualizer/draw.py#L36) for demonstrate the extracted samples. 
Also notice TransNet works with arbitrary subset of supported datasets. 
<b>For detailed usage please check the example in [<b>example.ipynb </b>](https://github.com/DongxuGuo1997/TransNet/blob/main/example.ipynb). <br>
</b><br>

![ex2](imgs/TITAN.gif)

## Statistics
After integrating JAAD, PIE and TITAN, TransNet contains <b>1078</b> "GO" samples and <b>1216</b> "STOP" samples, 
involving <b>936</b> and <b>1060</b> unique pedestrians respectively. A more detailed analysis of the statistics is presented in the table below.
<br/>
<br>
![stats1](imgs/GO_stats.PNG)
<br>
<br>
![stats2](imgs/STOP_stats.PNG)
<br>
where <br>
* `#samples`: number of transition instances
* `#pedestrians`: number of unique pedestrians
* `#frames`: number of frames in all transition history(sequence) samples, with default sampling rate 10 frames per second.

## Further work
TransNet serves as the primary data source for the on-going project "Pedestrian Stop and Go Forecasting" in [VITA](https://www.epfl.ch/labs/vita/).
In future we expected more relevant datasets to be integrated in this benchmark.
Please send email to [dongxu.guo@epfl.ch]() if you encounter any problems using the interface or have suggestions for improving the usability.

## References
- Rasouli et al., ["Are They Going to Cross? A Benchmark Dataset and Baseline for Pedestrian
Crosswalk Behavior"](https://openaccess.thecvf.com/content_ICCV_2017_workshops/papers/w3/Rasouli_Are_They_Going_ICCV_2017_paper.pdf), ICCV 2017
- Rasouli et al., ["PIE: A Large-Scale Dataset and Models for Pedestrian Intention Estimation and
Trajectory Prediction"](https://openaccess.thecvf.com/content_ICCV_2019/papers/Rasouli_PIE_A_Large-Scale_Dataset_and_Models_for_Pedestrian_Intention_Estimation_ICCV_2019_paper.pdf),
 ICCV 2019
- Malla et al., ["TITAN: Future Forecast using Action Priors"](https://arxiv.org/abs/2003.13886), CVPR 2020
