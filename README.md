# Flow Gaussian Mixture Model (FlowGMM)
This repository contains a PyTorch implementation of the Flow Gaussian Mixture Model (FlowGMM) model from our paper

[Semi-Supervised Learning with Normalizing Flows ](https://invertibleworkshop.github.io/accepted_papers/pdfs/INNF_2019_paper_28.pdf)

by Pavel Izmailov, Polina Kirichenko, Marc Finzi and Andrew Gordon Wilson.

# Introduction

![Screenshot from 2019-12-29 19-32-26](https://user-images.githubusercontent.com/14368801/71559657-fa771280-2a71-11ea-8deb-5b3b422c6c8f.png)


# Dependencies
* [PyTorch](http://pytorch.org/) version 1.0.1
* [torchvision](https://github.com/pytorch/vision/) version 0.2.1
* [tensorboardX](https://github.com/lanpa/tensorboardX)

# Usage

We provide the scripts and example commands to reproduce the experiments from the paper. 

## Synthetic Datasets

The experiments on synthetic data are implemented in [this ipython notebook](https://github.com/izmailovpavel/flow_ssl/blob/public/experiments/synthetic_data/synthetic.ipynb).
We additionaly provide [another ipython notebook](https://github.com/izmailovpavel/flow_ssl/blob/public/experiments/synthetic_data/synthetic-labeled-only.ipynb)
applying FlowGMM to labeled data only. 

## Image Classification

To run experiments with FlowGMM on image classification problems you first need to download and prepare the data.
To do so, run the following scripts:
```bash
./data/bin/prepare_cifar10.sh
./data/bin/prepare_mnist.sh
./data/bin/prepare_svhn.sh
```

To run FlowGMM, you can use the following script
```bash
python3 experiments/train_flows/train_semisup_cons.py \
  --dataset=<DATASET> \
  --data_path=<DATAPATH> \
  --label_path=<LABELPATH> \
  --logdir=<LOGDIR> \
  --ckptdir=<CKPTDIR> \
  --save_freq=<SAVEFREQ> \ 
  --num_epochs=<EPOCHS> \
  --label_weight=<LABELWEIGHT> \
  --consistency_weight=<CONSISTENCYWEIGHT> \
  --consistency_rampup=<CONSISTENCYRAMPUP> \
  --lr=<LR> \
  --eval_freq=<EVALFREQ> \
```
Parameters:

* ```DATASET``` &mdash; dataset name [MNIST/CIFAR10/SVHN]
* ```DATAPATH``` &mdash; path to the directory containing data; if you used the data preparation scripts, you can use e.g. `data/images/mnist` as `DATAPATH`
* ```LABELPATH``` &mdash; path to the label split generated by the data preparation scripts; this can be e.g. `data/labels/mnist/1000_balanced_labels/10.npz` or `data/labels/cifar10/1000_balanced_labels/10.txt`.
* ```LOGDIR``` &mdash; directory where tensorboard logs will be stored
* ```CKPTDIR``` &mdash; directory where checkpoints will be stored
* ```SAVEFREQ``` &mdash; frequency of saving checkpoints in epochs
* ```EPOCHS``` &mdash; number of training epochs (passes through labeled data)
* ```LABELWEIGHT``` &mdash; weight of cross-entropy loss term (default: `1.`)
* ```CONSISTENCYWEIGHT``` &mdash; weight of consistency loss term (default: `1.`)
* ```CONSISTENCYRAMPUP``` &mdash; length of consistency ramp-up period in epochs (default: `1`); consistency weight is linearly increasing from  `0.` to `CONSISTENCYWEIGHT` in the first `CONSISTENCYRAMPUP` epochs of training
* ```LR``` &mdash; learning rate (default: `1e-3`)
* ```EVALFREQ``` &mdash; number of epochs between evaluation (default: `1`)


Examples:

```bash
# MNIST, 100 labeled datapoints
python3 experiments/train_flows/train_semisup_cons.py --dataset=MNIST --data_path=data/images/mnist/ \
  --label_path=data/labels/mnist/100_balanced_labels/10.npz --logdir=<LOGDIR> --ckptdir=<CKPTDIR> \
  --save_freq=5000 --num_epochs=30001 --label_weight=3 --consistency_weight=1. --consistency_rampup=1000 \
  --lr=1e-5 --eval_freq=100 
  
# CIFAR-10, 4000 labeled datapoints
python3 experiments/train_flows/train_semisup_cons.py --dataset=CIFAR10 --data_path=data/images/cifar/cifar10/by-image/ \
  --label_path=data/labels/cifar10/4000_balanced_labels/10.txt --logdir=<LOGDIR> --ckptdir=<CKPTDIR> \ 
  --save_freq=500 --num_epochs=1501 --label_weight=3 --consistency_weight=1. --consistency_rampup=100 \
  --lr=1e-4 --eval_freq=50
  ```

## Tabular Data: UCI and NLP

For class balanced data splitting and for training of FlowGMM on the UCI and NLP datasets, we use the
[snake-oil-ml](https://github.com/mfinzi/snake-oil-ml/) library.

### UCI Data Preparation

Downdload the miniboone and hepmass datasets [here](https://zenodo.org/record/1161203#.Wmtf_XVl8eN)
We follow the preprocessing (where sensible) from [Masked Autoregressive Flow for Density Estimation](https://github.com/gpapamak/maf).
Unpack the files into a reasonable location (the default expected location for the files are ~/datasets/UCI/hepmass/ and ~/datasets/UCI/miniboone/)

### NLP Data Preparation

To run experiments on the text data, you first need to download the data and compute the BERT embeddings. To get the data run `data/nlp_datasets/get_text_classification_data.sh`. 
Then, you [this ipython notebook](https://github.com/izmailovpavel/flow_ssl/blob/public/data/nlp_datasets/text_preprocessing/AGNewsPreprocessing.ipynb) shows an example of computing BERT embeddings for the data.

### Running the Models

After the data has been prepared, the notebook [here](https://github.com/izmailovpavel/flow_ssl/blob/public/experiments/baselines/graphssl.ipynb) can be used to run the kNN, Logistic Regression, and Label Spreading baselines.

The 3-Layer NN with dropout and Pi-Model baseline experiments are implemented in [train_semisup_text_baselines.py](https://github.com/izmailovpavel/flow_ssl/blob/public/experiments/train_flows/train_semisup_text_baselines.py).

Finally the FlowGMM method can be trained on these datasets using [train_semisup_flowgmm_tabular.py](https://github.com/izmailovpavel/flow_ssl/blob/public/experiments/train_flows/train_semisup_flowgmm_tabular.py).


# References

* RealNVP: [github.com/chrischute/real-nvp](https://github.com/chrischute/real-nvp)
