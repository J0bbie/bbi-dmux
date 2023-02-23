# bbi-dmux

# Installation.

## Set-up conda env.

The pipeline will be performed within a conda environment which will handle most of the dependencies:
`micromamba create -f environment.yml`

We can now activate this conda environment with:
`micromamba activate sexomics_nf`

In addition, the following python packages need to be installed manually within this environment:
```
pip install --user pysam
pip install --user scrublet
pip install git+https://github.com/andrewhill157/barcodeutils#egg=barcodeutils
pypy3 -m pip install Bio --user
```

## Intro
This pipeline is the under-construction pipe for processing 2-level and 3-level data. It uses the pipeline management system Nextflow to operate.

The pipeline is run in two parts, the first is [bbi-dmux](https://github.com/bbi-lab/bbi-dmux) which runs the demultiplexing, and the second is [bbi-sci](https://github.com/bbi-lab/bbi-sci/) which completes the preprocessing. The instructions below apply to both pipelines, and both pipelines can use the same configuration file.


## Run pipeline.

Run the program:
`nextflow run -profile dkfz .`


#### Sample sheet:
The sample sheet should be a csv with 3 columns: RT Barcode, Sample ID, and Reference Genome. Here's an example:

```
RT Barcode,Sample ID,Reference Genome
2P09-A01,Sample1,Mouse
2P09-A02,Sample1,Mouse
2P09-A03,Sample2,Human
2P09-A04,Sample2,Human
```

#### Configuration file:

The second things you need are configuration files that pass in arguments to the pipeline. These are the *experiment.config* and *nextflow.config* files.

##### *experiment.config* file

 The *experiment.config* file is helpful as it allows you to specify if your data is 2-level or 3-level, allocate memory requirements, process only a subset of your samples and use custom genomes to map your data. We highly recommend using this instead of passing arguments on the command line so that you have a record of the run you called.

Notes:
- an example experiment configuration file is included in the package and includes further information on usage

- for the Shendure lab cluster use `process.queue = "shendure-long.q"` in either the *experiment.config* or *nextflow.config* files

##### *nextflow.config* file

The *nextflow.config* file defines processing values such as the required modules, memory, and number of CPUs for each processing stage, which do not change typically from run-to-run. The file can be left in the bbi-\* installation directory where Nextflow searches for it automatically when the pipeline starts up. The supplied *nextflow.config* file has two profiles: the default profile, called *standard*, defines modules used by the pipeline on CentOS 7 systems in the UW Genome Sciences cluster, and the *centos_6* profile, which defines modules used by the pipeline on CentOS 6 systems in the UW Genome Sciences cluster. In order to run the pipelines with the *centos_6* profile, add the command line parameter `-profile centos_6` to the nextflow run command, for example


```
nextflow run bbi-dmux -profile centos_6 -c experiment.config
```

This *nextflow.config* file has comments that give additional information.

#### Run the pipeline:

After making your configuration file, run the pipeline by first running:

```
nextflow run bbi-dmux -c experiment.config
```

After running the dmuxing step, check out the dmuxing dashboard by downloading the dmux_dash folder and opening the html in a web browser. When you're satisfied that dmuxing was successful, run:

```
nextflow run bbi-sci -c experiment.config
```


For either piece of the pipeline, if there is an error, you can continue the pipeline where it left off with either

```
nextflow run bbi-dmux -c experiment.config -resume
```
or

```
nextflow run bbi-sci -c experiment.config -resume
```

#### Recovery mode:

After running the dmux part of the pipeline, you will find all unassigned reads in fastq files labeled "Undetermined...". You can run bbi-dmux in 'recovery mode' to generate a table with information about why each read wasn't assigned, and a summary file with percentages. There will be a table and summary file for each lane.

Run recovery mode (AFTER running bbi-dmux as above) like this:

```
nextflow run bbi-dmux --run_recovery true -c experiment.config
```
You should provide the same experiment.config file that you used above.

#### The work folder:
Nextflow stores all of the intermediate files in its 'work' folder, which will be in the output directory you specified. This folder can get quite large, so after the pipeline is finished, you can delete it using:

```
rm -r work/
```

Warning: After you delete the work folder, -resume will no longer restart from the middle of the run, you'll have to start from the beginning if you need to regenerate any files. So please make sure that your pipeline has fully completed before deleting the work folder.

#### Troubleshooting:

##### I got a "UTF encoding" error for my samplesheet
If you get this error, it means that the python script for reading the samplesheet encountered an issue with the UTC encoding
of your samplesheet. We haven't pinpointed why this sometimes happens (probably something to do with your Excel settings when making 
the sheet), but fixing it is pretty straight forward. You have two options:
- Option 1 (from the terminal): Open the samplesheet in the terminal (using your preferred editor, e.g. vim or nano) and 
copy the contents by highlighting and using Cmd + C or Ctrl + C. Open a new file in your editor and paste the contents. Save and 
use this new file as your samplesheet.

- Option 2 (from your local computer): Open the samplesheet in Excel and use Save As to save the sheet using the file format 
"Comma Separated Values (.csv)". Use this new file as your samplesheet. 

#### Questions and errors:
If you run into problems, please leave a detailed description in the issue tab above!

### Acknowledgements
Many members of the Shendure and Trapnell labs as well as the BBI team have worked on portions of this pipeline or its predecessors, especially Andrew Hill and Jonathan Packer.
