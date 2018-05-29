# PEP environment files

To use PEP project objects (or `looper`) with a cluster resource manager (SGE, SLURM, *etc.*) or with linux containers (`docker`, `singularity`, *etc.*), you have to provide a few configuration settings. We refer to this as the *computing environment configuration*. This repository provides templates to help you set up cluster computing, containerized computing, or both.

## Setting up your environment

If you're lucky enough to be using a computing environment where `looper` is already deployed, set-up is very simple.

1. Clone this repository
2. Point an environment variable (`$PEPENV`) to the appropriate config file in this repository (add this to your `.profile` or `.bashrc` if you want it to persist).

	```
	export PEPENV=path/to/compute_config.yaml
	```

	In this repository are a few files that set up configuration at places where looper is in use. Just point PEPENV to the appropriate one of these if there's a match:
   * `uva_rivanna.yaml`: [Rivanna cluster](http://arcs.virginia.edu/rivanna) at University of Virginia
   * `cemm.yaml`: Cluster at the Center for Molecular Medicine, Vienna
   * `nih_biowulf2.yaml`: [Biowulf2](https://hpc.nih.gov/docs/userguide.html) cluster at the NIH
   * `stanford_sherlock.yaml`: [Sherlock](http://sherlock.stanford.edu/mediawiki/index.php/Current_policies) cluster at Stanford
   * `ski-cer_lilac.yaml`: *lilac* cluster at Memorial Sloan Kettering
   * `local_containers.yaml`: A generic local desktop or server (with no cluster management system) that will use docker or singularity containers.

   And that's it, you're done! If the existing config files do not fit your environment, you will need to edit the config file to match your environment by following these instructions:

## Configuring a new environment

To start, just point at the default file, `compute_config.yaml`. This file is as a starting point to configure your own. Look at the contents of [compute_config.yaml](compute_config.yaml)) and customize for your compute environment. Likely, the only thing you will need to change is the `partition` variable, which should reflect your submission queue or partition name used by your cluster resource manager.

## PEPENV configuration file explained

Consider an example environment configuration file:

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

Generically, you just use `looper run --compute PACKAGE`, and PACKAGE could be `local` (which in this case does the same thing as the default) or `develop` or `big`, which would run the jobs on slurm, with queue `develop`, or `bigmem`. You can make as many compute packages as you wish.

## Using docker or singularity containers

To run a command in a container, basically we just have to pass that command to some other command, like `docker run`, so using the `PEPENV` framework is a natural way to solve this problem. Take a look at the basic `localhost_container.yaml` configuration file:

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

This should work out of the box for most users; if you want to run a singularity container, just specify `--compute singularity` and it will use the appropriate template (and likewise for `docker`). You will just need to make sure your `project_config.yaml` file contains a `compute.image` attribute pointing it to the image you want to use.

## Understanding templates

Most users will not need to tweak the template files. It will only be useful if you have particular computing environment needs and don't fit into these generic submission templates. If you find this is the case, please contribute your new templates back to this repository! These instructions should get you started on writing your own template:

Each compute package specifies a path to a template file (`submission_template`). These paths can be relative or absolute; relative paths are considered *relative to the pepenv file*. This `pepenv` repository comes with some commonly used templates (in the [templates](/templates) folder):
* SLURM: [slurm_template.sub](/templates/slurm_template.sub)
* SGE: [sge_template.sub](/templates/sge_template.sub)
* localhost (compute locally): [localhost_template.sub](/tempaltes/localhost_template.sub)

Here's an example of a generic SLURM template file:

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

A template file uses variables (encoded like `{VARIABLE}`), which will be populated independently for each sample. The variables specified in these template files (like `{LOGFILE}` or `{CORES}`) are replaced by looper when it creates a job script. It populates the variables with information from a few different sources. Some of the variables are built-ins, and some are user-customizable:

Built-in variables:
- `{CODE}` is a reserved variable that refers to the actual command string that will run the pipeline.
- `{JOBNAME}` -- automatically produced by looper using the `sample_name` and the pipeline name.
- `{LOGFILE}` -- automatically produced by looper using the `sample_name` and the pipeline name.

User-defined variables:
- `{MEM}` -- pulled from the `resources` section of the [`pipeline_interface`](http://looper.readthedocs.io/en/latest/pipeline-interface.html) file.
- `{CORES}` -- pulled from the `resources` section of the [`pipeline_interface`](http://looper.readthedocs.io/en/latest/pipeline-interface.html) file.
- `{TIME}` -- pulled from the `resources` section of the [`pipeline_interface`](http://looper.readthedocs.io/en/latest/pipeline-interface.html) file.
- `{PARTITION}` -- pulled from the `compute` section of the `pepenv` file.

You can also create your own templates, giving looper ultimate flexibility to work with any compute infrastructure in any environment; just follow these examples and point your `pepenv` config file to your custom template using the `submission_template` attribute. When you develop a pipeline or configure a pepenv, you can use any variables you like; these are just the defaults. You can create your own variables by defining them in the `pipeline_interface` and then you can use them in your `submit_template` file to configure things for your local environment. [Read more about pipeline_interface.yaml here](http://looper.readthedocs.io/en/latest/pipeline-interface.html).

### Advanced template variables

What's actually happening behind the scene is that settings from your chosen compute and resource packages are combined, and then used to populate the template variables. You specify a pepenv "compute package" by what you pass on the command line to `--compute`. Then, the system automatically chooses a resources package (from `pipeline_interface`) based on file size of the input sample. The compute and resource packages are merged into a single object, which now has access to any variables defined at either level. Then, there's a third layer: anything specified in a `compute` section of `project_config` file is added last of all, which has then the highest precedence. This is used to populate any variables in your submission template. So, in theory, you can define these variables in any of these 3 places. The pipeline interface definition is loaded later, so it has precedence (that is, if you specify a variable in pipeline interface with the same name as one in the pepenv, the pipeline setting overrides the environment setting). 

