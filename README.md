DolphinNext Developer Tutorial
========

Introduction
-------

DolphinNext is an easy-to-use platform for creating, deploying, and executing complex nextflow pipelines for high throughput data processing.

DolphinNext provides: 
--------

1. A drag and drop user interface to build nextflow pipelines
2. Reproducible pipelines with version tracking 
3. Seamless portability to different computing environments with containerization
4. Simplified pipeline sharing using GitHub (github.com)
5. Support for continuous integration and tests (travis-ci.org)
6. Easy re-execution of pipelines by copying previous runs settings
7. Integrated data analysis and reporting interface with R markdown support

The aim is;
--------

1. Reusability
2. Reproducibility
3. Shareability
4. Easy execution
5. Easy monitoring
6. Easy reporting


Expected learning outcome
========

To understand the basics of DolphinNext, how to use pipeline builder for different objectives and to familiarize yourself with Nextflow and some standard software packages for such analysis.

Overview
========
  * [Before you start](#before-you-start)
  * [Getting Started](#getting-started)
  * [Exercise 1: Creating processes](#exercise-1---creating-processes)
    * [FastQC process](#1-fastqc-process)
    * [Hisat2 process](#2-hisat2-process)
    * [RSeQC process](#3-rseqc-process)
  * [Exercise 2: Building a pipeline](#exercise-2---building-a-pipeline)
  * [Exercise 3: Running a pipeline](#exercise-3---executing-a-pipeline)


If you prefer, you can click on the video links to follow the tutorial in a video.

[![Watch the video](dolphinnext_images/developer_guide1_youtube.png)](https://youtu.be/zfLXCgJFls4)

**Video Summary:**
  * Exercise 1: Creating processes
    - FastQC process [at 0:07](https://youtu.be/zfLXCgJFls4?t=7)
    - Hisat2 process [at 0:55](https://youtu.be/zfLXCgJFls4?t=55)
    - RSeQC process [at 2:02](https://youtu.be/zfLXCgJFls4?t=122)
  * Exercise 2: Building a pipeline [at 2:43](https://youtu.be/zfLXCgJFls4?t=163)
  * Exercise 3: Running a pipeline [at 4:34](https://youtu.be/zfLXCgJFls4?t=274)
  

# Before you start

## DolphinNext docker version

DolphinNext can be run standalone using a docker container.
First docker image need to be build unless you want to use prebuild from dockerhub. So, any changes in the Dockerfile require to build the image. But in this tutorial, we will pull it from dockerhub and start the container.

## Pull the docker image


1. Pull DolphinNext-studio
```   
docker pull ummsbiocore/dolphinnext-studio
```

## Start the container


1. We move database outside of the container to be able to keep the changes in the database every time you start the container.
Please choose a directory in your machine to mount. For example, I will use ~/export directory for this purpose.
```
mkdir ~/export
```
2. While running the container;
```
docker run --privileged -m 10G -p 8080:80 -v ~/export:/export -ti ummsbiocore/dolphinnext-studio /bin/bash
```
3. After you start the container, you need to start the mysql and apache server using the command below;
```
startup
```
4. Now, you can open your browser to access DolphinNext using the url below.

http://localhost:8080/dolphinnext

# Tutorial guide

This guide will walk you through how to start creating Nextflow processes using DolphinNext and creating new Nextflow pipelines using DolphinNext pipeline builder and executing them.

## Getting Started

First, you need to access DolphinNext web page: https://localhost:8080/dolphinnext and click **Sign Up** or **Sign in with Google** buttons. You will be asked to enter some information about your institution, username, etc. 


<img src="dolphinnext_images/sign_in.png" width="80%">

Once you login, you will be the administrator of this mirror. You can add more users to your system and manage them from profile/admin section.


## Exercise 1 - Creating processes

Once logged in, click on the pipeline tab in the top left of the screen to access pipeline builder page. 

<img src="dolphinnext_images/build1-builderpage.png" width="80%">

### What is a "process"?

Process is a basic programming element in Nextflow to run user scripts. Please [click here](https://www.nextflow.io/docs/latest/process.html) to learn more about Nextflow's processes.

A process usually has inputs, outputs and script section. In this tutorial, you will see sections that include necesseary information to define a process shown in the left side of the picture below. Please, use that information to be fill "Add new process" form shown in the middle section the picture below. DolphinNext will then convert this information to a nextflow process shown in the left side of the picture. Once a process created, it can be used in the pipeline builder. The example how it looks is shown in the bottom left side in the picture. The mapping between the sections shown in colored rectangles.      

<img src="dolphinnext_images/build-processmapping.png" width="100%">

### The process we will create in this exercise;

1. FastQC process
2. Hisat2 process
3. RSeQC process

You’ll notice several buttons at the left menu. New processes are created by clicking green “New process” button.

### 1. FastQC process
**a.** First, please click, green “New process” button to open "Add New Process" modal.

**b.** Please enter FastQC for the process name and define a new "Menu Group". 

<img src="dolphinnext_images/build2-fastqc-addmenugroup.png" width="80%">

**c.** In the FastQC process, we have an input, an output and a line of a command we are going to use to execute the fastqc process. Please click "Add New Process" button and use the information below to fill the "Add New Process" form.

```
Name: "FastQC"
Menu Group: "Tutorial"
Inputs: 
  reads(fastq,set) name: val(name),file(reads)
  
Outputs: 
  outputFileHTML(html,file) name: "*.html"
  
Script:
  fastqc ${reads}
```  

**d.** Lets select input and output parameters (`reads` and `outputFileHTML`) and define their "Input Names" that we are going to use in the script section.

<img src="dolphinnext_images/build5-fastqc-parameters.png" width="100%">
 
**e.** Let's enter the script section

<img src="dolphinnext_images/build6-fastqc-script.png" width="80%">

**f.** Press "Save changes" button at the bottom of the modal to create the process. Now this process is ready to use. We will use it in the Exercise 2.

### 2. Hisat2 process

Let's create Hisat2 process. 

**a.** First, please click, green “New process” button to open "Add New Process" modal.

**b.** Inputs, outputs and scripts should be defined like below;

```
Name: "Hisat2"
Menu Group: "Tutorial"
Inputs: 
  reads(fastq,set) name: val(name),file(reads)
  hisat2IndexPrefix(val) name: hisat2Index
      
Outputs: 
  mapped_reads(bam,set) name: val(name), file("${name}.bam")
  outputFileTxt(txt,file) name: "${name}.align_summary.txt"

Script:
  hisat2 -x ${hisat2Index} -U ${reads} -S ${name}.sam &> ${name}.align_summary.txt
  samtools view -bS ${name}.sam > ${name}.bam

```

**c.** After you select input output parameters (`hisat2IndexPrefix`, `mapped_reads` and `outputFileTxt` parameters), add their names and enter the script. The page should look like this;

<img src="dolphinnext_images/build11-alignment-summary.png" width="80%">

**d.** Please save changes before you close the screen.

### 3. RSeQC process

**a.** First, please click, green “New process” button to open "Add New Process" modal.

**b.** The form should be filled using the information below;

```
Name: "RSeQC"
Menu Group: "Tutorial"
Inputs:
  mapped_reads(bam,set) name: val(name), file(bam)
  bedFile(bed,file) name: bed
  
Outputs: 
  outputFileTxt(txt,file) name: "RSeQC.${name}.txt"
  
Script:
  read_distribution.py  -i ${bam} -r ${bed}> RSeQC.${name}.txt
```

**c.** After you select input output parameters, enter their names and the script. The page should look like this;

<img src="dolphinnext_images/build13-RSeQC-process.png" width="80%">

**d.** Please, save changes before you close the screen.

Here Exercise 1 is finished. Please move to Exercise 2 to build the pipeline using the processes you created in Exercise 1. 

## Exercise 2 - Building a pipeline


Once logged in, click to the pipeline button in the top left of the screen. You’ll notice Enter Pipeline Name box, just below the Pipelines button.

Note*: If you could not finish the Exercise 1. Please "import" the [RNA-Seq-Tutorial.dn](https://raw.githubusercontent.com/UMMS-Biocore/dolphinnext-tutorial/master/RNA-Seq-Tutorial.dn) file in github using your pipeline builder and the processes defined in Exercise 1 will be in your left menu that you can use them while doing Exercise 2.

Before you start building the pipeline make sure you have the processes available in your menu.

<img src="dolphinnext_images/build14-menu.png" width="50%">

**a.** Please enter a name to your pipeline. E.g. "RNA-Seq-Tutorial" and select your menu group "public pipelines" and press save button.

**b.** Please drag and drop FastQC, Hisat2 and RSeQC to your workspace;

<img src="dolphinnext_images/build15-drag-drop.png" width="80%">

**c.** Please drag and drop three Input parameters and change their names to "Input_Reads", "Hisat2_Index" and "bedFile" and connect them to their processes;     

**d.** Connect your Hisat2 process with RSeQC process using mapped_reads parameter in both. You will observe that, when the types match you can connect the two processes using their matching input and output parameters.

**e.** Drag & Drop three "output parameters" from the side bar and name them "FastQC_output", "Hisat2_Summary", and "RSeQC_output" and connect them to their corresponding processes. While naming, click their "Publish to Web Directory" and choose the right output format according to the output type of the process.
 
<img src="dolphinnext_images/build16-name1.png" width="50%">

<img src="dolphinnext_images/build17-name2.png" width="50%">

<img src="dolphinnext_images/build18-name3.png" width="50%">

**f.** Overall pipeline should look like below;

<img src="dolphinnext_images/build19-pipeline.png">
 
## Exercise 3 - Executing a pipeline

  **1.** Once a pipeline is created, you will notice “Run” button at the right top of the page.

<img src="dolphinnext_images/project_runbutton.png" width="20%">

  **2.** This button opens a new window where you can create a new project by clicking “Create a Project” button. After entering and saving the name of the project, it will be added to your project list. 

<img src="dolphinnext_images/execute1_create_project.png" width="80%">

  **3.** Now you can select your project by clicking on the project.
You will then proceed by entering run name which will be added to your run list of the project. Clicking “Save run” will redirect you to the “run page” where you can initiate your run.

<img src="dolphinnext_images/execute2_enter_runname.png" width="80%">

  **4.** Here, please enter your working directory, choose your "Run Environment", click "Use Singularity Image" and enter the values below;
<pre>
Work Directory: /export/tests/test1
Run Environment: Local
Image Path: dolphinnext/rnaseq:1.0
Run Options: --bind /export --bind /data

Inputs:
  - bedFile: /data/genome_data/mousetest/mm10/refseq_170804/genes/genes.bed (Use <b>Manually</b> tab)
  - Hisat2_Index: /data/genome_data/mousetest/mm10/refseq_170804/Hisat2Index/genome (Use <b>Manually</b> tab)
  - Input_Reads: First go to <b>Files</b> Tab in "<b>Select/Add Input File</b>" modal and click "<b>Add File</b>" button. 
Then enter "File Directory (Full Path)" as: <b>/data/fastq_data/single</b> and follow <a href="#creating-collection">Creating Collection</a> section.
</pre>  

<img src="dolphinnext_images/execute3_enter_workdir.png" width="80%">

  **5.** Now we are ready to enter inputs we defined for the pipeline.
Please choose "Manually" tab.
First enter the location of the bed file. 
**bedFile:**
```
/data/genome_data/mousetest/mm10/refseq_170804/genes/genes.bed
```
<img src="dolphinnext_images/execute4_enter_bedfile.png" width="100%">

  **6.** Second enter the prefix for the hisat2 index files. Please choose "Manually" tab. 
**Hisat2_Index:** 
```
/data/genome_data/mousetest/mm10/refseq_170804/Hisat2Index/genome
```

<img src="dolphinnext_images/execute5_enter_index.png" width="100%">

  #### Creating Collection
  **7.**  Now, we are ready to add files;
First go to Files Tab in "Select/Add Input File" modal and click "Add File" button

<img src="dolphinnext_images/execute6_add_files.png" width="100%">

  **8.**  Enter the full path of the location of your files. Please choose "Files tab" and click "Add File" button. Here for the test case we will use the path below;
File Directory (Full Path): 
```
/data/fastq_data/single
```
And choose "Single List" for the "Collection Type" and press add all files button.

<img src="dolphinnext_images/execute7_enter_fullpath.png" width="100%">

  **9.** Here there is an option to change the names but we will keep them as they are and enter a collection name and "save files".
```
collection name: test collection
```
<img src="dolphinnext_images/execute8_enter_collectionname.png" width="100%">

  **10.** In the next screen, the user can still add or remove some samples. Let's click "Save file" button to process all samples.

<img src="dolphinnext_images/execute_9_choosefiles.png" width="100%">

  **11.** After we fill the inputs it should look like below and the orange "Waiting" button at the top right should turn to green "Ready to Run" button;

<img src="dolphinnext_images/execute10_filled_inputs.png" width="100%">

  **12.** Press that "Ready to Run" button.

  **13.** All run should finish in a couple of minutes. When the run finalized the log section will be look like below;

  **a. Logs**:

<img src="dolphinnext_images/execute11_logtab.png" width="100%">

  **b. Timeline**:
<img src="dolphinnext_images/execute12_timeline.png" width="100%">

  **c. Nextflow Reports**:
<img src="dolphinnext_images/execute13_nextflowreports.png" width="100%">

  **14.** In the report section, you can monitor all defined reports in the pipeline;
<img src="dolphinnext_images/execute14_Reports.png" width="100%">

  **a. FastQC**
<img src="dolphinnext_images/execute15_FastQC.png" width="100%">

  **b. Hisat2**
<img src="dolphinnext_images/execute16_Hisat2.png" width="100%">

  **c. RSeQC**
<img src="dolphinnext_images/execute17_RSeQC.png" width="100%">

  **15.** As you can tell from Timeline report, it used only a cpu and it didn't parallelize the run. To be able to start parallel runs, the profile for the environment should be changed.

<img src="dolphinnext_images/execute18_profile_fix.png" width="100%">

With this change there will be 3 parallel jobs.

<img src="dolphinnext_images/execute19_parallel_run.png" width="100%">

