Tutorial for the new server
================
Flavia Leotta
2026-03-04

# Tutorial on how to download the pipeline nf-core/eager and transfer your files in the new server

### Requisites: have an account on the new server \[anthill\]\[ver0.4\]

You should have received an e-mail on your institutional e-mail, after
you supervisor has requested an account for you. In this e-mail you will
find the following information: login: your_account_name pass:
one_time_link_for_your_password gid: group_name (important later)

Log-in in the new server by running the following command and
substituting ‘login’ with your user name:

    ssh login@sih-17.cent.uw.edu.pl

There proceed to change your password with command `passwd`.

## 1. Download and install Nextflow

Do not install Nextflow through conda: the updated version of Nextflow
is not compatible with nf-core/eager pipeline. First, start an
interactive session and change directory to the tmp directory:

    srun -c4 --mem=10G --pty bash -l
    cd /tmp

And install Nextflow with these commands:

    curl -L https://github.com/nextflow-io/nextflow/releases/download/v22.10.6/nextflow -o "nextflow_dsl1"
    chmod +x nextflow_dsl1
    mkdir -p $HOME/.local/bin/
    mv nextflow_dsl1 $HOME/.local/bin/

Add nextflow_ds1 location to your shell by changing the working
directory back to your home folder, and running command:

    echo 'export PATH="PATH:$HOME/.local/bin"' >> ~/.bashrc

## 2. Confirm Nextflow installation

This version of Nextflow can only work with Java up to version 18. Even
though Java 21 is installed on the server (you can check its version
with `java -version`), a module for Java 18 is available. You can check
module availability with command `module avail java`, which returns a
list: java/openjdk-18 should be present. Load the module with this
command:

    module load java/openjdk-18

Remember you have to run this command every time you start a new
session. Now you can confirm Nextflow’s correct installation by running:

    nextflow_dsl1 info

The correct version is 22.10.6 build 5843. Note: from now on, remember
that when you want to use this version of nextflow, you have to use
command `nextflow_dsl1`, simply typing `nextflow` will not work.

## 3. Install nf-core/eager pipeline and test it

To install nf-core/eager pipeline, we clone it from github:

    git clone https://github.com/nf-core/eager/

This pipeline requires Singularity (or a similar containerization
software), which is already available on the server under its new name,
Apptainer. Even if the name has changed, all commands calling it by its
old name still work.

If you test the pipeline now, it theoretically works, but the process
gets killed because one of the temporary files (nfcore-eager-2.5.1.sif)
is too big. To solve it, it is enough to download this file locally: as
a group, we don’t need many copies of the same file, so at the moment it
is available in my folder for anyone to use. The directory is:
`fleotta/singularity_images/nfcore-eager-2.5.1.sif`. If the directory
changes, I will update this tutorial with the correct one.

Nevertheless, if you still wish to download this file, it requires to
start a new session that uses more CPUs and memory, and run the
following commands:

    srun -c10 –mem=20G –pty bash -l

    mkdir -p $HOME/tmp_singularity
    export SINGULARITY_TMPDIR=$HOME/tmp_singularity

    mkdir $HOME/singularity_images

    cd $HOME/singularity_images
    singularity pull --tmpdir $SINGULARITY_TMPDIR nfcore-eager-2.5.1.sif docker://nfcore/eager:2.5.1

Now, you can return to your home folder, create a temporary folder
(<test_folder>) where to store the outputs and test the pipeline with
the following command:

    nextflow_dsl1 run nf-core/eager -profile test,singularity -r 2.5.1 --outdir <test_folder> -with-singularity $HOME/singularity_images/nfcore-eager-2.5.1.sif

Note: This pipeline doesn’t work on sessions with less than 2 CPUs.

## 4. Transfer your files from the old server

If everything worked fine, you can test the pipeline on your own data.
To transfer your data from your folder in the old server (IP:
pier23.cent1.uw.edu.pl), log on IP pier23.cent1.uw.edu.pl and run this
command:

    scp -r ./<testfolder> <your_username_in_new_anthill_server>@sih-17.cent.uw.edu.pl:/home/users/<target_directory>

The `-r` flag is necessary to transfer directories and their files:
remove it if you wish to simply transfer files.
