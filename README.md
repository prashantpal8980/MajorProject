<div align="center">

# <b>GaussianAvatar</b>: Towards Realistic Human Avatar Modeling from a Single Video via Animatable 3D Gaussians






</div>



## Introduction



## Installation

To deploy and run GaussianAvatar, run the following scripts:
```
conda env create --file environment.yml
conda activate gs-avatar
```

Then, compile ```diff-gaussian-rasterization``` and ```simple-knn``` as in [3DGS](https://github.com/graphdeco-inria/gaussian-splatting) repository.

## Download models and data 

- SMPL/SMPL-X model: register and download [SMPL](https://smpl.is.tue.mpg.de/) and [SMPL-X](https://smpl-x.is.tue.mpg.de/), and put these files in ```assets/smpl_files```. The folder should have the following structure:
```
smpl_files
 └── smpl
   ├── SMPL_FEMALE.pkl
   ├── SMPL_MALE.pkl
   └── SMPL_NEUTRAL.pkl
 └── smplx
   ├── SMPLX_FEMALE.npz
   ├── SMPLX_MALE.npz
   └── SMPLX_NEUTRAL.npz
```

- Data: download the provided data from [OneDrive](https://hiteducn0-my.sharepoint.com/:f:/g/personal/lx_hu_hit_edu_cn/EsGcL5JGKhVGnaAtJ-rb1sQBR4MwkdJ9EWqJBIdd2mpi2w?e=KnloBM). These data include ```assets.zip```, ```gs_data.zip``` and ```pretrained_models.zip```. Please unzip ```assets.zip``` to the corresponding folder in the repository and unzip others to `gs_data_path` and `pretrained_models_path`.


## Run on People Snapshot dataset

We take the subject `m4c_processed` for example. 

### Training

```
python train.py -s $gs_data_path/m4c_processed -m output/m4c_processed --train_stage 1
```

### Evaluation

```
python eval.py -s $gs_data_path/m4c_processed -m output/m4c_processed --epoch 200
```

### Rendering novel pose

```
python render_novel_pose.py -s $gs_data_path/m4c_processed -m output/m4c_processed --epoch 200
```

## Run on Your Own Video

### Preprocessing

- masks and poses: use the bash script `scripts/custom/process-sequence.sh` in [InstantAvatar](https://github.com/tijiang13/InstantAvatar). The data folder should have the followings:
```
smpl_files
 ├── images
 ├── masks
 ├── cameras.npz
 └── poses_optimized.npz
```
- data format: we provide a script to convert the pose format of romp to ours (remember to change the `path` in L50 and L51):
```
cd scripts & python sample_romp2gsavatar.py
```
- position map of the canonical pose: (remember to change the corresponding `path`)
```
python gen_pose_map_cano_smpl.py
```

### Training for Stage 1

```
cd .. &  python train.py -s $path_to_data/$subject -m output/{$subject}_stage1 --train_stage 1 --pose_op_start_iter 10
```

### Training for Stage 2

- export predicted smpl:
```
cd scripts & python export_stage_1_smpl.py
```
- visualize the optimized smpl (optional):
```
python render_pred_smpl.py
```
- generate the predicted position map:
```
python gen_pose_map_our_smpl.py
```
- start to train
```
cd .. &  python train.py -s $path_to_data/$subject -m output/{$subject}_stage2 --train_stage 2 --stage1_out_path $path_to_stage1_net_save_path
```

