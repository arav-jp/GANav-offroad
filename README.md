# GANav: Efficient Terrain Segmentation for Robot Navigation in Unstructured Outdoor Environments (Accepted by RAL/IROS 2022)

[![License](https://img.shields.io/badge/License-Apache%202.0-green.svg)](https://github.com/rayguan97/GANav-offroad/blob/master/LICENSE)

This is the code base for:

[GANav: Efficient Terrain Segmentation for Robot Navigation in Unstructured Outdoor Environments](https://gamma.umd.edu/offroad).
<br> Tianrui Guan, Divya Kothandaraman, Rohan Chandra, Adarsh Jagan Sathyamoorthy, Kasun Weerakoon, Dinesh Manocha

<img src="./resources/video.gif" width="560">

If you find this project useful in your research, please cite our work:

```latex
@misc{guan2021ganav,
      title={GANav: Efficient Terrain Segmentation for Robot Navigation in Unstructured Outdoor Environments}, 
      author={Tianrui Guan and Divya Kothandaraman and Rohan Chandra and Adarsh Jagan Sathyamoorthy and Kasun Weerakoon and Dinesh Manocha},
      year={2021},
      eprint={2103.04233},
      archivePrefix={arXiv},
      primaryClass={cs.RO}
}

```

Our video can be found [here](https://www.youtube.com/watch?v=QN5FKakQwfo).

## Updates:
06-17-2022: Accepted by RAL/IROS. Latest code still in preparation.

02-21-2022: Updated code and checkpoints in preparation.

12-09-2021: Updated ros-support for GANav [here](https://github.com/rayguan97/GANav-offroad/tree/main/ros_support).

05-10-2021: Trained models are ready. Please refer to `./trained_models` folder. Please download the trained model [here](https://drive.google.com/drive/folders/1PYn_kT0zBGOIRSaO_5Jivaq3itrShiPT?usp=sharing).

# Introduction

<img src="./resources/cover.png" width="700">


We propose GANav, a novel group-wise attention mechanism to identify safe and navigable regions in off-road terrains and unstructured environments from RGB images. Our approach classifies terrains based on their navigability levels using coarse-grained semantic segmentation. Our novel group-wise attention loss enables any backbone network to explicitly focus on the different groups' features with low spatial resolution. Our design leads to efficient inference while maintaining a high level of accuracy compared to existing SOTA methods. 

Our extensive evaluations on the RUGD and RELLIS-3D datasets shows that GANav achieves an improvement over the SOTA mIoU by 2.25-39.05% on RUGD and 5.17-19.06% on RELLIS-3D. We interface GANav with a deep reinforcement learning-based navigation algorithm and highlight its benefits in terms of navigation in real-world unstructured terrains. We integrate our GANav-based navigation algorithm with ClearPath Jackal and Husky robots, and observe an increase of 10% in terms of success rate, 2-47% in terms of selecting the surface with the best navigability and a decrease of 4.6-13.9% in trajectory roughness. Further, GANav reduces the false positive rate of forbidden regions by 37.79%. Code, videos, and a full technical report are available at https://gamma.umd.edu/offroad/.

# Environment


### Step 1: Create Conda Environment

```
conda create -n ganav python=3.7 -y
conda activate ganav
conda install pytorch=1.6.0 torchvision cudatoolkit=10.1 -c pytorch
# or use 
# conda install pytorch=1.10.0 torchvision cudatoolkit=11.3 -c pytorch
```

### Step 2: Installing MMCV

```
pip install mmcv-full -f https://download.openmmlab.com/mmcv/dist/cu101/torch1.6.0/index.html
# or use
# pip install mmcv-full -f https://download.openmmlab.com/mmcv/dist/cu113/torch1.10.0/index.html
```
Note: Make sure you mmcv version is compatible with your pytorch and cuda version.

### Step 3: Installing GANav
```
git clone https://github.com/rayguan97/GANav-offroad.git
cd GANav-offroad
pip install einops
pip install -e . 
```


# Get Started

In this section, we explain the data generation process and how to train and test our network.

## Data Processing

To be able to run our network, please follow those steps for generating processed data.

### Dataset Download: 

Please go to [RUGD](http://rugd.vision/) and [RELLIS-3D](https://github.com/unmannedlab/RELLIS-3D/blob/main/README.md#annotated-data) (we use the ID annotation instead of color annotation for RELLIS-3D) officail website to download their data. Please structure the downloaded datain as follows:

```
GANav
├── data
│   ├── rellis
│   │   │── test.txt
│   │   │── train.txt
│   │   │── val.txt
│   │   │── annotation
│   │   │   ├── 00000 & 00001 & 00002 & 00003 & 00004 
│   │   │── image
│   │   │   ├── 00000 & 00001 & 00002 & 00003 & 00004 
│   ├── rugd
│   │   │── test_ours.txt
│   │   │── test.txt
│   │   │── train_ours.txt
│   │   │── train.txt
│   │   │── val_ours.txt
│   │   │── val.txt
│   │   │── RUGD_annotations
│   │   │   ├── creek & park-1/2/8 & trail-(1 & 3-7 & 9-15) & village
│   │   │── RUGD_frames-with-annotations
│   │   │   ├── creek & park-1/2/8 & trail-(1 & 3-7 & 9-15) & village
├── configs
├── tools
...
```

### Dataset Processing: 

In this step, we need to process the groundtruth labels, as well as generating the grouped labels.

For RELLIS-3D dataset, run:

   ```
   python ./tools/convert_datasets/rellis_relabel[x].py
   ``` 

For RUGD dataset, run:

   ```
   python ./tools/convert_datasets/rugd_relabel[x].py
   ``` 

Replease [x] with 4 or 6, to generated data with 4 annotation groups or 6 annotation groups.

## Training

To train a model on RUGD datasets with our methods on 6 groups:
```
python ./tools/train.py ./configs/ours/ganav_group6_rugd.py
```

Please modify `./configs/ours/*` to play with your model and read `./tools/train.py` for more details about training options.

## Testing

In order to test the model, download the trained models from [here](https://drive.google.com/drive/folders/1PYn_kT0zBGOIRSaO_5Jivaq3itrShiPT?usp=sharing) and copy them into the
directory `./trained_models`.

An example to evaluate our method with 6 groups on RUGD datasets with mIoU metrics:

```
python ./tools/test.py ./trained_models/rugd_group6/ganav_rugd.py \
          ./trained_models/rugd_group6/ganav_rugd.pth --eval=mIoU
```
Please read `./tools/test.py` for more details.

<!-- To repreduce the papers results, please refer `./trained_models` folder. Please download the trained model [here](https://drive.google.com/drive/folders/1PYn_kT0zBGOIRSaO_5Jivaq3itrShiPT?usp=sharing). -->



# License

This project is released under the [Apache 2.0 license](LICENSE).

# Acknowledgement

The source code of GANav is heavily based on [MMSegmentation](https://github.com/open-mmlab/mmsegmentation). 

