# Looper environment files

To use Looper with a cluster resource manager (SGE, SLURM, etc.), you have to tell looper about a few settings, like queue name. This repository helps you set this up easily.

1. Clone this repository
2. Point an environment variable (LOOPERENV) to the config file (add this to your `.profile` or `.bashrc`).

	```
	export LOOPERENV=path/to/looper_env.yaml
	```

	For example:

	```
	export LOOPERENV=${CODEBASE}looperenv/rivanna.yaml
	```

3. Edit the config file to match your environment (mostly, this means changing the `partition` variable to match the queue name you want to submit to).

For example, if you make your environment config file say:

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

## Templates

`looperenv` comes with some commonly used templates (in the [templates](/templates) folder):
	- [slurm_template.sub](/templates/slurm_template.sub)
	- [sge_template.sub](/templates/sge_template.sub)
	- [localhost_template.sub](/tempaltes/localhost_template.sub)]

You can also add your own. Just follow these examples and point your `looperenv` config file to your custom template using the `submission_template` attribute.


#### Template variables

The variables specified in these template files (like `{LOGFILE}` or `{CORES}`) are replaced by looper when it creates a job script. It populates the variables with information from a few different sources:

- {JOBNAME} -- automatically produced by looper using the `sample_name` and the pipeline name.
- {LOGFILE} -- automatically produced by looper using the `sample_name` and the pipeline name.
- {MEM} -- pulled from the `resources` section of the `pipeline_interface` file.
- {CORES} -- pulled from the `resources` section of the `pipeline_interface` file.
- {TIME} -- pulled from the `resources` section of the `pipeline_interface` file.
- {PARTITION} -- pulled from the `compute` section of the `looperenv` file.

You can create your own variables by defining them in the `pipeline_interface` and then you can use them in your `submit_template` file to configure things for your local environment.
