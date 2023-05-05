# vldb-reproduce
Links to specific branches/commits needed to run all experiments

Run the following when cloning this repository: 
```
git clone --recurse-submodules git@github.com:discslab-dl-bench/vldb-reproduce.git
```

We've tried to self-contain the various repositories, most have a `requirements.txt` that you can `pip install` to get the dependencies. You can also create a python virtual environement for each.


<br>

# Reproduce the paper's results from our data

## Generate the timleine and instrumentation measures plots
To regenerate the plots from the paper, you can download the timeline and instrumentation run data from

https://mcgill-my.sharepoint.com/:u:/g/personal/oana_balmau_mcgill_ca/EV8SApq87yJFtZfb_n_vgwYBoSZwNaKa4z7lCuEn_EGUmA?e=b8h4jL

https://mcgill-my.sharepoint.com/:u:/g/personal/oana_balmau_mcgill_ca/Ecmosl4mMFlDry-HH8oDDDIBKb1ZQJhLQ4s80M-BSa34JQ?e=HeLksK

decompress them in `trace_visuals` and run the following commands:

```
# Install project dependencies - you could also create a virtual environement
pip install -r trace_visuals/requirements.txt
./trace_visuals/generate_timeline_plots.sh
./trace_visuals/generate_instrumentation_plots.sh
```

See the README in `trace_visuals` for more information.

<br>


## Generate similarity metric results

Given the post-processed timeline data above, you can calculate their similarity with the following commands.
```
# Install project dependencies - you could also create a virtual environement
pip install -r tracesim/requirements.txt
./tracesim/generate_similarity_breakdowns.sh
```

A result file will be written for each real and benchmark workload pair in `tracesim/results/`.



<br>

# Rerun experiments / Run new experiments
You will need NVIDIA GPUs and the following installed on your system to run all experiments successfully:
```
- bpftrace
- docker
- tmux
- nvidia-smi
- mpstat
- python3
```

The general pattern for each experiment is the following:
- There is a submodule linking to the correct branch for each workload
- For each workload, you'll have to download the original dataset, and generate a synthetic one for the generated data experiment
- Then, you'll modify the launch scripts to point to your datasets and build the docker images.
- Finally, run the experiments and post-process the results. The launch scripts will be used to start the workload docker container and attach any traces. 

<br>

# Setting up all workloads

We will create a docker image for each workload. If you only want to run timeline experiments, you only need to setup the original workloads.

<br>

## UNET3D - Original

Follow the original workload instructions in `unet3d_original/README.md` to download and preprocess the real dataset, install dependencies etc.
Once that is done, build the workload's docker image and update the `OUTPUT_DIR` and `DATA_DIR` in the launch script `unet3d_original/start_training.sh` with your desired checkpoint output directory and the location of the original dataset.
```
docker build -t unet3d:original unet3d_original/
```

<br>

## UNET3D - Instrumented

The instrumented workload has some extra logging statements allowing us to measure time spent in each part of a training step.

For the baseline instrumentation experiments, we'll train the intrumented workload on the original dataset. For the generated data experiments, we'll train it on a synthetically generated dataset that we will now create:
```
# Generate the synthetic data of desired size for UNET3D -- in the paper we used the size of the original dataset.
pip install -r datagen/requirements.txt
python3 datagen/data_generation.py -o <output_dir> -s 30786160607 -w unet3d
```
Now, we can build the instrumented workload's docker image and update the `unet3d_instrumented/start_training.sh` and `unet3d_instrumented/start_training_on_generated.sh` launch scripts to point to the original and generated datasets, respectively, and the desired checkpoint directory.
```
docker build -t unet3d:instrumented unet3d_instrumented/
```

<br>

## UNET3D - Sleep

Finally, we'll build the docker image for the sleep workload, which replaces the real model computation with an appropriately long sleep time. This workload trains on the original dataset.

```
docker build -t unet3d:sleep unet3d_sleep/
```

Once again, don't forget to update the `unet3d_sleep/start_training.sh` script. This time tyou must also give the container the `WIKI_DIR`. 

<br>

## BERT - Original

