# vldb-reproduce
Links to specific branches/commits needed to run all experiments

Run the following to copy submodule code 
```
git clone --recurse-submodules git@github.com:discslab-dl-bench/vldb-reproduce.git
```

We've tried to self-contain the various repositories, most have a `requirements.txt` that you can `pip install` to get the dependencies. You can also create a python virtual environement for each.


# Generate the plots from data
To regenerate the plots from the paper, you can download the timeline and instrumentation run data from

https://mcgill-my.sharepoint.com/:u:/g/personal/oana_balmau_mcgill_ca/EV8SApq87yJFtZfb_n_vgwYBoSZwNaKa4z7lCuEn_EGUmA?e=b8h4jL

https://mcgill-my.sharepoint.com/:u:/g/personal/oana_balmau_mcgill_ca/Ecmosl4mMFlDry-HH8oDDDIBKb1ZQJhLQ4s80M-BSa34JQ?e=HeLksK

decompress them in `trace_visuals` and run the following two scripts:

```
./trace_visuals/generate_timeline_plots.sh
./trace_visuals/generate_instrumentation_plots.sh
```

See the README in `trace_visuals` for more information.

# Rerun experiments / Run new experiments

Dependencies:
- bpftrace
- docker
- tmux
- nvidia-smi
- mpstat
- python3

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