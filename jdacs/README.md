# JDACS

## About
This is the training code of the JDACS framework proposed in our AAAI-21 paper: "*Self-supervised Multi-view Stereo via Effective Co-Segmentation and Data-Augmentation*" [[paper]](https://www.aaai.org/AAAI21Papers/AAAI-2549.XuH.pdf).

It is noted that [MVSNet](https://github.com/xy-guo/MVSNet_pytorch) is utilized as the backbone in default. You can also replace the backbone network with your own customized model.

## How to use?

### Environment
 - Python 3.6 (Anaconda) 
 - Pytorch 1.1.0

### Training
 - Download the preprocessed DTU training dataset \[ [Google Cloud](https://drive.google.com/file/d/1eDjh-_bxKKnEuz5h-HXS7EDJn59clx6V/view) or [Baidu Cloud](https://pan.baidu.com/s/1sQAC3pmceyochNvnqpE9oA#list/path=%2F) (The password is `mo8w`) \].
 - Edit the training settings in `train.sh`.
   - `MVS_TRAINING`: the path of your training dataset. 
   - `--gpu_device`: the ids of adopted GPUs. You should modify it according to the available GPUs in your server.
   - `--logdir`: the name of the checkpoint dir.
   - `--w_aug`: weight for data-augmentation consistency loss. The default value is 0.01.
   - `--w_seg`: weight for co-segmentation consistency loss. The default value is 0.01.
   - `--seg_clusters`: the cluster centroids for NMF (Co-segmentation). The default value is 4.
   - `--num_depth`: the number of hypothesized depth planes in the cost volume The default value is 192.
 - Train the model by running `./train.sh`.

### Evaluating
 - Download the preprocessed DTU testing data \[ [Google Cloud](https://drive.google.com/file/d/135oKPefcPTsdtLRzoDAQtPpHuoIrpRI_/view) or [Baidu Cloud](https://pan.baidu.com/s/1sQAC3pmceyochNvnqpE9oA#list/path=%2F) (The password is `mo8w`) \].
   - In the Baidu Cloud link, you can find the target file in the directory: `preprocessed_input/dtu.zip`.
 - Edit the testing setting in 'eval_dense.sh`:
   - `--testpath`: the path of the testing dataset.
   - `--num_depth`: the number of hypothesized depth planes in the cost volume The default value is 256.
   - `--loadckpt`: the path of the checkpoint file. For example: `./log-2020-07-01/model_00060000.ckpt`. It is noted that `model_00060000.ckpt` means the model saved with an iteration of 60000 steps. You can also replace the number with other available ones.
 - Generate the depth maps by running `./eval_dense.sh`.

### Fusion
 - To fuse the generated per-view depth maps to a 3D point cloud, we utilize the code of [fusibile](https://github.com/kysucix/fusibile) for depth fusion.
 - To build the binary executable file from [fusibile](https://github.com/kysucix/fusibile), please follow the following steps:
   - Enter the `fusion/fusibile` directory of this project.
   - Check the gpu architecture in your server and modify the corresponding settings in `CMakeList.txt`:
     - If 1080 Ti GPU is used, please add: `set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS};-O3 --use_fast_math --ptxas-options=-v -std=c++11 --compiler-options -Wall -gencode arch=compute_60,code=sm_60 -gencode arch=compute_60,code=sm_60)`.
     - If 2080 Ti GPU is used, please add: `set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS};-O3 --use_fast_math --ptxas-options=-v -std=c++11 --compiler-options -Wall -gencode arch=compute_75,code=sm_75 -gencode arch=compute_75,code=sm_75)`.
   - Create the directory by running `mkdir build`.
   - Enter the created directory, `cd build`.
   - Configure the CMake setting, `cmake ..`.
   - Build the project, `make`.
   - Then you can find the binary executable file named as `fusibile` in the `build` directory.
 - Go back to the `fusion/` directory and edit the fusion setting in `fusion.sh`:
   - `DTU_TEST_ROOT`: the path of the testing dataset.
   - `FUSIBILE_EXE_PATH`: the path of the binary executable `fusibile` file.
   - `--prob_threshold`/`--disp_threshold`/`--num_consistent`: hyperparamters for depth fusion.
 - Run the fusion code, `./fusion.sh`.
 - Move the fused 3D point cloud to the same folder, `python arange.py`. You can find the reconstructed 3D ply files in the directory of `outputs_dense/mvsnet_0.4_0.25`.

### Benchmark
 - To reproduce the quantitative performance of the model, we can use the official code provided by [DTU](http://roboimagedata.compute.dtu.dk/?page_id=36). The original codes are implemented in Matlab and requires huge time for calculating the evaluation metrics. We provide a parallel version of the code in `matlab_eval/dtu` of this repo.
 - Download the [Sample Set.zip](roboimagedata2.compute.dtu.dk/data/MVS/SampleSet.zip) and [Points.zip](http://roboimagedata2.compute.dtu.dk/data/MVS/Points.zip) from [DTU](http://roboimagedata.compute.dtu.dk/?page_id=36)'s official website. Decompress the zip files and arange the ground truth point clouds following the official instructions of [DTU](http://roboimagedata.compute.dtu.dk/?page_id=36).
 - Edit the path settings in `ComputeStat_web.m` and `BaseEvalMain_web.m`.
   - The `datapath` should be changed according to the path of your data. For example, `dataPath='/home/xhb/dtu_eval/SampleSet/MVS Data/';`
 - Enter the `matlab_eval/dtu` directory and run the matlab evaluation code, `./run.sh`. The results will be presented in a few hours. The time consumption is up to the available threads enabled in the Matlab environment. 

