# building the image
cd ubuntu20.04-cuda11.4.2-noetic/ && \
sudo docker build -t gpu-voxels-ubuntu20.04-cuda11.4.2-noetic .

# starting a container and running tests

sudo docker run --rm -ti gpu-voxels-ubuntu20.04-cuda11.4.2-noetic "cd /opt/gpu-voxels/build && ls -l bin/" # show binaries
sudo docker run --runtime=nvidia --rm -ti nvidia/cuda:11.0-base bash # for docker 18 "--gpus all" does not yet exist?
sudo docker run --rm --gpus all -ti gpu-voxels-ubuntu20.04-cuda11.4.2-noetic nvidia-smi #test GPU driver
sudo docker run --rm --gpus all -ti gpu-voxels-ubuntu20.04-cuda11.4.2-noetic "cd /opt/gpu-voxels/build && ./bin/test_" #run tests

