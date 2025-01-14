# Biotrainer

*Biotrainer* is an open-source tool to simplify the training process of machine-learning models for biological
applications. It specializes on training models to predict features for **proteins**. 
Using *biotrainer* comes as simple as providing your sequence and label data in the 
[correct format](#data-standardization), along with a [configuration file](#example-configuration-file).

## Data standardization

*Biotrainer* provides a lot of data standards, designed to ease the usage of machine learning for biology. 
This standardization process is also expected to improve communication between different scientific disciplines
and help to keep the overview about the rapidly developing field of protein prediction.


### Available protocols

The protocol defines, how the input data should be interpreted and which kind of prediction task has to be applied.
The following protocols are already implemented:
```text
D=embedding dimension (e.g. 1024)
B=batch dimension (e.g. 30)
L=sequence dimension (e.g. 350)
C=number of classes (e.g. 13)

- residue_to_class --> Predict a class C for each residue encoded in D dimensions in a sequence of length L. Input BxLxD --> output BxLxC
- residues_to_class --> Predict a class C for all residues encoded in D dimensions in a sequence of length L. Input BxLxD --> output BxC
- sequence_to_class --> Predict a class C for each sequence encoded in a fixed dimension D. Input BxD --> output BxC
- sequence_to_value --> Predict a value V for each sequence encoded in a fixed dimension D. Input BxD --> output BxV
```

### Input file standardization

For every protocol, we created a standardization on how the input data must be provided. You can find detailed 
information for each protocol [here](docs/data_standardization.md).

Below, we show an example on how the sequence and label file must look like for the *residue_to_class* protocol:

**sequences.fasta**
```fasta
>Seq1
SEQWENCE
```

**labels.fasta**
```fasta
>Seq1 SET=train VALIDATION=False
DVCDVVDD
```

### Configuration file

To run *biotrainer*, you need to provide a configuration file in `.yaml` format along with your sequence and label data. 
Here you can find an exemplary file for the *residue_to_class* protocol. All configuration options are listed
[here](docs/config_file_options.md).

**Example configuration for residue_to_class**:
```yaml
protocol: residue_to_class
sequence_file: sequences.fasta # Specify your sequence file
labels_file: labels.fasta # Specify your label file
model_choice: CNN # Model architecture 
optimizer_choice: adam # Model optimizer
learning_rate: 1e-3 # Optimizer learning rate
loss_choice: cross_entropy_loss # Loss function 
use_class_weights: True # Balance class weights by using class sample size in the given dataset
num_epochs: 200 # Number of maximum epochs
batch_size: 128 # Batch size
embedder_name: prottrans_t5_xl_u50 # Embedder to use
```

### (Bio-)Embeddings

To convert the sequence data to more meaningful input for a model, embeddings generated by 
*protein language models* (pLMs) have become widely applied in the last years. 
Hence, *biotrainer* enables automatic calculation of embeddings on a *per-sequence* and *per-residue* level, 
depending on the protocol. 
Take a look at [bio_embeddings](https://github.com/sacdallago/bio_embeddings/) to find out about all the available
embedding methods. However, it is also possible to provide your own embeddings file using your own embedder, 
independent of *bio_embeddings*. Please refer to the 
[data standardization](docs/data_standardization.md#embeddings) document and the 
[relevant examples](examples/custom_embeddings/) to learn how to do this. Pre-calculated embeddings can be used for
the training process via the `embeddings_file` parameter, 
as described in the [configuration options](docs/config_file_options.md#embeddings).

## Installation

1. Make sure you have [poetry](https://python-poetry.org/) installed: 
```bash
curl -sSL https://install.python-poetry.org/ | python3 - --version 1.4.2
```

2. Install dependencies and biotrainer via `poetry`:
```bash
# In the base directory:
poetry install
# Optional: Add bio-embeddings to compute embeddings
poetry install --extras "bio-embeddings"
# Optional: Add jupyter to use notebooks
poetry install --extras "jupyter"
# You can also install all extras at once
poetry install --all-extras
```

## Running

```bash
cd examples/residue_to_class
poetry run biotrainer config.yml
```

You can also use the provided `run-biotrainer.py` file for development and debugging (you might want to set up your 
IDE to directly execute run-biotrainer.py with the provided virtual environment):
```bash
# residue_to_class
poetry run python3 run-biotrainer.py examples/residue_to_class/config.yml
# sequence_to_class
poetry run python3 biotrainer.py examples/sequence_to_class/config.yml
```

### Docker

```bash
# Build
docker build -t biotrainer .
# Run
docker run --rm \
    -v "$(pwd)/examples/docker":/mnt \
    -v bio_embeddings_weights_cache:/root/.cache/bio_embeddings \
    -u $(id -u ${USER}):$(id -g ${USER}) \
    biotrainer:latest /mnt/config.yml
```

Output can be found afterwards in the directory of the provided configuration file.


## Citation

If you are using *biotrainer* for your work, please add a citation:

```text
@inproceedings{
sanchez2022standards,
title={Standards, tooling and benchmarks to probe representation learning on proteins},
author={Joaquin Gomez Sanchez and Sebastian Franz and Michael Heinzinger and Burkhard Rost and Christian Dallago},
booktitle={NeurIPS 2022 Workshop on Learning Meaningful Representations of Life},
year={2022},
url={https://openreview.net/forum?id=adODyN-eeJ8}
}
```
