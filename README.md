# vldb-reproduce
Links to specific branches/commits needed to run all experiments

Run the following to copy submodule code 
```
git clone --recurse-submodules git@github.com:discslab-dl-bench/vldb-reproduce.git
```

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