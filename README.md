# vldb-reproduce
Links to specific branches/commits needed to run all experiments

Run the following to copy submodule code 
```
git clone --recurse-submodules git@github.com:discslab-dl-bench/vldb-reproduce.git
```

Dependencies:
bpftrace
docker
nvidia-smi
mpstat
python3

We've tried to self-contain the various repositories.


# Generate the plots from data


# Rerun all experiments from scratch

General pattern for each experiment:
- you have a submodule linking to the correct branch
- download/generate the dataset
- modify the launch scripts to point to your dataset
- build the docker image of that branch, mounting the correct directories
- run the experiment through the experiment runner script
- postprocess traces and generate plots

# UNET3D 
## Original workload

Go into `unet3d_original` and follow the original instrumentations in the README to download and preprocess the real dataset, install dependencies etc.
Building the docker image by running `docker build -t unet3d:original .` while in `unet3d_original`.

## Generate the synthetic datasets for each workload
```
python3 datagen/data_generation.py -o <output_dir> -s <size> -w [unet3d, dlrm, bert]
```

# Benchmark runs
Go into the `benchmark/` submodule and update the `start_datagen.sh` and `start_training.sh` scripts with your desired directories for data, logs and checkpoints. Optionally change the default workload configurations in `configs/workload`. Then build the docker image:
```
cd benchmark/
docker build -t dlio:latest .
```

Generate the dataset for the workload you want to run, then run the benchmark:
```
./benchmark/start_datagen.sh <workload>
./benchmark/start_training.sh <workload>
```