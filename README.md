# PEP environment files

To use PEP project objects (or `looper`) with a cluster resource manager (SGE, SLURM, etc.), you have set a few settings, like queue name. This repository helps you set this up easily.

## Setting up your environment

1. Clone this repository
2. Point an environment variable (PEPENV) to the appropriate config file in this respository (add this to your `.profile` or `.bashrc`).

	```
	export PEPENV=path/to/compute_config.yaml
	```

	In this repository are a few files that set up configuration at places where looper is in use. Just point PEPENV to the appropriate one of these if there's a match:
	 * `uva_rivanna.yaml`: [Rivanna cluster](http://arcs.virginia.edu/rivanna) at University of Virginia
	 * `cemm.yaml`: Cluster at the Center for Molecular Medicine, Vienna
	 * `nih_biowulf2.yaml`: [Biowulf2](https://hpc.nih.gov/docs/userguide.html) cluster at the NIH
	 * `stanford_sherlock.yaml`: [Sherlock](http://sherlock.stanford.edu/mediawiki/index.php/Current_policies) cluster at Stanford
	 * `compute_config.yaml`: Generic config file. Use this as a starting point to configure your own.

	 And that's it, you're done! If the existing config files do not fit your environment, you will need to edit the config file to match your environment by following these instructions:

## Configuring a new environment

Look at the examples files in this repository (start with the default [compute_config.yaml](compute_config.yaml)) and customize for your compute environment. Likely, the only thing you will need to change is the `partition` variable, which should reflect your submission queue or partition name used by your cluster resource manager.

## PEPENV configuration file and compute packages

```
compute:
  default:
    submission_template: pipelines/templates/local_template.sub
    submission_command: sh
  local:
    submission_template: pipelines/templates/local_template.sub
    submission_command: sh
  develop:
    submission_template: templates/slurm_template.sub
    submission_command: sbatch
    partition: develop
  big:
    submission_template: pipelines/templates/slurm_template.sub
    submission_command: sbatch
    partition: bigmem
  ```

The sub-sections below `compute` each define a *compute package* that can be activated. By default, the package named `default` will be used, which in this case is identical to the `local` package. You can make your default whatever you like. You can switch from local compute to the `develop` partition __on the fly__ by specifying the `--compute` argument to `looper run` like so:

```
looper run --compute develop
```

Generically, you just use `looper run --compute PACKAGE`, and PACKAGE could be `local` (which would do the same thing as the default, so doesn’t change anything), or `develop` or `big`, which would run the jobs on slurm, with queue `develop`, or `bigmem`. You can make as many compute packages as you wish.

## Understanding templates

Each compute package specifies a path to a file. These paths can be relative or absolute; relative paths are considered *relative to the pepenv file*. A template file uses variables (encoded like `{VARIABLE}`), which will be populated independently for each sample as defined in `pipeline_interface.yaml`. The one variable ``{CODE}`` is a reserved variable that refers to the actual command that will run the pipeline. Otherwise, you can use any variables you define in your `pipeline_interface.yaml`. You can also create your own templates, giving looper ultimate flexibility to work with any compute infrastructure in any environment.

This `pepenv` repository comes with some commonly used templates (in the [templates](/templates) folder):
* SLURM: [slurm_template.sub](/templates/slurm_template.sub)
* SGE: [sge_template.sub](/templates/sge_template.sub)
* localhost (compute locally): [localhost_template.sub](/tempaltes/localhost_template.sub)]

You can also add your own. Just follow these examples and point your `pepenv` config file to your custom template using the `submission_template` attribute.

### Writing a new template

If none of the existing templates fit what you need, you can write your own! The variables specified in these template files (like `{LOGFILE}` or `{CORES}`) are replaced by looper when it creates a job script. It populates the variables with information from a few different sources:

- {JOBNAME} -- automatically produced by looper using the `sample_name` and the pipeline name.
- {LOGFILE} -- automatically produced by looper using the `sample_name` and the pipeline name.
- {MEM} -- pulled from the `resources` section of the [`pipeline_interface`](http://looper.readthedocs.io/en/latest/connecting-pipelines.html) file.
- {CORES} -- pulled from the `resources` section of the [`pipeline_interface`](http://looper.readthedocs.io/en/latest/connecting-pipelines.html) file.
- {TIME} -- pulled from the `resources` section of the [`pipeline_interface`](http://looper.readthedocs.io/en/latest/connecting-pipelines.html) file.
- {PARTITION} -- pulled from the `compute` section of the `pepenv` file.

You can create your own variables by defining them in the `pipeline_interface` and then you can use them in your `submit_template` file to configure things for your local environment. [Read more about pipeline_interface.yaml here](http://looper.readthedocs.io/en/latest/connecting-pipelines.html).
