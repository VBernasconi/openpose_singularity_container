# Singularity container for openpose environment
This repository holds the definition file for a singularity container that is used on a virtual machine on a cloud. The environment allows you to run CMU Openpose https://github.com/CMU-Perceptual-Computing-Lab/openpose. I do not hold any rights for the openpose project.

Change the cuda and cudnn versions on the .def file according to what is installed on your virtual machine. Then you can create your environment and rsync it to your VM.
```
sudo singularity build openpose_env_04.sif openpose_env_04.def
```
Then you can ssh to your VM. The modules that have to be used on your host are:
 * vesta partition
 * singularity
 * cmake
 * nvidia/cuda11.2-cudnn8.1.0

Go to the directory holding your .sif file and build the environment
```
singularity build --sandbox openpose_env openpose_env_04.sif
```
Once the environment is created, clone the openpose repository and place your singularity environment inside the openpose folder
```
git clone https://github.com/CMU-Perceptual-Computin-Lab/openpose
mv openpose_env_04_2022 openpose/
cd openpose
```
Then you can launch the singularity environment (-B information is for the directories present in your VM that you need to access. In my case, the openpose folder is located in data, my images in scratch)
```
singularity shell -B /data -B /scratch openpose_env
```
From there, either you are already in the openpose folder, or you have to move inside the openpose folder. Then, run the following commands:
```
mkdir build
cd build && cmake .. -DUSE_CUDNN=OFF && make-j`nproc`
```
Once the installation is done, enter exit. Then you can create your .sh file to launch the job on your images
```
#!/usr/bin/env bash
#SBATCH --gres gpu:2
#SBATCH --mem=600GB
#SBATCH --time=80:00:00

singularity shell -B /data -B /scratch openpose_env ./build/examples/openpose/openpose.bin --hand --image_dir ~/scratch/dataset --write_json ~/scratch/output_json --render_pose 0 --display 0 --model_pose BODY_25
```
