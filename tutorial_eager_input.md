- [ **Generate nf-core/eager input files using LPCG custom scripts** ](#-generate-nf-coreeager-input-files-using-lpcg-custom-scripts-)
  - [Introduction](#introduction)
    - [ What are nf-core/eager inputs and ouputs? ](#-what-are-nf-coreeager-inputs-and-ouputs-)
  - [The four scripts](#the-four-scripts)
    - [ 1. create\_tsv\_run.py ](#-1-create_tsv_runpy-)
    - [ 2. create\_tsv\_indiv.py ](#-2-create_tsv_indivpy-)
    - [ Bonus 1: Pipying the first two scripts ](#-bonus-1-pipying-the-first-two-scripts-)
    - [ Bonus 2: Fixing the files paths ](#-bonus-2-fixing-the-files-paths-)
  - [Accompanying files](#accompanying-files)
    - [ Configuration: `conf/lpcg_lib_params.yaml` ](#-configuration-conflpcg_lib_paramsyaml-)

# <span style="color:#326273"> **Generate nf-core/eager input files using LPCG custom scripts** </span>

Flavia Leotta
Last updated: 25/06/2026

##  Introduction 

This tutorial refers to a set of four scripts available on the shared folder of the Laboratory of Paleogenetics and Conservation Genetics (LPCG), created with the intention of automatize and facilitate running the pipeline nf-core/eager (Fellows Yates J. A. _et al._, 2021) on sequencing results. Please refer to the pipeline extensive [documentation](https://nf-co.re/eager/2.5.1/) for more information about nf-core/egaer functionalities.

### <span style="color:#5C9EAD"> What are nf-core/eager inputs and ouputs? </span>

To summarise, nf-core/eager is a pipeline created specifically for aDNA analysis that takes either `.fastq` or `.bam` files as inputs and produces a set of different, and customizable, outputs including reports on endogenous DNA percentage, damage estimation, contamination calculation, sex determination and more!

##  The four scripts 

The scripts I have prepared are intendend to be run in the following order:

1. [create_tsv_run.py](#create_tsv_run.py)
2. [create_tsv_indiv.py](#create_tsv_indiv.py)
    1. Optional step: fix_paths.sh
3. launch_eager_per_run.sh
4. launch_eager_merged_individuals.sh

### <span style="color:#5C9EAD"> 1. create_tsv_run.py </span>

When running nf-core/eager on one library, it is possible to provide the path to the input files and a series of flags directly on the command line. One of the pipelines main features is, though, its ability to merge libraries at different levels (i.e. first at library level and then at sample[^sample] level) which is only possible if a `.tsv` input file is provided. An extensive description on what information is required in this `.tsv` input file can be found in the [documentation](https://nf-co.re/eager/2.5.1/docs/usage/#tsv-input-method) (which I'd advise to check for an explanation about the library merging process). This script parses a folder containing `.fastq` files and generates the `.tsv` input file with all or a subset of the samples in the folder: there are no mandatory flags, but if none is provided this script will make a few assumptions...

  Usage:
    
    create_tsv_run.py [additional flags]

  Output:
    
    eager_input_(ProjectName)_(FolderName).tsv

**Input/Output files**:
    
  - **\-c**, **\-\-config**: path to `.yaml` file with projects parameters. Default: [`/mnt/workspace03/gr7001/share/conf/lpcg_lib_params.yaml`](#configuration-conflpcg_lib_params.yaml).
  - **\-\-data**: `.csv` file that contains Lane information. Must contain a row that starts with "[Data]". If none is provided, the script looks for it in the same folder as **\-\-dir** and, next, in the parent folder of **\-\-dir**.
  - **\-\-dir**: directory where `.fastq` files are stored. Default: current working directory.
  - **\-p**, **\-\-project**: Project name(s). When providing more than one, separate them with a space. Matches projects in the [`/mnt/workspace03/gr7001/share/conf/lpcg_lib_params.yaml`](#configuration-conflpcg_lib_params.yaml) configuration file and/or Sample_Project column in the **\-\-data** `.csv` file. Default: "global".
  - **\-r**, **\-\-run**: run name. Used to: 1. correct the Library ID by adding the run name at the end (this ensures that a Library sequenced more than once produces two unique outputs), 2. if provided, be the FolderName in the output `.tsv` file name.

**Script functionality options**:
    
  - **\-\-dry\-run**: Displays what the script would do but don't actually create the `.tsv` file.
  - **\-\-ignore\-samplesheet**: Ignore the `SampleSheet.csv`[^samplesheet] in the folder and use default parameters.

**nf-core/eager input options**: these last flags are some of the information that an input `.tsv` file for nf-core/eager should include. This information is usually extracted from the configuration file [`/mnt/workspace03/gr7001/share/conf/lpcg_lib_params.yaml`](#configuration-conflpcg_lib_params.yaml), but those values can be overridden using these flags.
    
  - **\-\-chemistry**: Illumina sequencer colour chemistry number.
  - **\-\-seqtype**: paired end or single end data.
  - **\-\-species**: species scientific name.
  - **\-\-strand**: strandedness of the data.
  - **\-\-udg**: UDG treatment information.

The script will create a .tsv file with the following columns:

| Column Name | Mandatory? | Description | Default | How the script obtains this information |
| :---------- | :--------: | :---------------------------- | :------ | :---------------------------------- |
| Sample_Name | Yes | A text string containing the name of a given sample of which there can be multiple libraries. | _None_ | A REGEX string[^regex] from [`conf/lpcg_lib_params.yaml`](#configuration-conflpcg_lib_params.yaml) is applied to the sample `.fastq` file name |
| Library_ID | Yes | A text string containing the name of a given library | _None_ | A second REGEX string from [`conf/lpcg_lib_params.yaml`](#configuration-conflpcg_lib_params.yaml) is applied to the sample `.fastq` file name |
| Lane | Yes | A number indicating which lane the library was sequenced on | 1 | The information is extracted from the Lane column in the `SampleSheet.csv` file |
| Colour Chemistry | No, unless Poly-G trimming is enabled | The number of colour chemistry used by the Illumina sequencer (2 for Next/NovaSeq, 4 for Hi/MiSeq) | 2 | Extracted from [`conf/lpcg_lib_params.yaml`](#configuration-conflpcg_lib_params.yaml) |
| SeqType | Yes | A text string ("PE"/"SE") indicating paired end or single end data | "PE" | Extracted from [`conf/lpcg_lib_params.yaml`](#configuration-conflpcg_lib_params.yaml) |
| Organism | No | A text string of the organism scientific name | "NA" | Extracted from [`conf/lpcg_lib_params.yaml`](#configuration-conflpcg_lib_params.yaml) |
| Strandedness | Yes | A text string indicating "single" or "double" strandedness | "single" | Extracted from [`conf/lpcg_lib_params.yaml`](#configuration-conflpcg_lib_params.yaml) |
| UDG_treatment | Yes | A text string indicating "full", "half" or "none" UDG treatment | "half" | Extracted from the sample `.fastq` file name using a REGEX name in the case of project[^project] "human", extracted from [`conf/lpcg_lib_params.yaml`](#configuration-conflpcg_lib_params.yaml) in the others |
| R1 | Yes if the inputs are `.fastq` files | A text string of a file path pointing to the forward reads file | _None_ | Extracted from the path in which the `.fastq` files are |
| R2 | No, unless SeqType is "PE" | A text string of a file path pointing to the reverse reads file | _None_ | Takes the R1 path and substitutes "R1" with "R2". WARNING: this is not validated (as in, the presence of the file is not checked), and could generate errors |
| BAM | No, unless the inputs are `.bam` files | A text string of a file path pointing to the `.bam` file. This is **incompatible** with columns R1 and R2, and should be set to NA when those column are not empty | "NA" | Simply always set to "NA" | 

### <span style="color:#5C9EAD"> 2. create_tsv_indiv.py </span>

> Before this step it is **highly** recommended to check the previous output file `eager_input_(ProjectName)_(FolderName).tsv` for any errors: it will be harder to correct anything downstream.

As previously mentioned, the nf-core/eager pipeline is able to merge libraries on many levels and, because it follows the Nextflow syntax, it is able to parallelize processes and avoid rerunning analysis that have previously succeeded. Quite useful when libraries from the same Individual are sequenced in different moments: theoretically, a previously mapped library, shouldn't have to me re-mapped everytime. What we have noticed, though, is that if new samples are added to the input `.tsv` file, the pipeline is not able to recognise the previous samples as already cleaned and mapped, thus repeating, unnecessarily, these longer steps.

Since the sample merging happens after the mapping step (undoubtedly the most time consuming step), the solution we have found is to divide **every** library in its own folder, and then re-run eager for each Individual using the already mapped libraries' `.bam` files as the input. To do so, we have designed this folder structure:

```text
share/
└── eager_outputs/
    ├── global/
    └── {Project_Name}/
        └── {Sample_Name}/
            ├── {Library_ID}/
            ├── {Library_ID}.tsv
            ├── {Sample_Name}_merged/
            └── {Sample_Name}.tsv
```

The second script will take the previously created `eager_input_(ProjectName)_(FolderName).tsv` file and will:

- locate the correct ProjectName folder;
- create the folders and/or `{Library_ID}.tsv` files that do not exist already;
- update any existing `{Sample_Name}.tsv file`, by appending the new libraries information.

Only one flag is required.

  Usage:

    `create_tsv_indiv.py --input [additional flags]`

  Output:

    many folders and `.tsv` files as described earlier

**Input/Output files**:
    
  - **\-i**, **\-\-input**: path to the `eager_input_(ProjectName)_(FolderName).tsv` (or any other `.tsv` file with the same structure). This flag is required.
  - **\-o**, **\-\-outdir**: path to the output directory. If not provided, it will try to extract the ProjectName from the input `.tsv` file: if no ProjectName will be found, it will output the results in the `global/` folder. This folder is meant to be **temporary** and serve only to manually re-assign the correct location to each ouput. Do not keep there any folder or file for an indeterminate amount of time. If an **\-\-outdir** is provided but doesn't match the extracted ProjectName, it will throw a warning.
  - **\-r**, **\-\-run**: sequencing run ID used to substitute the Illumina suffix "_SXXX" with the run ID, ensuring that the same library sequenced twice will produce two distinct outputs. This flag is an artefact of when the previous script did not allow for such correction, but now it is not necessary if this was already done in the previous script.

**Script functionality options**:

  - **\-y**, **\-\-yes**: The script will print a preview and ask the user if all the information is correct. The user has to type "Y" or "YES" to proceed, but if this flag is provided it will not require the manual check.

The library-specific `{Library_ID}.tsv` files will be composed by only two rows: the header, and the row corresponding to that Library_ID in the `eager_input_(ProjectName)_(FolderName).tsv` file. Then, the same row is appended to `{Sample_Name}.tsv file`: the only differences are that the "R1" and "R2" columns will be set to "NA", the "SeqType" will be always converted to "SE" and the "BAM" column will have the `{Library_ID}.bam` file path. To not worry if the `{Library_ID}.bam` was still not created at this stage: it is expected, the path is generated simply because the output folder structure is known.

### <span style="color:#5C9EAD"> Bonus 1: Pipying the first two scripts </span>

The two scripts can be run one right after the other by launching this command from the directory where the `.fastq` files are stored:

`/mnt/workspace03/gr7001/share/scripts/tsv_input_parser.py -i $(/mnt/workspace03/gr7001/share/scripts/create_tsv_run.py)`

### <span style="color:#5C9EAD"> Bonus 2: Fixing the files paths </span>

## Accompanying files

I have prepared some additional files. With the assumption that the shared folder is available at path `/mnt/workspace03/gr7001/share/`:

1. Configuration file `conf/lpcg_lib_params.yaml`
2. Configuration file `conf/lpcg_warsaw.config`
3. Configuration file `conf/lpcg_human.config`
4. Common files available in the folders `singularity_images/`, `references/`, `genotyping/`.
5. SLURM script `slurm_eager_profiles.sh`

### <span style="color:#5C9EAD"> Configuration: `conf/lpcg_lib_params.yaml` </span>

This file contains information on how we name our samples and other parameters (i.e. UDG treatment) we use to create the nf-core/eager input `.tsv` file. There are global settings which include default values (known to be common within our samples) and "most likely" fallbacks (for example, most commonly used sample naming conventions), and there are settings specific for each project. The file is readable by everyone in the group, but editable only by me: if you wish to add your project-specific settings, please contact me in office or by e-mail.

[^project]: A project is the name of the research project to which each sample is associated to. It is one of the column names in the `SampleSheet.csv` file that accompanies each sequencing run.
[^regex]: A [REGEX](https://en.wikipedia.org/wiki/Regular_expression) string is a sequence of characters that defines a search pattern, used to match, search, or manipulate text based on specific criteria.
[^sample]: By sample, nf-core/eager is referring to an individual. For example, if we collected the bone powder from a femur and from a phalanx of individual 1, the individual 1 will be the Sample (the femur and phalanx will be different libraries of the same Sample).
[^samplesheet]: The `SampleSheet.csv` file is a text file with information about the sequencing run. It can be found in each sequencing folder in the `/mnt/workspace03/gr7001/share/fastqs/` folder.
