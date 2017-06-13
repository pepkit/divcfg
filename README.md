# PEP environment files

To use PEP project objects (or `looper`) with a cluster resource manager (SGE, SLURM, etc.), you have set a few settings, like queue name. This repository helps you set this up easily.

## Setting up your environment

1. Clone this repository
2. Point an environment variable (PEPENV) to the config file (add this to your `.profile` or `.bashrc`).

	```
	export PEPENV=path/to/compute_config.yaml
	```

	In this repository are a few files that set up configuration at places where looper is in use. Just point LOOPERENV to the appropriate one of these if there's a match:
	 * `rivanna.yaml`: Supercomputer at University of Virginia
	 * `cemm.yaml`: Cluster at the Center for Molecular Medicine, Vienna
	 * `nih.yaml`: Biowulf2 cluster at the NIH
	 * `stanford.yaml`: [Sherlock](http://sherlock.stanford.edu/mediawiki/index.php/Current_policies) cluster at Stanford

	 And that's it, you're done! If the existing config files do not fit your environment, you will need to edit the config file to match your environment by following these instructions:

## Configuring a new environment

Look at the examples files in this repository (start with the default [compute_config.yaml](compute_config.yaml) and customize for your compute environment. Likely, the only thing you will need to change is the `partition` variable, which should reflect your submission queue or partition name used by your cluster resource manager. For example, if you make your environment config file say:

```
compute:
  default:
    submission_template: templates/slurm_template.sub
    submission_command: sbatch
    partition: longq
  develop:
    submission_template: templates/slurm_template.sub
    submission_command: sbatch
    partition: develop
  ```

Then your runs will all be submitted to SLURM using the partition called `longq` (the `default` setting). You can switch from the `long` partition to the `develop` partition __on the fly__ by specifying the `--compute` argument to `looper run` like so:

```
looper run --compute develop
```

## Understanding templates

**Templates**. This `pepenv` repository comes with some commonly used templates (in the [templates](/templates) folder):
	- SLURM: [slurm_template.sub](/templates/slurm_template.sub)
	- SGE: [sge_template.sub](/templates/sge_template.sub)
	- localhost (compute locally): [localhost_template.sub](/tempaltes/localhost_template.sub)]

You can also add your own. Just follow these examples and point your `pepenv` config file to your custom template using the `submission_template` attribute.

### Writing a new template

If none of the existing templates fit what you need, you can write your own! The variables specified in these template files (like `{LOGFILE}` or `{CORES}`) are replaced by looper when it creates a job script. It populates the variables with information from a few different sources:

- {JOBNAME} -- automatically produced by looper using the `sample_name` and the pipeline name.
- {LOGFILE} -- automatically produced by looper using the `sample_name` and the pipeline name.
- {MEM} -- pulled from the `resources` section of the `pipeline_interface` file.
- {CORES} -- pulled from the `resources` section of the `pipeline_interface` file.
- {TIME} -- pulled from the `resources` section of the `pipeline_interface` file.
- {PARTITION} -- pulled from the `compute` section of the `pepenv` file.

You can create your own variables by defining them in the `pipeline_interface` and then you can use them in your `submit_template` file to configure things for your local environment.
