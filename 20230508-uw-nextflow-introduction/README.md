# Using Nextflow on Beartooth

In this workshop, we will learn how to use Nextflow to run pipelines on Beartooth. We will
cover the following topics:

* Workflow Managers and Nextflow
* How to use Nextflow on Beartooth
* Submitting jobs to the SLURM scheduler
* Running existing nf-core pipelines
* Example use case applying Nextflow to statistical analysis

## Exercises

| Exercise                      | Description                           |
|-------------------------------|---------------------------------------|
| [Exercise 1](./exercise-1.md) | Create Beartooth profile for Nextflow |
| [Exercise 2](./exercise-2.md) | Run existing Nextflow pipelines       |

### Exercise 1: Create Beartooth profile for Nextflow

In this exercise we will create a profile for Beartooth in Nextflow. This will allow Nextflow
communicate with the Beartooth SLURM scheduler and submit jobs to the cluster directly. A major
benefit of this approach is the Nextflow will automatically manage the jobs, dependencies,
and resource allocation for you.

To get start more details please head over to [Exercise 1](./exercise-1.md).

### Exercise 2: Run existing Nextflow pipelines

In this exercise we will run existing Nextflow pipelines on Beartooth. We will make use of
the configuration we created in Exercise 1. While you are welcome to run which ever pipeline
you like, we will be demonstrate nf-core pipelines using their built in test profiles.

To get start more details please head over to [Exercise 2](./exercise-2.md).
