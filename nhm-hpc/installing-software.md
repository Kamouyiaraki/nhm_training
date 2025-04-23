# How to install software

You can install software in the following locations:

- Your home folder (using tools such as Conda).
- In a Singularity/Apptainer container image.
- The shared software area (by compiling the software).

Each method has pros and cons as described below.

| Location             | Pros                       | Cons                       |
|----------------------|----------------------------|----------------------------|
| Home folder          |<ul><li>Easy to do using tools like Conda</li><li>Only accessible by you, so anything you install won't affect other users</li></ul>|<ul><li>Space in your home folder is limited to 50GB|
| Singularity image    |<ul><li>An isolated environment where you can install all the dependencies you need</li><li>A consistent and reproducible software environment</li><li>Portable - you can transfer the image to your own computer or other HPC clusters</li><li>Shareable - other users can use the image as well (provided the image file is in a shared location, like a group workspace)|<ul><li>Learning curve to creating the image and running the container|
| Shared software area |<ul><li>No limits on space</li><li>Other users can also use the software|<ul><li>You must compile the software, so it's more complicated and takes longer</li><li>It can be difficult to do if the software requires a lot of dependencies|
                
## In your home folder

The easiest method of installing software is in your home folder. To do this you can use something like Conda, as described below.

### Installing conda


