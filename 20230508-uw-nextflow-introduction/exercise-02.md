# Exercise 2: Run existing Nextflow pipelines

OK! By now you should have a functional Nextflow profile for Beartooth. If not, please return
back to [Exercise 1 - Create Beartooth profile for Nextflow](./exercise-1.md) and complete it.

For the remainder of this workshop, we will be using the profile we created in Exercise 1 to
run existing workflows. For this, we'll focus on a few nf-core pipelines. [nf-core](https://nf-co.re/)
is a community effort to collect a curated set of high quality, well tested, and reproducible
Nextflow pipelines. We'll focus on these because they each have a `test` profile for a quick
and easy way to run the pipeline.

## nf-core pipelines

Currently there are more than [80 pipelines](https://nf-co.re/pipelines) available from nf-core.
Each of these pipelines has undergone extensive testing and review to ensure that they work
as expected. nf-core pipelines, while a majority are bioinformatics themed, there is a growing
number of non-bioinformatics nf-core pipelines.

For the next few minutes, we'll try executing a few nf-core pipelines on Beartooth. You are
free to try which ever you want, but we'll document `viralrecon`, `spinningjenny`, and `rnaseq`.

### nf-core/viralrecon

[nf-core/viralrecon](https://nf-co.re/viralrecon) is a pipeline for viral pathogen detection
and has played a critical role for many public health labs for the on-going surveillance of
SARS-CoV-2.

Now, let's test viralrecon using it's test profile along with our Beartooth profile.

```bash
module load nextflow
nextflow run nf-core/viralrecon --help

# Show all hidden parameters
nextflow run nf-core/viralrecon --help_all

# Run the test profile
nextflow run nf-core/viralrecon -profile test,beartooth -c beartooth.config --outdir test_viralrecon
```


### nf-core/spinningjenny

[nf-core/spinningjenny](https://nf-co.re/spinningjenny) is a pipeline for simulating the first industrial revolution
using Agent Based Models. spinningjenny is a great example of a non-bioinformatics pipeline taking adavantage of 
Nextflow's features.

Let's to a quick test run of spinningjenny.

```bash
module load nextflow
nextflow run nf-core/spinningjenny --help

# Show all hidden parameters
nextflow run nf-core/spinningjenny --help_all

# Run the test profile
nextflow run nf-core/spinningjenny -profile test,beartooth -c beartooth.config --outdir test_spinningjenny
```

### nf-core/rnaseq

[nf-core/rnaseq](https://nf-co.re/rnaseq) is a pipeline for RNA sequencing data analysis. For this you'll need
RNA-seq data with a reference genome and annotation file. nf-core/rnaseq is an extensive pipeline with many
common tools and provides a very useful starting point for RNA-seq analysis.

Let's test it out using our profile.

```bash
module load nextflow
nextflow run nf-core/rnaseq --help

# Show all hidden parameters
nextflow run nf-core/rnaseq --help_all

# Run the test profile
nextflow run nf-core/rnaseq -profile test,beartooth -c beartooth.config --outdir test_rnaseq
```


## Wrapping Up

Congratulations! You've successfully run a few Nextflow pipelines on Beartooth. From here, you
can begin to learn more about Nextflow and how you might be able to use it in your own research.
