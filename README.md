# TrainingLogWizard
This tool automates the collection and analysis of DL training info contained in log files, consolidating the results into 1 excel spreadsheet.

# Description

## Context
The code provided within this repository is a collection of auxiliary function. The code containing the model, the parameter settings and data samples will be released -eventually- later on with the publication. However, since the challenge described in the *problem statement* is more related to the procedure and operations rather than the project focus itself, hopefully someone else will be able to benefit from it.

## Problem Statement
### Challenge 1: process monitoring
One of the great advantages of experimenting with distributed training using [Horovod](https://github.com/horovod/horovod) on top of an HPC cluster with [mpi](https://www.openmp.org/uncategorized/openmp-40/) consist in being able to split the job across multiple resources. However, while all instances are regularly syncing-up with one another from time to time as they are all contributing to the same training process, monitoring how performances are evolving (for debugging or model improvements) in each single part might be difficult. We can surely rely on Tensorflow (framework of choice in this case) to get the final results in terms of accuracy, loss and whatever other callback we may decide to expose, however -to the best of my knowledge- there is no easy way to get such granular visibility over the cores associated to each single instance.

In a nutshell, we can think of a DL training process being split in sub-instances *P1, P2, ..., Pn*, where each *Pi* performs its own indipendent piece of training resulting in loss, accuracy, etc.. What we would like to do is to capture all these details in a data format that we can lately use to better understand the behaviour of the distributed training.

### Challenge 2: multiple experiments
Challenge 1 is focussing on having a granular understading of what's going on within the multiple instances associated to 1 process.
However, as we know, in DL it is never matter of 1 process. Finding the correct set of hyperparameters is a must and takes multiple runs. While there are clever ways to perform the research, the whole activity boils down to looping over a bunch of parameters and collect the results achieved with a specific set of values.

In summary, considering the 2 challenges, what we have is an environment where:
1) We need to run multiple (>1000) experiments for hyperparameters tuning *(please NOTE: 1 experiment = 1 set of hyperparameter values)*
2) We want to track performances of each experiment, knowing that it may or may not be deployed using Data-Parallelism on MultiSocket and/or Multinode;
3) We need to access the results of all the experiments in a practical and effective way

## Proposed solution
This repo provides solutions for both the challenges. More in details:
1. To tackle challenge 1: __log_parser.py__
  - Cleans the log file by removing unrecognized characters and aligning row headers (printed by mpirun) with the correct line; 
  - Extracts and return a JSON pkg (excel format available too) containing all the training information such as: Loss, Accuracy, #Epochs (included early stopping); It also automatically generates a pdf containing the plots showing the perf. behaviour of each single instances
  
2. To takcle challenge 2: __aggregate_results.py__
Given an input directory containing all the experiments output (please see the "assumption" for the dirname formats), it loops over the the folders, load the JSON pkg produced by the *log_parser.py* and produce the excel spreadsheet organized as follows:
  - Rows = experiments
  - Columns = | processes per socket | cores per process | total processes | param ... | param N |	#training images	| learning rate |	momentum	| batch size | dropout	| total epochs | steps per epoch |	total training time	| early stop iteration	| val_loss	| val_acc	| val_custom_metric
