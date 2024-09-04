# docker-demo
## Instructions to self for building, running, and pushing Docker containers to Docker Hub, pulling them into VT ARC's file system, and running a batch job with the new container
These instructions assume you have already created a DockerHub account and have downloaded Docker Desktop onto your computer.

### 1. Identify the container you would like to use as a starting point (e.g., rocker/ml-verse) on Docker Hub

### 2. Using Terminal, pull the container you have identified from Docker Hub to your computer
`docker image pull [OPTIONS] rocker/ml-verse:tag`
### 3. Create a working directory that you will use to house your Dockerfile, which is the file that is run to build the new container

### 4. In Terminal, navigate to that working directory, e.g.,
`cd Documents/DockerProjects`

### 5. Write the Dockerfile; you need to include the container you are building FROM and the RUN commands are where you add things to the container
```{bash}
FROM rocker/ml-verse

RUN pip install tqdm
```
### 6. Build the new Docker image in Terminal - make sure you have navigated to the correct directory in step 4! The -t indicates you are tagging the image. The period is important to tell the build function where the Dockerfile is.
`docker build -t melofton/ml-verse .`
### 7. Run the Docker container either using code commands in Terminal or from Docker Desktop. This builds a container to go with the new image you have just made. 

### 8. Navigate to your Docker Hub account (sign in). Create a new repository that matches the tag you gave your new container (e.g., melofton/ml-verse).

### 9. Push your new container to Docker Hub in Terminal.
`docker push melofton/ml-verse:latest`

### 10. Log into open-on-demand on VT's ARC and create an .sh file to pull the new Docker container into your files on ARC.
```{bash}
#!/bin/bash
#SBATCH -J build ml-verse container
#SBATCH --account=flare
#SBATCH --partition=normal_q
#SBATCH --nodes=1 --ntasks-per-node=1 --cpus-per-task=25 # this requests 1 node, 1 core.
#SBATCH --time=0-48:00:00 # 48 hr

module load containers/singularity
singularity build ml-verse.sif docker://melofton/ml-verse:latest
```
### 11. Go to Tinker Cliffs Shell access and submit the job. Make sure you either delete previous versions of the container with the same name or add --force to the .sh script or the job will fail.
`sbatch submit_job.sh`

### 12. Once your new container is on ARC, you can submit jobs using that container. Below is an example .sh file for such a job.
```{bash}
#!/bin/bash
#SBATCH -J multi-model-chla-prediction-model-runs
#SBATCH --account=flare
#SBATCH --partition=normal_q
#SBATCH --nodes=1 --ntasks-per-node=1 --cpus-per-task=25 # this requests 1 node, 1 core.
#SBATCH --time=0-48:00:00 # 48 hr

module load containers/singularity
singularity exec --bind=/home/melofton/multi-model-chla-prediction:/home/rstudio/multi-model-chla-prediction /home/melofton/ml-verse.sif /bin/bash run_r_LSTM.sh
```


