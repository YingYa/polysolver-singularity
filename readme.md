# Singularity image of Polysolver

## Building the Singularity image of Polysolver

To build the [Singularity](https://www.sylabs.io/singularity) image of [Polysolver](https://hub.docker.com/r/sachet/polysolver), run:

```bash
sudo singularity build polysolver_v4_singularity.sif build_spec.txt
```

The `build_spec.txt` file instructs Singularity to build the Polysolver version 4 image coming from the [Docker Hub](https://hub.docker.com) repository of its [author](https://hub.docker.com/u/sachet).

It also instructs Singularity to perform some changes on the image in order to make the HLA typing functionality of Polysolver working.

To do so, the Polysolver script `shell_call_hla_type` located (in the image) at `/home/polysolver/scripts/` needs some modifications, as described below.

### The `build_spec.txt` file

First of all, creates (in the image) a directory where users can bind-mount (see the `-B` option of `singularity exec`) their data, and also output their results (see the `outDir` argument of `shell_call_hla_type`):

```bash
mkdir /data
```

The script `shell_call_hla_type` is a bash script, so the shebang should point to bash, not to sh:

```bash
sed -i 's.#!/bin/sh.#!/bin/bash.g' /home/polysolver/scripts/shell_call_hla_type
```

In the image, the temporary directory should be `/tmp`:

```bash
sed -i 's.TMP_DIR=\$outDir.TMP_DIR=/tmp.g' /home/polysolver/scripts/shell_call_hla_type
sed -i 's.TMP_DIR=/home/polysolver.TMP_DIR=/tmp.g' /home/polysolver/scripts/shell_call_hla_type
```

The variable `SAMTOOLS_DIR` is not defined and is actually `/home/polysolver/binaries`:

```bash
sed -i 's.\$SAMTOOLS_DIR./home/polysolver/binaries.g' /home/polysolver/scripts/shell_call_hla_type
```

In hg38, the chromosome 6 is named `chr6`, not `6`:

```bash
sed -i 's.6:29941260-29945884.chr6:29941260-29945884.g' /home/polysolver/scripts/shell_call_hla_type
sed -i 's.6:31353872-31357187.chr6:31353872-31357187.g' /home/polysolver/scripts/shell_call_hla_type
sed -i 's.6:31268749-31272105.chr6:31268749-31272105.g' /home/polysolver/scripts/shell_call_hla_type
```

## Running the HLA typing functionality of Polysolver

Once the image `polysolver_v4_singularity.sif` built using the `build_spec.txt` file as described above, one can execute the `shell_call_hla_type` script as follows:

```bash
singularity exec -C \
-B /path/to/your/data/folder:/data \
-B /path/to/your/temporary/folder:/tmp \
polysolver_v4_singularity.sif /home/polysolver/scripts/shell_call_hla_type \
/data/yourBamFile.bam race includeFreq build format insertCalc /data
```

The `-C` option of `singularity exec` can prevent unwanted interferences between your system and the one within the image (if any) by running in an "isolated" mode.

The `-B` option of `singularity exec` mounts a folder of your system somewhere in the image system, making its content available for the image.

In the above example, `yourBamFile.bam` is in the `/path/to/your/data/folder/` directory of your system, and is mounted at `/data` in the image.

Consequently, in the image (and therefore for Polysolver), `yourBamFile.bam` is in the `/data/` folder.

The same applies for the temporary directory: the image `polysolver_v4_singularity.sif` will use a folder of your system (`/path/to/your/temporary/folder` in the above example) as temporary directory.

For the remaining arguments passed to `/home/polysolver/scripts/shell_call_hla_type` see below, or run it without arguments:

```
Usage: /home/polysolver/scripts/shell_call_hla_type bam race includeFreq build format insertCalc outDir
    -bam: path to the BAM file to be used for HLA typing
    -race: ethnicity of the individual (Caucasian, Black, Asian or Unknown)
    -includeFreq: flag indicating whether population-level allele frequencies should be used as priors (0 or 1)
    -build: reference genome used in the BAM file (hg18, hg19 or hg38)
    -format: fastq format (STDFQ, ILMFQ, ILM1.8 or SLXFQ; see Novoalign documentation)
    -insertCalc: flag indicating whether empirical insert size distribution should be used in the model (0 or 1)
    -outDir: output directory
```

## Cautions

The `build_spec.txt` file only allows to build a Singularity image of Polysolver version 4 coming from the Docker Hub repository of its author: no warranties are provided concerning the correctness of Polysolver itself.

Except the HLA typing functionality of Polysolver, its other functionalities are not yet addressed, if any.

Singularity version 3.1.1 was used.
