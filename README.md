# 1. Overview of PEP environment configuration

To use PEP project objects (or `looper`) with a cluster resource manager (SGE, SLURM, *etc.*) or with linux containers (`docker`, `singularity`, *etc.*), you have to provide a few configuration settings. We refer to this as the *computing environment configuration*. This repository provides templates to help you set up cluster computing, containerized computing, or both.

# 2. Setting up your environment

## Plugging into a compute environment where PEPENV is already deployed

For server environments where `looper` is being actively used, this repository already contains the correct configuration settings. If you're lucky enough to be at one of these places, set-up is very simple. Here's a list of pre-configured computing environments:

   * `uva_rivanna.yaml`: [Rivanna cluster](http://arcs.virginia.edu/rivanna) at University of Virginia
   * `cemm.yaml`: Cluster at the Center for Molecular Medicine, Vienna
   * `nih_biowulf2.yaml`: [Biowulf2](https://hpc.nih.gov/docs/userguide.html) cluster at the NIH
   * `stanford_sherlock.yaml`: [Sherlock](http://sherlock.stanford.edu/mediawiki/index.php/Current_policies) cluster at Stanford
   * `ski-cer_lilac.yaml`: *lilac* cluster at Memorial Sloan Kettering
   * `local_containers.yaml`: A generic local desktop or server (with no cluster management system) that will use docker or singularity containers.

To set up your looper to use cluster resources at one of these locations, all you have to do is:

1. Clone this repository
2. Point the `$PEPENV` environment variable to the appropriate config file by executing this command:
	```
	export PEPENV=path/to/compute_config.yaml
	```
 	(Add this line to your `.profile` or `.bashrc` if you want it to persist).

And that's it, you're done! If the existing config files do not fit your environment, you will need to create a PEPENV config file to match your environment by following these instructions:

## Configuring a new environment

To configure a new environment, we'll follow the same steps, but just point at the default file, `compute_config.yaml`, which we will then edit to match your local computing environment.

1. Clone this repository
2. Point the `$PEPENV` environment variable to the **default config file** by executing this command:
  ```
  export PEPENV=path/to/compute_config.yaml
  ```
  (Add this line to your `.profile` or `.bashrc` if you want it to persist).

3. Next, use this file as a starting point to configure your environment. Likely, the only thing you will need to change is the `partition` variable, which should reflect your submission queue or partition name used by your cluster resource manager.

4. Once you have it working, consider submitting your configuration file back to this repository with a pull request.


# 3. PEPENV configuration explained

The `PEPENV` computing environment configuration consists of two components: 1) The `PEPENV` file itself, and 2) a series of template files.

## The PEPENV file

The PEPENV file is a `yaml` file listing different *compute packages*. Consider an example environment configuration file:

```
compute:
  default:
    submission_template: templates/local_template.sub
    submission_command: sh
  local:
    submission_template: templates/local_template.sub
    submission_command: sh
  develop:
    submission_template: templates/slurm_template.sub
    submission_command: sbatch
    partition: develop
  big:
    submission_template: templates/slurm_template.sub
    submission_command: sbatch
    partition: bigmem
  ```

The sub-sections below `compute` each define a *compute package* that can be activated. By default, the package named `default` will be used, which in this case is identical to the `local` package. You can make your default whatever you like. You can switch from local compute to the `develop` partition __on the fly__ by specifying the `--compute` argument to `looper run` like so:

```
looper run --compute develop
```

Generically, you just use `looper run --compute PACKAGE`, and PACKAGE could be `local` (which in this case does the same thing as the default) or `develop` or `big`, which would run the jobs on slurm, with queue `develop`, or `bigmem`. You can make as many compute packages as you wish, and name them whatever you wish. You can also add whatever attributes to the compute package. They should at least have `submission_template` and `submission_command`, but you can also add other variables which could populate parts of your templates. Each compute package specifies a path to a template file (`submission_template`). These paths can be relative or absolute; relative paths are considered *relative to the pepenv file*.

## Templates files

Template files are taken by looper, populated with sample-specific information, and then run as scripts. Here's an example of a generic SLURM template file:

```{bash}
#!/bin/bash
#SBATCH --job-name='{JOBNAME}'
#SBATCH --output='{LOGFILE}'
#SBATCH --mem='{MEM}'
#SBATCH --cpus-per-task='{CORES}'
#SBATCH --time='{TIME}'
#SBATCH --partition='{PARTITION}'
#SBATCH -m block
#SBATCH --ntasks=1

echo 'Compute node:' `hostname`
echo 'Start time:' `date +'%Y-%m-%d %T'`

{CODE}
```

Template files use variables (encoded like `{VARIABLE}`), which will be populated independently for each sample. The variables specified in these template files (like `{LOGFILE}` or `{CORES}`) are replaced by looper when it creates a job script.

This `pepenv` repository comes with some commonly used templates (in the [templates](/templates) folder):
* SLURM: [slurm_template.sub](/templates/slurm_template.sub)
* SGE: [sge_template.sub](/templates/sge_template.sub)
* localhost (compute locally): [localhost_template.sub](/tempaltes/localhost_template.sub)

Most users will not need to tweak the template files. But, you can also create your own templates, giving looper ultimate flexibility to work with any compute infrastructure in any environment. To create a custom template, just look through the examples and build a new template file. Then, point your `pepenv` config file to your custom template using the `submission_template` attribute. If you make a useful template, please contribute your new templates back to this repository!

### The source of the values

What is the source of values used to populate the variables? Well, they are pooled together from several sources. To start, there are a few built-ins:

Built-in variables:
- `{CODE}` is a reserved variable that refers to the actual command string that will run the pipeline.
- `{JOBNAME}` -- automatically produced by looper using the `sample_name` and the pipeline name.
- `{LOGFILE}` -- automatically produced by looper using the `sample_name` and the pipeline name.


Other variables are not automatically created by `looper` and are specified in a few different places:

*PEPENV*. First, you have access to any variables specified in the `PEPENV` file for that compute package. The `{PARTITION}` variable is an example of this; that variable will be populated from the `partition` attribute for the compute package pulled from the `PEPENV` file.

*pipeline_interface.yaml*. Variables that are specific to a pipeline can be defined in the pipeline interface file. These variables are pulled from two different sections: `compute` and `resources`. The difference between the two is that variables in the `compute` section are common to all samples, while variables in the `resources` section vary based on sample input size. As an example of a variable pulled from the `compute` section, we defined in our pipeline interface a variable pointing to the singularity or docker image that can be used to run the pipeline, like this:

```
compute:
  singularity_image: path/to/images/image
```

Now, this variable will be available for use in a template as `{SINGULARITY_IMAGE}`. The other pipeline interface section is `resources`. Some examples of variables we use in the existing templates are these:
- `{MEM}` -- pulled from the `resources` section of the [`pipeline_interface`](http://looper.readthedocs.io/en/latest/pipeline-interface.html) file.
- `{CORES}` -- pulled from the `resources` section of the [`pipeline_interface`](http://looper.readthedocs.io/en/latest/pipeline-interface.html) file.
- `{TIME}` -- pulled from the `resources` section of the [`pipeline_interface`](http://looper.readthedocs.io/en/latest/pipeline-interface.html) file.

[Read more about pipeline_interface.yaml here](http://looper.readthedocs.io/en/latest/pipeline-interface.html).

*project_config.yaml*. Finally, project-level variables can also be populated from the `compute` section of a project config file.


# 4. Using docker or singularity containers

The `PEPENV` framework is a natural way to run commands in a container. All we need to do is 1) design a template that will run the job in the container, instead of natively; and 2) create a new compute package that will use that template. 

This repository includes templates for the following scenarios:

- singularity on SLURM: [slurm_singularity_template.sub](templates/slurm_singularity_template.sub)
- singularity on localhost: [localhost_singularity_template.sub](templates/localhost_singularity_template.sub)
- docker on localhost: [localhost_singularity_template.sub](templates/localhost_singularity_template.sub)

If you need a different system, looking at those examples should get you started toward making your own.

To add a package for these templates to a `PEPENV` file, we just add a new section. Take a look at the basic `localhost_container.yaml` PEPENV file:

```
compute:
  default:
    submission_template: templates/localhost_template.sub
    submission_command: sh
  singularity:
    submission_template: templates/localhost_singularity_template.sub
    submission_command: sh
    singularity_args: -B /ext:/ext
  docker:
    submission_template: templates/localhost_docker_template.sub
    submission_command: sh
    docker_args: |
      --user=$(id -u) \
      --env="DISPLAY" \
      $(env | cut -f1 -d= | sed 's/^/-e /') \
      --volume ${HOME}:${HOME} \
      --volume="/etc/group:/etc/group:ro" \
      --volume="/etc/passwd:/etc/passwd:ro" \
      --volume="/etc/shadow:/etc/shadow:ro"  \
      --volume="/etc/sudoers.d:/etc/sudoers.d:ro" \
      --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
      --workdir="`pwd`" \
```

This should work out of the box for most users; if you want to run a singularity container, just specify `--compute singularity` and it will use the appropriate template (and likewise for `docker`). You will just need to make sure your `pipeline_interface.yaml` file contains a `compute: singularity_image:` attribute pointing it to the image you want to use.

