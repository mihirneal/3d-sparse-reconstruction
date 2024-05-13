# 3D Reconstruction Via Sparse depthless images

[![Video](https://img.shields.io/badge/YouTube-Video-c4302b?logo=youtube&logoColor=red)](https://youtu.be/CarJgsx3DQY)



---------------------------------------------------
<p align="center" >
  <a href="">
    <img src="https://github.com/zhiwenfan/zhiwenfan.github.io/blob/master/Homepage_files/videos/FSGS_gif.gif?raw=true" alt="demo" width="85%">
  </a>
</p>


## Environmental Setups
We provide install method based on Conda package and environment management:
```bash
conda env create --file environment.yml
conda activate FSGS
```
**CUDA 11.7** is strongly recommended.

## Data Preparation
In data preparation step, we reconstruct the sparse view inputs using SfM using the camera poses provided by datasets. Next, we continue the dense stereo matching under COLMAP with the function `patch_match_stereo` and obtain the fused stereo point cloud from `stereo_fusion`. 

``` 
cd FSGS
mkdir dataset 
cd dataset

# download LLFF dataset
gdown 16VnMcF1KJYxN9QId6TClMsZRahHNMW5g

# run colmap to obtain initial point clouds with limited viewpoints
python tools/colmap_llff.py

# download MipNeRF-360 dataset
wget http://storage.googleapis.com/gresearch/refraw360/360_v2.zip
unzip -d mipnerf360 360_v2.zip

# run colmap on MipNeRF-360 dataset
python tools/colmap_360.py
``` 


We use the latest version of colmap to preprocess the datasets. If you meet issues on installing colmap, we provide a docker option. 
``` 
# if you can not install colmap, follow this to build a docker environment
docker run --gpus all -it --name fsgs_colmap --shm-size=32g  -v /home:/home colmap/colmap:latest /bin/bash
apt-get install pip
pip install numpy
python3 tools/colmap_llff.py
``` 


We provide both the sparse and dense point cloud after we proprecess them. You may download them [through this link](https://drive.google.com/drive/folders/1lYqZLuowc84Dg1cyb8ey3_Kb-wvPjDHA?usp=sharing). We use dense point cloud during training but you can still try sparse point cloud on your own.

## Training
Train FSGS on LLFF dataset with 3 views
``` 
python train.py  --source_path dataset/nerf_llff_data/horns --model_path output/horns --eval  --n_views 3 --sample_pseudo_interval 1
``` 


Train FSGS on MipNeRF-360 dataset with 24 views
``` 
python train.py  --source_path dataset/mipnerf360/garden --model_path output/garden  --eval  --n_views 24 --depth_pseudo_weight 0.03  
``` 


## Rendering
Run the following script to render the images.  

```
python render.py --source_path dataset/nerf_llff_data/horns/  --model_path  output/horns --iteration 10000
```

You can customize the rendering path as same as NeRF by adding `video` argument

```
python render.py --source_path dataset/nerf_llff_data/horns/  --model_path  output/horns --iteration 10000  --video  --fps 30
```

## Evaluation
You can just run the following script to evaluate the model.  

```
python metrics.py --source_path dataset/nerf_llff_data/horns/  --model_path  output/horns --iteration 10000
```

## Acknowledgement

Special thanks to the following awesome projects!

- [Gaussian-Splatting](https://github.com/graphdeco-inria/gaussian-splatting)
- [DreamGaussian](https://github.com/ashawkey/diff-gaussian-rasterization)
- [SparseNeRF](https://github.com/Wanggcong/SparseNeRF)
- [MipNeRF-360](https://github.com/google-research/multinerf)
