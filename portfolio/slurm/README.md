
# How to make (almost) any script run in parallel using sbatch

As a regular user on a High Performance Computing Cluster
([MASSIVE](https://docs.massive.org.au/), to be specific), one thing I
struggled with is the incredibly convoluted
[documentation](https://slurm.schedmd.com/documentation.html) for the job
scheduling software, slurm. In addition, there is very few examples on the web
of how to use slurm [`sbatch`](https://slurm.schedmd.com/sbatch.html) jobs
effectively, limited typically to [cluster-specific
docs](https://hpc.nmsu.edu/discovery/slurm/tasks/parallel-execution/). There is
even less information on the more arcane use of
[`srun`](https://slurm.schedmd.com/srun.html).

This is probably why many `slurm` users (in my experience) stick to simple,
serial `sbatch` jobs, which is fine until you have to deal with a script that
executes many small actions that nevertheless take up a non-trivial amount of
time.

This really should be simple to do but slurm makes this overly complex. This
is what I've tried:

## sbatch each job-script call

```sh
find <dir> -name <glob> -exec sbatch <job-script>.sh {} +
```

This is fairly simple, but the downside is that the jobs are not grouped in any
way. This makes it difficult to `scancel` them, and makes it even harder to
setup [`sbatch` dependent jobs](https://hpc.nih.gov/docs/job_dependencies.html).

## sbatch arrays

```sh
sbatch --array=1-<n> <job-script>.sh
```

I won't go into much detail about this
[approach](https://blog.ronin.cloud/slurm-job-arrays/) but needless to say it
is overly complex. Your job script needs to be configured to match
`$SLURM_ARRAY_TASK_ID` to their respective inputs, so a config file is needed.
and the `job-script` needs to be written with fetching these configs in mind.
However, the benefit over the previous approach is that all the jobs are nested
in one sbatch call, making it easier to manage and set dependencies.

## Using srun in job-script

```sh
 #!/bin/bash

#SBATCH --job-name pytest
#SBATCH --output pytest.out
#SBATCH --ntasks=3
#SBATCH --cpus-per-task=2
#SBATCH --mem-per-cpu=500M
#SBATCH --partition=interactive
#SBATCH --time=0-00:8:00

# load modules
module load spack/2022a  gcc/12.1.0-2022a-gcc_8.5.0-ivitefn python/3.9.12-2022a-gcc_12.1.0-ys2veed

for i in {1..20}; do
    while [ "$(jobs -p | wc -l)" -ge "$SLURM_NTASKS" ]; do
        sleep 30
    done
    srun --ntasks=1 --cpus-per-task=$SLURM_CPUS_PER_TASK python pyScript.py "$i" &
done
wait
```

The above snippet was taken from the following
[guide](https://hpc.nmsu.edu/discovery/slurm/tasks/parallel-execution/#_advanced_srun_parallelism)
which explains the logic in more detail. A great solution and very close to what I need. But lets see if we can make this
generalisable to (almost) any script.

## srun wrapper script

```sh
#!/usr/bin/env bash
#SBATCH --job-name=SRUN_PARALLEL
#SBATCH --output=slurm-%j_SRUN_PARALLEL.out
#SBATCH --cpus-per-task=2
#SBATCH --mail-type=END,FAIL
#SBATCH --mail-user=vikesh.ajith@hudson.org.au
#SBATCH --mem-per-cpu=1G
#SBATCH --ntasks=5
#SBATCH --time=0-01:00:00

if [[ ${DEBUG-} =~ ^1|yes|true$ ]]; then
    set -o xtrace # Trace the execution of the script (debug)
fi

# Functions ----------------------------------------------------------------- #

script_usage() {
    cat <<EOF

Wrapper script that runs <job-script> in parallel.

Usage:

sbatch \\
    --ntasks=<n-parallel> \\
    --cpus-per-task=<n-cpus> \\
    --mem-per-cpu=<mem-per-cpu> \\
        $(basename "${BASH_SOURCE[0]}") \\
        "<job-script> [opt_fixed_args]" <input_1> [input_2 ...]

find <dir> -name <glob> \\
    -exec sbatch $(basename "${BASH_SOURCE[0]}") \\
    "<job-script> [opt_fixed_args]" {} +

inputs:
    job-script                  A script that takes zero or more optional fixed
                                args, followed by one input.

    opt_fixed_args              fixed args to <job-script>. Must be nested with
                                <job-script> in the same set of quotes to
                                distinguish it from the input arguments.

    input_*                     An array of inputs to be processed in parallel
                                by <job-script>

options:
    -h|--help                   Displays this help
    --ntasks=<n-parallel>       How many tasks to run in parallel [default 5]
    --cpus-per-task=<n-cpus>    How many cpus needed per task [default 2]
    --mem-per-cpu=<mem-per-cpu> How much ram per cpu per task [default 1G]
    --time=<D-HH:MM:SS>         Time limit for all tasks [default 0-01:00:00]

EOF
}

parse_params() {
    local min_args=2
    help_flag_set() { [[ "$#" -eq 0 || "$*" =~ ^--help$|^-h$ ]]; }
    less_than_min_args() { [[ "$#" -lt ${min_args} ]]; }
    not_sbatch_exec() { [[ -z ${SLURM_JOBID+x} ]]; }

    if help_flag_set "$@"; then
        script_usage && exit 0
    elif less_than_min_args "$@"; then
        printf \
            "[ERROR]: '%s' args required, '%s' provided: '%s'\n" \
            "${min_args}" "$#" "$@"
        script_usage && exit 1
    elif not_sbatch_exec; then
        printf "[ERROR] Script must be executed with sbatch\n"
        script_usage && exit 1
    fi

    readonly script_and_fixed_args="$1"
    shift
    input_array=("$@")
    readonly input_array
}

# Return `0` if there are equal or more running jobs than `$SLURM_NTASKS`
concurrent_jobs_maxed() { [[ "$(jobs -p | wc -l)" -ge "$SLURM_NTASKS" ]]; }

# `script_and_fixed_args` unquoted to allow arg expansion
input_loop() {
    for input in "${input_array[@]}"; do
        while concurrent_jobs_maxed; do sleep 30; done

        srun --ntasks=1 --nodes=1 --cpus-per-task="${SLURM_CPUS_PER_TASK}" \
            ${script_and_fixed_args} "${input}" &
    done
    wait # Necessary otherwise sbatch wrapper may end early
}

main() {
    parse_params "$@"
    input_loop
}

# Main ---------------------------------------------------------------------- #

main "$@"
```

### How it works

The script above is a bit lengthy (due to the usage instructions and parameter
parsing) but how it works is fairly simple.

```sh
sbatch \
    --ntasks=<n-parallel> \
    --cpus-per-task=<n-cpus> \
    --mem-per-cpu=<mem-per-cpu> \
    srun_parallel.sh
```

It is first wrapped in an `sbatch` call allowing the user to set any desired
`sbatch` args, such as time (`--time`) needed for all tasks to complete, how
many cpus are required per task (`--cpus-per-task`), and how much memory is
needed per cpu (`--mem-per-cpu`), and how many tasks to run in parallel
(`--ntasks`).

```sh
srun_parallel.sh "<job-script> [opt_fixed_args]" <input_1> [input_2 ...]
```

Next, its first argument is a single string containing `<job-script>` and any
optional fixed args (`opt_fixed_args`) to be provided to all `<job-script>`
tasks. These must be nested in the same quotes so the script can distinguish it
from the following inputs (`input_*`) downstream.

`inputs_*` can be supplied manually on the command line, or by using a helper
like GNU `find`:

```sh
find <dir> -name <glob> \\
    -exec sbatch srun_parallel.sh \\
    "<job-script> [opt_fixed_args]" {} +
```

Note that `{} +` is used here to supply all the outputs of find to a single
call. If `{} \;` is used, sbatch will be called `n` times, creating `n` jobs
(which is not what you want).

### Example Usage

I've used this script in many different pipelines where I needed to quickly
make a step run in parallel. For example:

#### Generating md5 hashes

```sh
gen_md5() {
    find "${work_dir}/results" \
        -regextype "egrep" \
        -regex '.+\.(vcf\.gz|tbi|cram|crai)$' \
        -exec sbatch \
        --qos "genomics" \
        --partition "genomics" \
        --time "0-00:30:00" \
        --job-name "GEN-MD5(${work_dir_name})" \
        --output "${slurm_log_dir}/slurm-%j_GEN-MD5.out" \
        srun_parallel.sh gen_md5.sh {} +
}
```

#### Parallel AnnotSV

- Note the use of `--parsable` here. By returning a single `SLURM_JOBID` I can
  make other `sbatch` tasks dependent on this one completing successfully.

```sh
run_annotsv() {
    find "${controlfreec_dir}" \
        -name "${controlfreec_bed_glob}" \
        -exec sbatch \
        --job-name="CNV_ANNOTSV_(${work_dir_name})" \
        --output=/dev/null \
        --ntasks=20 \
        --cpus-per-task=2 \
        --mem-per-cpu=2G \
        --time=0-00:30:00 \
        --parsable \
        srun_parallel.sh \
        "${pipeline_dir}/scripts/cnv_annotsv.sh ${work_dir}/datasets" {} +
}
```

### Caveats

Admittedly, while this script has served well for my needs, some parts could
use improvement, especially the jankiness of the quote expansion around the
script and fixed args. `<job-script>` must take the fixed arguments first, and
the inputs last, which is a reasonable expectation, though not all scripts may
take inputs in that order.

Some thought is also required with setting the `sbatch`
directives, if `--ntasks`, `--cpus-per-task=2` and `--mem-per-cpu` is too high,
your sbatch job may call an excessive amount of resources that might lead to a
high queue time (this is just a basic matter of multiplication).

If anyone has any suggestions of a better way to do this, I would definitely be
interested in alternative solutions.