You can download Conda from [here](https://docs.conda.io/en/latest/miniconda.html). Follow the instructions below to install it.

On hpc-head-002, change into your home folder:

```
cd ~
```

Download Miniconda installer:

```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

Make the file executable:

```
chmod +x Miniconda3-latest-Linux-x86_64.sh
```

Install Miniconda:

```
./Miniconda3-latest-Linux-x86_64.sh
```

For example:
```
robtest1234@hpc-head-002:~$ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
--2025-04-22 08:25:48--  https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
Resolving repo.anaconda.com (repo.anaconda.com)... 104.16.32.241, 104.16.191.158, 2606:4700::6810:20f1, ...
Connecting to repo.anaconda.com (repo.anaconda.com)|104.16.32.241|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 154615621 (147M) [application/octet-stream]
Saving to: ‘Miniconda3-latest-Linux-x86_64.sh’

Miniconda3-latest-Linux-x86_64.sh 100%[==========================================================>] 147.45M  43.8MB/s    in 3.4s

2025-04-22 08:26:04 (42.9 MB/s) - ‘Miniconda3-latest-Linux-x86_64.sh’ saved [154615621/154615621]

robtest1234@hpc-head-002:~$ chmod +x Miniconda3-latest-Linux-x86_64.sh
robtest1234@hpc-head-002:~$ ./Miniconda3-latest-Linux-x86_64.sh

Welcome to Miniconda3 py312_25.1.1-2

In order to continue the installation process, please review the license
agreement.
Please, press ENTER to continue
>>>
```

Answer yes to any questions, accept the default options, and wait for the installation to complete. Once done, reload your terminal:

```
. ~/.bashrc
```
You should see `(base)` in front of your prompt. This shows that Conda has been installed. For example:
```
(base) robtest1234@hpc-head-002:~$
```

### Installing software with conda

You are now ready to use conda to install any software you like. As an example, we will demonstrate how to use conda to install an application called R.

This command will create a conda environment called `r-environment` and install two groups of packages called `r-essentials` and `r-base`:

```
conda create -n r-environment r-essentials r-base
```

For example:
```
(base) robtest1234@hpc-head-002:~$ conda create -n r-environment r-essentials r-base
Retrieving notices: done
Channels:
 - conda-forge
 - defaults
Platform: linux-64
Collecting package metadata (repodata.json): done
Solving environment: done
...
```



It may take a few minutes to install. Once done, activate the environment. You will see your prompt change from `(base)` to `(r-environment)`:

```
conda activate r-environment
```

For example:
```
(base) robtest1234@hpc-head-002:~$ conda activate r-environment
(r-environment) robtest1234@hpc-head-002:~$
```

You can use the `which` command to check that R is indeed installed in your home directory:

```
which R
```

For example:
```
(r-environment) robtest1234@hpc-head-002:~$ which R
/home/robtest1234/miniconda3/envs/r-environment/bin/R
```

Finally, launch R:

```
R
```

Example:
```
(r-environment) robtest1234@hpc-head-002:~$ R

R version 4.4.3 (2025-02-28) -- "Trophy Case"
Copyright (C) 2025 The R Foundation for Statistical Computing
Platform: x86_64-conda-linux-gnu

R is free software and comes with ABSOLUTELY NO WARRANTY.
You are welcome to redistribute it under certain conditions.
Type 'license()' or 'licence()' for distribution details.

R is a collaborative project with many contributors.
Type 'contributors()' for more information and
'citation()' on how to cite R or R packages in publications.

Type 'demo()' for some demos, 'help()' for on-line help, or
'help.start()' for an HTML browser interface to help.
Type 'q()' to quit R.

>
```

## Singularity/Apptainer

Singularity (sometimes called Apptainer) is a useful tool that allows you to install software inside an isolated environment known as a container. This allows you to install the software you need, without worrying about missing dependencies or conflicts with other software already installed on the cluster. 

You can learn more about it [here](https://apptainer.org/docs/user/latest). 

If you've worked with docker, then this concept will be familiar to you. Singularity is commonly used on HPC systems instead of Docker, as Singularity can be run without admin rights. However, they are very similar to each other, and you can even convert Docker images to Singularity.

### Creating a singularity image 

You can create a Singularity image on your own computer, provided you have Singularity installed. Alternatively, you can use the software-building server `hpc-sw-003`, which already has it installed. Email TS-ServiceDesk@nhm.ac.uk to request access to it. 

Using PuTTy or a terminal, SSH to `hpc-sw-003`:

```
ssh <username>@hpc-sw-003
```

Create your Singularity image file. You can name it anything, but it should end in `.def`:

```
touch <filename>.def
```

Open it with a text editor and construct your file. Here is a simple example:

```vim
Bootstrap: docker
From: ubuntu:24.04

%post
    apt update
    apt-get install bwa samtools -y

%help
    This is an Ubuntu container with bwa and samtools installed.
```

Here is an explanation of the different sections.

This section means use the `ubuntu:24.04` image from the Docker Hub container registry: 

```vim
Bootstrap: docker
From: ubuntu:24.04
```
> You can use other container registries and images, depending on your use case.

The `%post` section allows you to list commands to run when building the image. In this example we are installing the `bwa` and `samtools` applications inside the image:

```vim
%post
    apt update
    apt-get install bwa samtools -y
```

The `%help` section contains a helpful label that describes what the container does:

```vim
%help
    This is an Ubuntu container with bwa and samtools installed.
```

Now that you have your definition file, you can build the image. This will download the base image (Ubuntu in this example), install the software (bwa and samtools), and create an image file ending in `.sif`:

```sh
singularity build <filename>.sif <filename>.def
```

Example:
```sh
robtest1234@hpc-sw-003:~/bwa$ singularity build bwa.sif bwa.def
INFO:    User not listed in /etc/subuid, trying root-mapped namespace
INFO:    The %post section will be run under the fakeroot command
INFO:    Starting build...
Copying blob 2726e237d1a3 done   |
Copying config 602eb6fb31 done   |
Writing manifest to image destination
2025/04/22 11:10:26  info unpack layer: sha256:2726e237d1a374379e783053d93d0345c8a3bf3c57b5d35b099de1ad777486ee
INFO:    Running post scriptlet
+ apt update
Get:1 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Get:2 http://archive.ubuntu.com/ubuntu noble InRelease [256 kB]

# ...

Processing triggers for ca-certificates (20240203) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
INFO:    Adding help info
INFO:    Creating SIF file...
INFO:    Build complete: bwa.sif

```

Now that you have an image file, you can run your container using the following command:

```sh
singularity run <filename>.sif
```
This is useful to see what you've created, as you can explore the file system and see what's installed. You can see from the `Apptainer>` prompt that you are inside the container. For example:
```sh
robtest1234@hpc-sw-003:~/bwa$ singularity run bwa.sif
Apptainer>
```

Now that you've built your image you can upload it to the HPC cluster using `sftp` or `scp`. For example:
```sh
robtest1234@hpc-sw-003:~/bwa$ sftp hpc-head-002
-----------------------------------------------------------
---  This is the Natural History Museum in London, UK.  ---
---  If you are not an authorised user GO NO FURTHER !  ---
---  If you have problems connecting, contact :         ---
---          ts-servicedesk@nhm.ac.uk                   ---
-----------------------------------------------------------
robtest1234@hpc-head-002's password:
Connected to hpc-head-002.
sftp> put bwa.sif
Uploading bwa.sif to /gpfs/nhmfsa/bulk/share/data/mbl/share/workspaces/users/robtest1234/bwa.sif
bwa.sif                                                                                   100%   76MB  16.6MB/s   00:04
sftp> exit
```

See [Example 3](using-the-job-scheduler.md/#example-3---running-a-job-using-a-singularity-container) for guidance on how to write your Slurm script to run your Singularity container on the HPC cluster.

### Converting a Docker image to Singularity 

Instead of creating a Singularity image from scratch, you can convert an existing Docker image to Singularity. 

To do this, pull the image from the container registry (typically Docker Hub, but could be somewhere else):

```sh
singularity pull docker://<image>
```

Example: 
```sh
a-robef3@hpc-sw-003:~$ singularity pull docker://rockylinux/rockylinux
INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...
Copying blob 71cc2ddb2ecf done   |
Copying config 523ffac7fb done   |
Writing manifest to image destination
2025/04/22 11:47:25  info unpack layer: sha256:71cc2ddb2ecf0e2a974aec10b55487120f03759e86e08b50a7f4c5d77638ab9b
INFO:    Creating SIF file...
```

This will create an image file ending in `.sif`. Then you can upload it to `hcp-head-002` as described in the previous section.

## In the shared software area

If you install software in the shared software area, it means other users can also use it. There are two shared software locations, as described below.

| Location              | Description                                                                                                              |
|-----------------------|--------------------------------------------------------------------------------------------------------------------------|
| ``/software/testing`` | <ul><li>Anyone can install software here.<li>It is useful for testing and trying out new software installations.             |        
| ``/software/common``  | <ul><li>Only superusers can install software here.<li>It is intended for software that has been tested and is known to work. |

You can check which software has already been installed in the shared software area:

```
ml avail
```

For example:
```
robtest1234@hpc-head-002:~$ ml avail

------------------------------------------------- /software/common/lmod/lmod/modulefiles/Core --------------------------------------------------
   bcl2fastq/2.2.0             diamond/0.9.31  (D)    java/8.211              minimap2/2.16         python/3.7.bkp
   bedtools/2.29.0             emacs/26.2             jre/8.211               minimap2/2.17  (D)    python/3.7         (D)
   blast/2.4.0                 flash/latest           kallisto/0.44.0         ncbi_ngs/2.9.0        samtools/1.9
   blast/2.9.0          (D)    gffread/0.11.4         kallisto/0.45.0         ncbi_ngs/2.9.4 (D)    samtools/1.10      (D)
   bowtie/2.2.5                gmap/2019-09-12        kallisto/0.46.0  (D)    perl/5.28             settarg
   centrifuge/1.0.4beta        gnuplot/5.2.6          lmod                    plink/1.07            stringtie/2.0
   class2/2.1.7                hmmer/3.2.1            magicblast/1.4.0        prodigal/2.6.3        tophat/2.1.1
   diamond/0.9.24              htop/2.2.0             megan/6.15.2            python/2.7            transdecoder/5.5.0

------------------------------------------------------ /software/testing/modulefiles/Core ------------------------------------------------------
   BAMscorer/BAMscorer           bam2prof/bam2prof                ibs/ibs                                  pear/pear
   Eigensoft/Eigensoft           bam2vcf/bam2vcf                  idba/1.1.3                               platanus/2.02
   FastQC/fastqc                 bcftools/fastqc                  iqtree/1.6.12                            platanus/2.2.2          (D)
   GATK/gatk                     beagle/4.0                       kraken-biom/1.0.1                        plink/1.9               (D)
   JAGS/4.3.0.OLD                beagle5.1/beagle51               kraken2/2.0.8-b                          preseq/2.0
   JAGS/4.3.0             (D)    beast/2.6.3                      kraken2/2.1.2                     (D)    raxml/8.2.12
```

You can use the `ml spider` command for more detail:
```
robtest1234@hpc-head-002:~$ ml spider

--------------------------------------------------------------------------------------------------------------------------------------------
The following is a list of the modules currently available:
--------------------------------------------------------------------------------------------------------------------------------------------
  BAMscorer: BAMscorer/BAMscorer

  BSgenome: BSgenome/3.9
    Software infrastructure for efficient representation of full genomes and their SNPs.

  DESeq: DESeq/1.36.0
    DESeq library for differential expression analysis.

  Eigensoft: Eigensoft/Eigensoft

  FastQC: FastQC/fastqc

  GATK: GATK/gatk

  JAGS: JAGS/4.3.0.OLD, JAGS/4.3.0
```

To load a piece of software into your environment:
```
ml <software_name>/<version>
```

For example:
```
ml python/3.7
```

When you go to the folders `/software/testing` or `/software/common`, you may find more names, but if they do not appear when you run any of the commands above, they haven’t been properly installed. 

### Example of installing shared software

Next, a real example: installing AdapterRemoval. You can follow this as a guideline to install any other software. Please use this as a rough guide, as the precise steps can vary depending on the software.

Move to the `/software/testing` folder:
```
cd /software/testing
```

Create the folder where you will keep the software:
```
mkdir -p adapterremoval/2.3.4/{src,x86_64/bin}
```

Move to the folder where you will download the software:
```
cd adapterremoval/2.3.4/src
```

Download the software (using the `-O` option, you can rename the downloaded file):
```
wget -O adapterremoval-2.3.4.tar.gz https://github.com/MikkelSchubert/adapterremoval/archive/v2.3.4.tar.gz
```

Decompress the files:
```
tar xvzf adapterremoval-2.3.4.tar.gz
```

Move to the folder where the software can be compiled:
```
cd adapterremoval-2.3.4
```

Compile the software:
```
make
```

Now you have a locally executable programme in `/software/testing/adapterremoval/2.3.4/src/adapterremoval-2.3.4/build/AdapterRemoval`. Copy the executable file to the bin folder you created at the beginning:
```
cp build/AdapterRemoval /software/testing/adapterremoval/2.3.4/x86_64/bin
```

Now it is necessary to create a module file which will allow you to load the software anytime you use it from any location. Create a folder to store the module file (make sure the folder name is the same as the one you created in `/software/testing`):
```
mkdir -p /software/testing/modulefiles/Core/adapterremoval
```

Then create the `.lua` file, which contains all the module information:
```
touch /software/testing/modulefiles/Core/adapterremoval/2.3.4.lua
```

This is the information that should be copied into the `.lua` file (using a text editor like vim or nano):
```
whatis("AdapterRemoval v2.3.4 - rapid adapter trimming, identification, and read merging http://adapterremoval.readthedocs.io/")
local name = "adapterremoval"
local version = "2.3.4"
local base = pathJoin("/software", "testing", name, version, "x86_64")
prepend_path("PATH", pathJoin(base, "bin"))
```

You can now see your new software is available for use by typing `ml --ignore-cache avail adapterremoval` (you can also see there is an older version 2.3.1 available, which is fine &ndash; it means users can use either version):
```
robtest1234@hpc-head-002:~$ ml --ignore-cache avail adapterremoval

------------------------------------------------ /software/testing/modulefiles/Core -------------------------------------------------
   adapterremoval/2.3.1    adapterremoval/2.3.2    adapterremoval/2.3.4 (D)

  Where:
   D:  Default Module

Use "module spider" to find all possible modules.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".
```


Now you're ready to use your new software, so you load it into your environment:
```
ml adapterremoval/2.3.4
```

You can check that it's loaded by typing `which AdapterRemoval`:
```
robtest1234@hpc-head-002:~$ which AdapterRemoval
/software/testing/adapterremoval/2.3.4/x86_64/bin/AdapterRemoval
```

Finally, you can run the software:
```
robtest1234@hpc-head-002:~$ AdapterRemoval
AdapterRemoval ver. 2.3.4

This program searches for and removes remnant adapter sequences from
your read data.  The program can analyze both single end and paired end
data.  For detailed explanation of the parameters, please refer to the
man page.  For comments, suggestions and feedback please use
https://github.com/MikkelSchubert/adapterremoval/issues/new
```

If at some point you want to install a newer version of the software, you don’t need to delete the old one (actually it can be useful to have both). You can install the new version in the same folder, for example:
```
mkdir -p /software/testing/adapterremoval/3.0.0/{src,x86_64/bin}
```

To load the old version:
```
ml adapterremoval/2.3.4
```

To load the new version:
```
ml adapterremoval/3.0.0
```
