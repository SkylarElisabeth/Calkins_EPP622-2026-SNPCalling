Your Name: - Skylar Calkins

Server: -

Repository URL: - https://github.com/SkylarElisabeth/Calkins_EPP622-2026-SNPCalling/edit/main/Test2.md

## Setup

Log onto the sphinx server and navigate to my directory. 
```
ssh sphinx
cd /lustre/isaac24/proj/UTK0505/yng832/
```
I made a directory for test 2 files, including a file for raw data and analyses. Also, into the raw_data file, I copied the reads from the class test2 directory. 
Create directory and symbolically link provided files.
```
mkdir test2_
cd test2_
mkdir raw_data
mkdir analyses
cd raw_data
cp /lustre/isaac24/proj/UTK0505/test2/reads/*gz .
cd ../analyses
mkdir 01_fastqc
mkdir 02_fastp
mkdir 03_bwamem2
mkdir 04_bctools
```

## Step 1: Quality Control (FastQC + MultiQC)
First, I opened the 01_fastqc folder and symbolically linked the raw data files to the directory. 
```
cd 01_fastqc
ln -s ../../raw_data/*gz .
```

Turn on the conda environment:
```
conda activate snps
```

Next, I ran fastqc for all the files.
```
fastqc -t 2 ../../raw_data/*.fastq.gz -o .
```

Once completed, I ran multiqc to compare samples and determine thier quality. Additionally, I renamed the files to reflect the multiqc reports for the raw data.
```
multiqc .
mv multiqc_data multiqc_raw_data
mv multiqc_report.html multiqc_report_raw.html
```
Then, in a new terminal, I moved this file to my desktop folder called EPP622 to open and view the MultiQC report.

```
scp yng832@login.isaac.utk.edu:/lustre/isaac24/proj/UTK0505/yng832/test2_/analyses/01_fastqc/multiqc_test2_raw.html .  
```
Linked results here: file:///Users/skylarcalkins/Desktop/EPP622/multiqc_test2_raw.html


MultiQC gives a rounded number for total reads and does not report total bases per sample. So to find out those numbers I used this:

```
nano reads_and_base_count.sh
```
inside reads_and_base_counts.sh:

```
for f in *.fastq.gz; do
  echo -ne "$f\t"
  zcat "$f" | awk 'NR%4==2{r++; b+=length($0)} END{printf "%d\t%d\t%.1f\n", r, b, b/r}'
done | column -t
```
Then back in terminal run
```
bash reads_and_base_count.sh
```

This gives the total reads, then the total bases, then the average read length for each file.

### Results

MultiQC report path and name: /lustre/isaac24/proj/UTK0505/yng832/test2_/analyses/01_fastqc/multiqc_test2_raw.html

| Sample | Total reads | Total bases | Quality issues? |
| --- | --- | --- | --- |
| A | 1609300 | 241395000 | Looks okay |
| B | 321860 | 48279000 | Low Coverage|
| C | 1609300 | 241395000 | Adapter Content / PolyG Issues|
| D | 1609300 | 241395000 | Low Quality|

## Step 2: Read Trimming and Filtering (fastp)




WARNING: Because these reads are simulated, fastp fails to identify that they need poly-G tail trimming. Turn that on with a flag.

ADVICE: Its easier to compare the reads pre and post trimming if you run multiqc on the results of fastqc of the trimmed files (and do not use the .json files from fastp).

### Commands

Next, we will trim the samples using fastp. First, we will move into the proper analyses folder and symbolically link our sample files:
```
cd ../02_fastp/
ln -s ../01_fastqc/*gz .
```
Now we will make a loop to run fastp across all the samples, both R1 and R2.
```
nano run_fastp.sh
```
Inside run_fastp.sh, I entered the following and then exited back to terminal:

```
#!/bin/bash
# run_fastp.sh
# Run fastp on every pair of paired-end FASTQ files in this folder.
# Usage:  bash run_fastp.sh

# Stop the script immediately if any command fails.
set -e

# Make a folder to hold the fastp reports (the -p means "don't complain if it alre$
mkdir -p fastp_reports
mkdir -p trimmed

# Loop over every R1 file in the current directory.
# The * is a wildcard: it matches ANY text, so this matches all 10 R1 files.
for R1 in *_R1.fastq.gz
do
    # Build the matching R2 filename by swapping "_R1" for "_R2" in the name.
    # The ${VAR/old/new} syntax replaces the first match of "old" with "new".
    R2="${R1/_R1/_R2}"

    # Build a short sample name by chopping "_R1.fastq.gz" off the end.
    # The ${VAR%pattern} syntax removes "pattern" from the END of the value.
    # Example: 21_FP_OH_FSB_F_10_R1.fastq.gz  ->  21_FP_OH_FSB_F_10
    SAMPLE="${R1%_R1.fastq.gz}"

    # Print a message so we can watch the progress in the terminal.
    echo "Now processing sample: $SAMPLE"

    # Safety check: make sure the R2 file actually exists before running fastp.
    if [ ! -f "$R2" ]; then
        echo "  WARNING: no R2 file found for $R1 -- skipping."
        continue    # "continue" jumps to the next sample in the loop
    fi

    # This is the fastp command, with the filenames replaced by variables.
    fastp \
	--in1 "$R1" \
        --out1 trimmed/"$SAMPLE"_R1.trimmed.fastq.gz \
        --in2 "$R2" \
        --out2 trimmed/"$SAMPLE"_R2.trimmed.fastq.gz \
        --detect_adapter_for_pe \
        --trim_poly_g \
        --thread 2 \
        --json fastp_reports/"$SAMPLE".fastp.json \
        --html fastp_reports/"$SAMPLE".fastp.html


done

echo "All done! Reports are in the fastp_reports/ folder."

```
Now back in terminal, enter
```
bash run_fastp.sh
```
This loop generates a folder containing our trimmed sample outputs, so we will open it up, run fastqc across each trimmed sample, and then run multiqc to compare the now trimmed samples.

