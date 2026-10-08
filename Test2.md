# Test 2
## Setup

1. hhs into sphinx and navigate to my directory
```
ssh sphinx
cd /lustre/isaac24/proj/UTK0505/yng832/
```
2. Make directory for all the test files and copy over data from the test files bank.
```
mkdir test2 #making test 2 file
cd test2 #entering test 2 file
mkdir analyses #making analyses file
mkdir 
mkdir raw_data #making raw_data file
cd raw_data #entering raw_data file
cp ../../../test2/reads/*gz ./ #copying raw_data files from the 
```
3. 
4. 



## Step 1: Quality Control (FastQC + MultiQC)
## Step 2: Step 2: Read Trimming and Filtering (fastp)
## Step 3: Step 3: Alignment (bwa-mem2 + samtools)
## Step 4: Variant Calling (bcftools mpileup + call)
## Step 5: Filtering (bcftools filter)
## Step 6: Evaluation Against the Truth Set
