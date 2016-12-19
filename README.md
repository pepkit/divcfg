# Looper environment files

To use Looper with a cluster resource manager (SGE, SLURM, etc.), you have to tell looper about a few settings, like queue name. Clone this repository and point an environment variable to the config file and edit it.

```
export LOOPERENV=path/to/looper_env.yaml
```

For example:

```
export LOOPERENV=${CODEBASE}looperenv/rivanna.yaml
```

Originally, we had to specify these things in each pipeline repository or for each project. Now you can just point looper to your global config file and it will use these settings instead.

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

Then you can switch from the `long` partition to the `develop` partition by specifying the `--compute` argument to `looper run` like so:

```
looper run --compute develop
```