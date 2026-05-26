Tutorial on how to run nf-core/eager pipeline as a SLURM job
================
Flavia Leotta
2026-04-13
================
Last update: 26th May 2026

- [0. Location of the script](#0-location-of-the-script)
- [1. Usage](#1-usage)
  - [a. Supported flags](#a-supported-flags)
  - [b. Difference between short and long
    flags](#b-difference-between-short-and-long-flags)
- [2. Check your analysis status.](#2-check-your-analysis-status)
- [3. Check your analysis’ progress.](#3-check-your-analysis-progress)
- [4. Possible causes of errors.](#4-possible-causes-of-errors)
  - [a. Job killed by SLURM.](#a-job-killed-by-slurm)
  - [b. Problems with inputs](#b-problems-with-inputs)
  - [c. Singularity failing at creating a .sif
    image](#c-singularity-failing-at-creating-a-sif-image)

## 0. Location of the script

This tutorial’s goal is to help navigating the script file I have
prepared to run the nf-core/eager pipeline as a SLURM job. The script is
not supposed to be edited, but accepts some parameters which let the
user personalize the input, reference genome used, and more. At the
moment the pipeline executes the standard steps as provided by
nf-core/eager documentation, with the addition of Sex Determination and
Contamination Estimation: we expect to add more parameters with time and
allow the user to personalize the run even more. If you have any
specific request please feel free to either copy the script in your own
home directory and edit it to fit your needs, or contact me to add these
options to the common script (f.leotta@cent.uw.edu.pl).

The location of this script is the following:

    /mnt/workspace03/gr7001/share/scripts/slurm_eager.sh

As part of the same group in the server, you have execution permissions,
but not editing.

## 1. Usage

The script can be run as a SLURM job as usual, by specifying the
parameters using the appropriate flags:

    sbatch /mnt/workspace03/gr7001/share/scripts/slurm_eager.sh -i input [OPTIONS]

Only one parameter (the input) is strictly mandatory, while the rest
have default values.

### a. Supported flags

Options supported, in alphabetical order:

- `-b, --bwa_index` (optional): **Path to the directory** that contains
  a bwa index file, which has to be in accordance with the fasta
  reference file provided. Important: this must be simply the path to
  the directory, not the file itself. \[DEFAULT:
  /mnt/workspace03/gr7001/share/references/ (index built on Human
  Reference Genome hs37d5)\];
- `-i, --input` (mandatory): Input .tsv file with samples information,
  built following [nf-core/eager
  documentation](https://nf-co.re/eager/2.3.3/docs/usage/#tsv-input-method);
- `-n, --nextflow` (optional): Name of the installed Nextflow version
  (with DSL1). Important: installing the last version of Nextflow will
  not work, as it is incompatible with nf-core/eager pipeline. A
  tutorial on how to install the correct version is available in our
  [Community
  Github](https://github.com/LPCG-ancientDNA-Warsaw/nfcore-eager-LPCG/blob/main/Tutorial_new_server.md)
  \[DEFAULT: nextflow_dsl1\];
- `-o, --outdir` (optional): Directory where to store the results.
  \[DEFAULT: ./results (created if not present in the current
  directory)\];
- `-r, --reference` (optional): fasta file with reference genome.
  \[DEFAULT: /mnt/workspace03/gr7001/share/references/hs37d5.fa
  (Human)\];
- `-s, --singularity` (optional): Singularity version. \[DEFAULT:
  2.5.1\].

### b. Difference between short and long flags

Each flag can be used in two different ways: there’s the short version
(ex: -i) and the long version (ex: –input). They are both read as the
same flag, but their usage is slightly different.

When using the short version of the flag, the script expects a space
between the flag and the argument. Example:

    sbatch /mnt/workspace03/gr7001/share/scripts/slurm_eager.sh -i input.tsv

Instead, when using the long version of the flag, the script expects
only a “=” between them, with no space, and with the argument written
between quoatation marks (“). Example:

    sbatch /mnt/workspace03/gr7001/share/scripts/slurm_eager.sh --input="input.tsv"

This is important, as writing:

    sbatch /mnt/workspace03/gr7001/share/scripts/slurm_eager.sh --input input.tsv

will result in an error.

## 2. Check your analysis status.

To check the status of your analysis you can use two different methods.
The first one, prints a list of all the jobs currently on queue:

    squeue

You can locate yours by finding your username (USER); NAME is by default
set to “EagerAnalysis”. Another method, outputting only the analysis
associated to your username, is:

    squeue -u <user_name>

where <user_name> is the username provided when the server account was
created. If the STATUS of your job is set to R, then it is running,
while if it is set to PD it is on queue.

## 3. Check your analysis’ progress.

There are four ways to check your analysis progress. While in the
directory where the analysis was started, you can:

- check your output directory with `ls results/`. The pipeline
  nf-core/eager creates a folder for each new step started, which can
  provide a rough idea of the status of your analysis;
- check the “interface” of Nextflow (what you would’ve seen by running
  the analysis on an interactive screen) with
  `less -S slurm-[JOBID]_EagerAnalysis.out`;
- check the list of **completed** steps, with the time they took, with
  `results/pipeline-info/execution_trace_[date-time].txt`. The downside
  is that it only shows completed steps, for each sample, but it also
  provides a good time estimate for those steps that are repeated for
  each sample (ex: bwa, usually the rate-determining step);
- check the hidden nextflow log file with `less -S .nextflow.log`. This
  file is updated every real-time 5 minutes, and displays completed,
  running and scheduled steps at each time point.

## 4. Possible causes of errors.

### a. Job killed by SLURM.

By default, the PARTITION is set to “Short”, as this type of analysis
should not take more than a couple of hours: if you encounter occasional
problems with analysis killed before they are finished, please make a
local copy of this script and change, in the header,
`#SBATCH --partition=short` to `#SBATCH --partition=long`. If it is a
recurring problem, please contact me for the possibility of creating a
personalized script for you and your group.

### b. Problems with inputs

In the directory where you called the script, you will find the
following file: `slurm-[JOBID]_EagerAnalysis.out`. Here you will find
the output of the analysis while it runs: if the analysis was stopped
because of input problems, the reason will be printed here. Remember:
the script requires an input file, it will always stop if it is not
provided. The rest of the parameters can be skipped, but if they are
provided in a format that differs from the instructions provided in
section 1b, it will cause an error.

### c. Singularity failing at creating a .sif image

If another version of Singularity is provided, the pipeline will
struggle with downloading the `.sif` image associated with it: this is a
very big temporary file which I have noticed can block the pipeline
until SLURM kills it, which is why I have downloaded the file and made
it available in the `common_files` directory. Please always use the
default Singularity version, unless strictly necessary otherwise.