Similarly, for BERT we'll follow the original workload instructions in `bert_original/README.md` to download and preprocess the dataset, then build the docker image and update the launch script `bert_original/start_training.sh` with your values of `WIKI_DIR`, `OUTPUT_DIR` and `DATA_DIR`.

```
docker build -t bert:original bert_original/
```

<br>

## BERT - Profiler

For BERT, we can't easily instrument the workload, so we'll use the Tensorflow profiler to export data instead. This workload has an extra profiler hook to do so and is equivalent to the instrumented versions of the other workloads.

```
docker build -t bert:profiler bert_instrumented/
```

We'll use this workload to train on the generated dataset, which can be created by running the datagen script again for BERT:

```
python3 datagen/data_generation.py -o <output_dir> -s 391434146993 -w bert
```
Now update the `bert_original/start_training.sh` and `bert_original/start_training_on_generated.sh` with your values of  `WIKI_DIR`, `OUTPUT_DIR` and `DATA_DIR`.

There is no sleep workload for BERT.

<br>

## DLRM - Original

Again for DLRM follow the original workload instructions in `dlrm_original/README.md` to download and preprocess the dataset. We use the Critero Terabyte Dataset, and preprocess it using the `--mlperf-bin-loader` option.

Then build the docker image 
```
docker build -t dlrm:original dlrm_original/
```
and update `dlrm_original/start_training.sh` with the correct `DATA_DIR` and `OUTPUT_DIR`.

<br>

## DLRM - Instrumented

Similarly for the instrumented workload, generate the synthetic dataset for the generated data experiments, build the docker image and update the launch scripts.

```
python3 datagen/data_generation.py -o <output_dir> -s 671231630720 -w dlrm

docker build -t dlrm:instrumented dlrm_instrumented/
```
Update `dlrm_instrumented/start_training.sh` and `dlrm_instrumented/start_training_on_generated.sh` with the correct `DATA_DIR` and `OUTPUT_DIR`.

<br>

## DLRM - Sleep

Same thing.
```
docker build -t dlrm:sleep dlrm_sleep/
```
Update `dlrm_sleep/start_training.sh` with the correct `DATA_DIR` and `OUTPUT_DIR`.


<br>

## Benchmark emulation

Now moving on to the Benchmark (also referred to as DLIO), we again have 'original' and 'instrumented' versions to be used either in timeline or instrumentation expeiments. We'll build these two images and they can be used to emulate the three workloads.

```
docker build -t dlio:original benchmark_original/

docker build -t dlio:instrumented benchmark_instrumented/
```

Once an image is built, we can generate the datasets for each emulated workload. This runs a similar procedure as the individual workload synthetic dataset generation, but the output is origanized slightly differently for the benchmark, so they cannot be straight-forwardly reused.

```
# Do this for each workload - the original and instrumented versions can share these datasets
./benchmark_original/start_datagen.sh <workload> dlio:original <data_dir>
```


<br>

# Running the experiments

Now we've built all the docker images and generated all the datasets, we're ready to run the experiments!

The data will be written to `tracing_tools/trace_results/` by default.

Note: the experimental run configs may not be able to run on your hardware and you may have to adapt them, i.e. if you have less than 8 GPUs or they don't have enough memory to support the maximum batch sizes we explore below.

<br>


## Timeline experiments

`tracing_tools/run_timeline_experiments.sh` will run the timeline experiments. These experiments run the original workloads and their emulation in a single configuration and capture eBPF trace data that will be used for plotting and the similarity measure.

Update the script with the checkpoint output directories of each workload, the docker image names if you've deviated from the ones used above and set `parent_repo_dir` to the path of this (`vldb-reproduce`) directory. 

<br>

## Instrumentation experiments

The paper's instrumentation experiments are all programmed in the `tracing_tools/run_instrumentation_experiments.sh` script. 

Update it to reflect the checkpoint output directories, the image names and the `parent_repo_dir` like for the timeline experiments then run it.

Note that we've run multiple instances of the instrumentation experiments.

<br>

# Post-processing the data

Once the experiments have been run, you should have the resulting data organized in a similar way as the original paper data linked to in the ["Reproduce the paper's results from our data" section](#generate-the-plots-from-data) and can follow similar instructions to process and plot it!


