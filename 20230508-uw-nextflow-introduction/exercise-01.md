# Exercise 1 - Create Nextflow profile for Beartooth

Nextflow allows users to create custom [configuration files](https://www.nextflow.io/docs/latest/config.html#)
for their systems. While this process is not always required, often times for HPC systems,
such as Beartooth, it is better to create a configuration file to allow Nextflow to directly
communicate with the scheduler. This will allow Nextflow to automatically manage the jobs,
dependencies, and resource allocations for you.

Nextflow [supports many executors](https://www.nextflow.io/docs/latest/executor.html#) out of
the box, but in the case of Beartooth, we will create a profile that will allow Nextflow to
submit jobs to [its SLURM scheduler](https://www.nextflow.io/docs/latest/executor.html#slurm). 

## Configuration File Structure

The configuration file is a simple text file that contains a list of key-value pairs. These
can be nested into sections to allow for more complex configurations. For example, the following
is a configuration file for [AWS Batch](https://aws.amazon.com/batch/), a cloud-scale scheduling
and compute management service from Amazon Web Services (AWS).

```bash
//Nextflow config file for running on AWS batch
params {
  config_profile_description = 'AWSBATCH Cloud Profile'
  config_profile_contact = 'Alexander Peltzer (@apeltzer)'
  config_profile_url = 'https://aws.amazon.com/batch/'

  awsqueue = false
  awsregion = 'eu-west-1'
  awscli = '/home/ec2-user/miniconda/bin/aws'
}

timeline {
  overwrite = true
}
report {
  overwrite = true
}
trace {
  overwrite = true
}
dag {
  overwrite = true
}

process.executor = 'awsbatch'
process.queue = params.awsqueue
aws.region = params.awsregion
aws.batch.cliPath = params.awscli
```

This configuration file is broken into a few sections:

* `params` - Key/value pairs that can be accessed throughout the pipeline
* `timeline`,`report`,`trace`,`dag` - Tell Nextflow to output useful run reports.

The `process` parameters `executor` and `queue` tell Nextflow to use the AWS Batch executor,
and which queue to use.

The `aws` parameters are what allow Nextflow to communicate with AWS Batch, such as which
region (`aws.region`) to use, and where the AWS CLI is located (`aws.batch.cliPath`) on the
virtual machine running the pipeline.

There are many pre-written Nextflow configuration files available on the
[nf-core/configs](https://github.com/nf-core/configs) repository. Many of these may be specific
to a users system, but they can be used as a starting point for creating your own configuration.

## Creating a Beartooth Configuration

We have a basic understanding of how the configuration file works, so let's create one for
Beartooth. We will start by creating a new directory for our configuration file.

```bash
mkdir nextflow-workshop
cd nextflow-workshop
touch beartooth.config
```

Next, we will open the configuration file in our favorite text editor. We will start by telling
Nextflow to use the SLURM executor.

```bash
// A configuration file for running Nextflow on Beartooth
process.executor = 'slurm'
```

Well, that's easy enough, we've told Nextflow to use the SLURM executor, but we haven't given
it any details about how to communicate with the scheduler. Let's add some more details.
For this let's take a look at the
[available SLURM options](https://www.nextflow.io/docs/latest/executor.html#slurm) in Nextflow.

| Option         | Description                                         |
|----------------|-----------------------------------------------------|
| clusterOptions | Additional options to pass to the SLURM scheduler   |
| cpus           | Maximum number of CPUs to use for each process      |
| memory         | Maximum amount of memory to use for each process    |
| queue          | Which queue to submit jobs to                       |
| time           | Maximum amount of time to allow each process to run |

For many pipelines `cpus`, `memory`, and `time` are already defined and will take priority over
the configuration file. However, we can still set these values in the configuration file to
be safe. So let's add these to our configuration file.

```bash
// A configuration file for running Nextflow on Beartooth
process.executor = 'slurm'
process.cpus = 1
process.memory = 1.GB
process.time = 60.m
```

That's better, but we still need to tell Nextflow which queue to submit jobs to. For this we'll
use the `queue` option.

```bash
// A configuration file for running Nextflow on Beartooth
process.executor = 'slurm'
process.cpus = 1
process.memory = 1.GB
process.time = 60.m
process.queue = 'moran'
```

Now we are telling Nextflow to submit jobs to the `moran` queue. You can provide more queues to
list by separating them with a comma, such as `process.queue = 'moran,teton'`. This will tell
Nextflow to submit to the `moran` and `teton` queues, the scheduler will then decide which queue
run the job on.

Final option, is the `clusterOptions` option. This allows us to pass additional options to the
scheduler. For users of BearTooth, this is where you will provide your account details
(`--account`).

So last thing to add:

```bash
// A configuration file for running Nextflow on Beartooth
process.executor = 'slurm'
process.cpus = 1
process.memory = 1.GB
process.time = 60.m
process.queue = 'moran'
process.clusterOptions = '--account=<YOUR_ACCOUNT>'
```

With this, you'll replace `<YOUR_ACCOUNT>` with your account name, and you'll have a basic 
config file for running Nextflow on Beartooth.

Let's try it:

```bash
module load nextflow
nextflow -version
nextflow run nextflow-io/hello -c beartooth.config
```

You should see the following output:

```bash
nextflow run nextflow-io/hello -c beartooth.config
N E X T F L O W  ~  version 22.10.4
Launching `https://github.com/nextflow-io/hello` [goofy_fermi] DSL2 - revision: 1d71f857bb [master]
executor >  slurm (4)
[94/c39a03] process > sayHello (3) [100%] 4 of 4 ✔
Hola world!

Bonjour world!

Ciao world!

Hello world!
```

The important part is `executor >  slurm (4)`, which tells us that Nextflow is using the SLURM
scheduler. If you do not see this, please ask for help because otherwise the jobs are running
on the login node, which is not good.

Cool, we have a workable configuration file for Beartooth!

## Optimizing the Configuration File

While the above configuration file will work, it is not optimal. For example, what if we want
to change the queue? or maybe the account? We would have to edit the configuration file each
time we want to make this change.

Instead, we can use the `params` section of the configuration file to allow us to change these
at runtime. So, let's update our configuration file to use the `params` section.

```bash
// A configuration file for running Nextflow on Beartooth
params {
  slurm_queue = 'moran'
  slurm_opts = '--account=<YOUR_ACCOUNT>'
}

process.executor = 'slurm'
process.cpus = 1
process.memory = 1.GB
process.time = 60.m
process.queue = "${params.slurm_queue}"
process.clusterOptions = "${params.slurm_opts}"
```

Now, we can change the queue and account at runtime by passing them to Nextflow with the
`--slurm_queue` and `--slurm_opts` options. For example:

```bash
module load nextflow
nextflow run nextflow-io/hello -c beartooth.config --slurm_queue teton
```

This will run the pipeline on the `teton` queue instead of the `moran` queue. This is getting
better, but did you notice how the `params` section is nested? We can do the same with the
`process` parameters. Let's update our configuration file to do this.

```bash
// A configuration file for running Nextflow on Beartooth
params {
  slurm_queue = 'moran'
  slurm_opts = '--account=<YOUR_ACCOUNT>'
}

process {
    executor = 'slurm'
    cpus = 1
    memory = 1.GB
    time = 60.m
    queue = "${params.slurm_queue}"
    clusterOptions = "${params.slurm_opts}"
}
```

Now we are starting to have a decent looking configuration file. Unfortunately, we still have
are missing something critical!

So, far we've only tested the configuration file with the `hello` pipeline. This pipeline only
requires `echo` and nothing else. What if we want to run a pipeline that requires a specific
tools and dependencies? For those we need to make use of Singularity containers, which Beartooth
supports.

## Adding Singularity Support

Beartooth supports Singularity containers, so we can use them in our configuration file. Let's
update our configuration file to use Singularity containers. Often times these containers are
specified in the pipeline itself, but we can also set a default container in the configuration.

Nextflow supports many different container types, but for this we'll use
[Singularity](https://www.nextflow.io/docs/latest/container.html#singularity).

We have access to the following options:

| Option         | Description                                                                        |
|----------------|------------------------------------------------------------------------------------|
| enabled        | Enable Singularity execution                                                       |
| engineOptions  | Provide any option supported by the Singularity engine                             |
| envWhitelist   | List of environment variable to be included in the container environment           |
| runOptions     | Provide any extra command line options supported by the singularity exec           |
| noHttps        | Pull the Singularity image with http protocol                                      |
| autoMounts     | Automatically mounts host paths in the executed container                          |
| cacheDir       | Directory where remote Singularity images are stored.                              |
| pullTimeout    | Maximum time to wait for a Singularity image to be pulled from a remote repository |
| registry       | The registry from where Docker images are pulled.                                  |

Many of these you likely will not need to change. More detail descriptions are available at
the [Nextflow Singulrity Scope](https://www.nextflow.io/docs/latest/singularity.html#scope)
documentation.

For our usage on Beartooth we'll need to use `enabled`, `autoMounts`, and `cacheDir`. So let's
add it to our configuration file.

```bash
// A configuration file for running Nextflow on Beartooth
params {
  slurm_queue = 'moran'
  slurm_opts = '--account=<YOUR_ACCOUNT>'
}

process {
    executor = 'slurm'
    cpus = 1
    memory = 1.GB
    time = 60.m
    queue = "${params.slurm_queue}"
    clusterOptions = "${params.slurm_opts}"
}

singularity {
    enabled = true
    autoMounts = true
    cacheDir = "/path/to/save/images"
}
```

Awesome! Now Nextflow will use Singularity containers when running pipelines. But, we should
probably make it so we can change the cache directory at runtime. Let's update our configuration:

```bash
// A configuration file for running Nextflow on Beartooth
params {
  slurm_queue = 'moran'
  slurm_opts = '--account=<YOUR_ACCOUNT>'
  singularity_cache = '/path/to/singularity/cache'
}

process {
    container = 'quay.io/nextflow/bash'
    executor = 'slurm'
    cpus = 1
    memory = 1.GB
    time = 60.m
    queue = "${params.slurm_queue}"
    clusterOptions = "${params.slurm_opts}"
}

singularity {
    enabled = true
    autoMounts = true
    cacheDir = "${params.singularity_cache}"
}
```

## Finalizing our Configuration File

We now have a configuration file that will work for Nextflow pipelines. Let's finalize it by
making it into a profile. This will allow us to specify it using the `-profile` option. This
is a very simple process, as we only need to wrap our configuration file with a `profile` block.

```bash
profiles {
    beartooth {
        // A configuration file for running Nextflow on Beartooth
        params {
            slurm_queue = 'moran'
            slurm_opts = '--account=<YOUR_ACCOUNT>'
            singularity_cache = '/path/to/singularity/cache'
        }

        process {
            container = 'quay.io/nextflow/bash'
            executor = 'slurm'
            cpus = 1
            memory = 1.GB
            time = 60.m
            queue = "${params.slurm_queue}"
            clusterOptions = "${params.slurm_opts}"
        }

        singularity {
            enabled = true
            autoMounts = true
            cacheDir = "${params.singularity_cache}"
        }
    }
}
```

That's it! Now we can use this profile to run any Nextflow pipeline on Beartooth. For example:

```bash
nextflow run nextflow-io/hello -profile beartooth -c beartooth.config
```

This will be more handy in the next section when we start to use pipelines with multiple
profiles.

## Wrapping Up

You should now have a Nextflow configuration file that will work on Beartooth. With this you
can begin to take advantage of Nextflow on Beartooth. Now it's time to start using it!

Please head on over to [Exercise 2](./exercise-02.md) to start using our configuration
file to run a few nf-core pipelines.