```
cd trimmed/
fastqc -t 2 ./*.fastq.gz -o .

```
Now, I will secure copy to me Desktop again to open the MultiQC of the trimmed files. In the base terminal and still in my EPP622 files directory, I am entering:
```
scp yng832@login.isaac.utk.edu:/lustre/isaac24/proj/UTK0505/yng832/test2_/analyses/02_fastp/trimmed//multiqc_test2_trimmed/multiqc_test2_trimmed.html .
```
Next, I am copying over the reads_and_base_counts.sh file from our 01_fastqc file to get a clean report on total reads and total bases per sample. Then I will run it.

```
cp ../../01_fastqc/reads_and_base_count.sh .
bash reads_and_base_count.sh
```

This gives the total reads, then the total bases, then the average read length for each file.

### Results

MultiQC report path and name: `_______`

For each sample, report R1 + R2. Use the reference genome size above to calculate coverage.

| Sample | Reads before | Reads after | Mean length before | Mean length after |
| --- | --- | --- | --- | --- |
| A | 1609300| 1609300 | 241395000 | 241371911 |
| B | 321860 | 321860 | 48279000 | 48276193 |
| C | 1609300 | 1577258 | 241395000 | 198503407 |
| D | 1609300 | 1609300 | 241395000 | 241334120 |

## Step 3: Alignment (bwa-mem2 + samtools)

### Commands
Comments 

```
commands
```


### Results

MultiQC report path and name: `_______`

| Sample | % mapped | % properly paired | Mean depth |
| --- | --- | --- | --- |
| A | \- | \- | \- |
| B | \- | \- | \- |
| C | \- | \- | \- |
| D | \- | \- | \- |

## Step 4: Variant Calling (bcftools mpileup + call)

### Commands

Comments 

```
commands
```

### Results

bcftools stats report path and name: `_______`

## Step 5: Filtering (bcftools filter)

### Commands

I am going to try a gentle filtering and more stingent filtering. (WARNING: These are different filters than what we have seen in class.)

Gentle: exclude variants with a QUAL less than 20 or less than 5 total read depth across all samples.

Stringent: exclude variants with a QUAL less than 30 or less than 10 total read depth across all samples or a site level mapping quality (root mean square) of less than 40

```bash
#filtering gentle
#filtering stringent
```

```bash
# bcftools stats on gentle
# bcftools stats on stringent
```

### Results

bcftools stats report path and name for gentle filtering: `_______`

bcftools stats report path and name: for stringent filtering: `_______`

| VCF | number of SNPs |
| --- | --- |
| raw | \- |
| gentle | \- |
| stringent | \- |

## Step 6: Evaluation Against the Truth Set

### Commands

Link to the provided evaluate_snps_v4.py and truth_snps.vcf. Find the precision, recall and F1 for gently filtered and stringently filtered SNP sets.

Example of how to run evaluate_snps_v4.py:

```bash
python3 evaluate_snps_v4.py \
    --truth truth_snps.vcf --calls <yourvcffile> \
    --all-samples --tag strict --outdir outdir
```

```
code
```



in 05_ do :: python3 ../04_bctools/evaluate_snps_v4.py --truth ../04_bctools/truth_snps.vcf --calls ../04_bctools/test2_data.stringent_filter.vcf.gz --all-samples --tag gentle --outdir ../05_evaluation/




## Directory structure

From `this folder` that holds all my analysis, here is the directory structure (paste the output of: `tree .`):

```
#output of tree command:
```

### Results

| Sample | Filter | TP | FP | FN | Precision | Recall | F1 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| A | raw | \- | \- | \- | \- | \- | \- |
| A | gentle | \- | \- | \- | \- | \- | \- |
| A | strict | \- | \- | \- | \- | \- | \- |
| B | raw | \- | \- | \- | \- | \- | \- |
| B | gentle | \- | \- | \- | \- | \- | \- |
| B | strict | \- | \- | \- | \- | \- | \- |
| C | raw | \- | \- | \- | \- | \- | \- |
| C | gentle | \- | \- | \- | \- | \- | \- |
| C | strict | \- | \- | \- | \- | \- | \- |
| D | raw | \- | \- | \- | \- | \- | \- |
| D | gentle | \- | \- | \- | \- | \- | \- |
| D | strict | \- | \- | \- | \- | \- | \- |

## Assignment Analysis - Questions to answer

Minimum one paragraph each.

> How did samples differ from each other in read count versus in read quality? Why does each matter for SNP calling?

> What are the relative merits and drawbacks of filtering? Interpret the above chart in terms of the original quality issues, and how those yielded different outcomes. When is filtering a good idea vs when it is it a bad idea? Think specifically about false positives vs false negatives.

> Pick **two false negatives** and **two false positives**, from any sample. For each, tell me the sample, tell me the chromosome and coordinate location, put in a screenshot of IGV or JBrowse, and explain the specific cause of the FN or FP call.

## AI Analysis - Pipeline Audit

This is an evaluation of the pipeline you generated at the beginning of the learning module. Provide full sentences and thoughtful rationale.

Model used:

Date generated:

Exact prompt:

Minimum one paragraph each.

> Three things it got right.

> Three things you don't understand and would want to learn more about before using. Or do you understand all of it? If so be ready to answer questions about it.

> How specifically would you assess the results of this pipeline for accuracy?

> What did the labs teach you that turned out to matter for evaluating the AI-generated code?

> What is the value of learning about a computational genomics task before using AI for it — or is there none?






































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





