Your Name: - Skylar Calkins

Server: -

Repository URL: - https://github.com/SkylarElisabeth/Calkins_EPP622-2026-SNPCalling/edit/main/Test2.md

## Setup

Create directory and symbolically link provided files.

```
ssh sphinx
cd /lustre/isaac24/proj/UTK0505/yng832/
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

```
cd 01_fastqc
ln -s ../../raw_data/*gz .
nano individual_fastqc.sh
```
Within the individual_fastqc.sh file write the following:
```
for file in *fastq.gz
do
    fastqc -t 2 $file
done
```
Now continue in main terminal:
```
conda activate snps
bash individual_fastqc.sh
```
Once complete, run the following for Multiqc:
```
multiqc .
mv multiqc_report.html multiqc_raw_test2.html
```








MultiQC gives a rounded number for total reads and does not report total bases per sample. To find out those numbers use this:

```
nano reads_and_base_count.sh 
for f in *.fastq.gz; do
  echo -ne "$f\t"
  zcat "$f" | awk 'NR%4==2{r++; b+=length($0)} END{printf "%d\t%d\t%.1f\n", r, b, b/r}'
done | column -t
```

This gives the total reads, then the total bases, then the average read length for each file.

### Results

MultiQC report path and name: `_______`

One sample has adapter and polyG issues, one sample has low quality, one sample has low coverage, and one is problem free. Label each sample below with its quality issues.
For each sample, report R1 + R2.

| Sample | Total reads | Total bases | Quality issues? |
| --- | --- | --- | --- |
| A | 1609300 | 241395000 | \- |
| B | 321860 | 48279000 | \- |
| C | 1609300 | 241395000 | \- |
| D | 1609300 | 241395000 | \- |

## Step 2: Read Trimming and Filtering (fastp)

WARNING: Because these reads are simulated, fastp fails to identify that they need poly-G tail trimming. Turn that on with a flag.

ADVICE: Its easier to compare the reads pre and post trimming if you run multiqc on the results of fastqc of the trimmed files (and do not use the .json files from fastp).

### Commands

Comments 

```
commands
```

MultiQC gives a rounded number for total reads and does not report total bases per sample. To find out those numbers use this:

```
for f in *.fastq.gz; do
  echo -ne "$f\t"
  zcat "$f" | awk 'NR%4==2{r++; b+=length($0)} END{printf "%d\t%d\t%.1f\n", r, b, b/r}'
done | column -t
```

This gives the total reads, then the total bases, then the average read length for each file.

### Results

MultiQC report path and name: `_______`

For each sample, report R1 + R2. Use the reference genome size above to calculate coverage.

| Sample | Reads before | Reads after | Mean length before | Mean length after |
| --- | --- | --- | --- | --- |
| A | \- | \- | \- | \- |
| B | \- | \- | \- | \- |
| C | \- | \- | \- | \- |
| D | \- | \- | \- | \- |

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





