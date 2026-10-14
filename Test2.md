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
mkdir 04_bcftools
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

MultiQC report path and name: /lustre/isaac24/proj/UTK0505/yng832/test2_/analyses/02_fastp/trimmed/multiqc_test2_trimmed/multiqc_test2_trimmed.html

For each sample, report R1 + R2. Use the reference genome size above to calculate coverage.

| Sample | Reads before | Reads after | Mean base-length before | Mean base-length after |
| --- | --- | --- | --- | --- |
| A | 1609300| 1609300 | 241395000 | 241371911 |
| B | 321860 | 321860 | 48279000 | 48276193 |
| C | 1609300 | 1577258 | 241395000 | 198503407 |
| D | 1609300 | 1609300 | 241395000 | 241334120 |

## Step 3: Alignment (bwa-mem2 + samtools)

Now to align the sequences!

First, I am going into the file I made for bwamem2 and calling the reference files into place.

```
cd ../../03_bwamem2/
ln -s ../../../../test2/ref/*.fa* .


```
Then, I will make a loop to take the trimmed files, convert them to SAM files and then convert to BAM.
```
nano sam_to_bam.sh
```
In nano of sam_to_bam.sh:
```
# Loop over every trimmed R1 file.
for R1 in ../02_fastp/trimmed/*_R1.trimmed.fastq.gz
do
    # Get R2 name by switching R1 with R2, retaining the full path
    R2="${R1/_R1/_R2}"

    # Find the short sample name - no path
    # first, extract the filename from a full file path.
    # The ## tells Bash to find the longest possible match from
    # the beginning of the string and delete it.
    # The */ matches any text ending with a forward slash.
    SAMPLE="${R1##*/}"
    # second, remove _R1.trimmed.fastq.gz from the file name
    SAMPLE="${SAMPLE%_R1.trimmed.fastq.gz}"

    echo "$R1"
    echo "$R2"
    echo "$SAMPLE"

    bwa-mem2 mem \
        sacCer3.fa \
        -t 2 \
	$R1 \
	$R2 \
      | samtools sort \
        -@ 2 \
        -o $SAMPLE.sorted.bam \
        -

    ## Build an index
    samtools index "$SAMPLE.sorted.bam"

done
```
Next, to pull the stats from our BAM files, including flagstats and coverage, we will make another loop.
```
nano check_stats.sh
```
In the nano of check_stats.sh:
```
for bamfile in *bam
do
        # The ${VAR%pattern} syntax removes "pattern" from the END of the value.
        SAMPLE=${bamfile%.sorted.bam}
        echo $SAMPLE

        samtools stats -@ 2 $bamfile > $SAMPLE.stats.txt
        samtools flagstat -@ 2 $bamfile > $SAMPLE.flagstat.txt
        samtools coverage $bamfile > $SAMPLE.coverage.txt

done
```
Now to run MultiQC using our stats data and export it to desktop.
```
multiqc . -n multiqc_test2_post-bam.html
```
In seperate, base terminal:
```
scp yng832@login.isaac.utk.edu:/lustre/isaac24/proj/UTK0505/yng832/test2_/analyses/03_bwamem2/multiqc_test2_post-bam.html .
```



### Results

MultiQC report path and name: /lustre/isaac24/proj/UTK0505/yng832/test2_/analyses/03_bwamem2/multiqc_test2_post-bam.html

| Sample | % mapped | % properly paired | Mean depth |
| --- | --- | --- | --- |
| A | 100 | 100 | 20 |
| B | 100 | 100 | 4 |
| C | 100 | 100 | 16.4 |
| D | 100 | 100 | 19.1 |

## Step 4: Variant Calling (bcftools mpileup + call)

First, I navigated into the bcftools folder and pulled in our BAM files.

```
cd ../04_bcftools
```

Then, I wrote a script for running mpileup
```
nano mpileup.sh
```
inside mpileup.sh:
```
bcftools mpileup \
  -Ou \
  -q 5 \
  -b bam_list.txt \
  -f /lustre/isaac24/proj/UTK0505/test2/ref/sacCer3.fa \
  -a AD,ADF,ADR,DP,SP | \
bcftools call \
  -m \
  -v \
  -Oz \
  -o test2_data.raw.vcf.gz
```
I then generated a summary statistic for test2_data.raw.vcf.gz
```
bcftools stats  test2_data.raw.vcf.gz > test2_data.raw.stats.txt
```

### Results

bcftools stats report path and name: /lustre/isaac24/proj/UTK0505/yng832/test2_/analyses/04_bcftools/test2_data.raw.stats.txt 

The raw VCF contained:
- 4 samples
- 4920 variant records
- 4920 SNPs
- 0 indels
- 0 MNPs
- 0 other variant types

## Step 5: Filtering (bcftools filter)

I am going to try a gentle filtering and more stingent filtering. 

Gentle: exclude variants with a QUAL less than 20 or less than 5 total read depth across all samples.

Stringent: exclude variants with a QUAL less than 30 or less than 10 total read depth across all samples or a site level mapping quality (root mean square) of less than 40

For gentle filtering:
```
bcftools filter  -e 'TYPE="indel" | QUAL<10 | INFO/DP<5'  -o test2_data.gentle_filter.vcf.gz test2_data.raw.vcf.gz

```
For stringent filtering:
```
bcftools filter  -e 'TYPE="indel" | QUAL<30 | INFO/DP<10 | INFO/MQ<40'  -o test2_data.stringent_filter.vcf.gz test2_data.raw.vcf.gz
```

### Results

bcftools stats report path and name for gentle filtering: /lustre/isaac24/proj/UTK0505/yng832/test2_/analyses/04_bcftools/test2_data.gentle_filter.stats.txt

bcftools stats report path and name: for stringent filtering: /lustre/isaac24/proj/UTK0505/yng832/test2_/analyses/04_bcftools/test2_data.stringent_filter.stats.txt

| VCF | number of SNPs |
| --- | --- |
| raw | 4920|
| gentle | 4869 |
| stringent | 4774 |

## Step 6: Evaluation Against the Truth Set
First, I am creating a directory for evaluation and linking to the provided evaluate_snps_v4.py and truth_snps.vcf.
```
cd ../
mkdir 05_evaluation
cd 05_evaluation/
ln -s  ../../../../test2/check/evaluate_snps_v4.py .
ln -s  ../../../../test2/check/truth_snps.vcf .
```
Now running evaluate_snps_v4.py on raw, gentle, and stringent files:
```
python3 ./evaluate_snps_v4.py --truth ./truth_snps.vcf --calls ../04_bcftools/test2_data.raw.vcf.gz --all-samples --tag raw --outdir ../05_evaluation/
python3 ./evaluate_snps_v4.py --truth ./truth_snps.vcf --calls ../04_bcftools/test2_data.gentle_filter.vcf.gz --all-samples --tag gentle --outdir ../05_evaluation/
python3 ./evaluate_snps_v4.py --truth ./truth_snps.vcf --calls ../04_bcftools/test2_data.stringent_filter.vcf.gz --all-samples --tag stringent --outdir ../05_evaluation/
```
Exporting the resulting summary table into a .csv so I can open in excel for easier viewing:
```
python3 -c 'import csv, sys; csv.writer(sys.stdout).writerows(csv.reader(sys.stdin, delimiter="\t"))'   < summary_all_runs.tsv > summary_all_runs.csv
```
## Directory structure

My test2 is structured like this: 
```
.
├── analyses
│   ├── 01_fastqc
│   │   ├── individual_fastqc.sh
│   │   ├── multiqc_raw_data
│   │   │   ├── fastqc_adapter_content_plot.txt
│   │   │   ├── fastqc_overrepresented_sequences_plot.txt
│   │   │   ├── fastqc_per_base_n_content_plot.txt
│   │   │   ├── fastqc_per_base_sequence_quality_plot.txt
│   │   │   ├── fastqc_per_sequence_gc_content_plot_Counts.txt
│   │   │   ├── fastqc_per_sequence_gc_content_plot_Percentages.txt
│   │   │   ├── fastqc_per_sequence_quality_scores_plot.txt
│   │   │   ├── fastqc_sequence_counts_plot.txt
│   │   │   ├── fastqc_sequence_duplication_levels_plot.txt
│   │   │   ├── fastqc-status-check-heatmap.txt
│   │   │   ├── fastqc_top_overrepresented_sequences_table.txt
│   │   │   ├── llms-full.txt
│   │   │   ├── multiqc_citations.txt
│   │   │   ├── multiqc_data.json
│   │   │   ├── multiqc_fastqc.txt
│   │   │   ├── multiqc_general_stats.txt
│   │   │   ├── multiqc.log
│   │   │   ├── multiqc.parquet
│   │   │   ├── multiqc_software_versions.txt
│   │   │   └── multiqc_sources.txt
│   │   ├── multiqc_test2_raw.html
│   │   ├── reads_and_base_count.sh
│   │   ├── Sample_A_R1_fastqc.html
│   │   ├── Sample_A_R1_fastqc.zip
│   │   ├── Sample_A_R1.fastq.gz -> ../../raw_data/Sample_A_R1.fastq.gz
│   │   ├── Sample_A_R2_fastqc.html
│   │   ├── Sample_A_R2_fastqc.zip
│   │   ├── Sample_A_R2.fastq.gz -> ../../raw_data/Sample_A_R2.fastq.gz
│   │   ├── Sample_B_R1_fastqc.html
│   │   ├── Sample_B_R1_fastqc.zip
│   │   ├── Sample_B_R1.fastq.gz -> ../../raw_data/Sample_B_R1.fastq.gz
│   │   ├── Sample_B_R2_fastqc.html
│   │   ├── Sample_B_R2_fastqc.zip
│   │   ├── Sample_B_R2.fastq.gz -> ../../raw_data/Sample_B_R2.fastq.gz
│   │   ├── Sample_C_R1_fastqc.html
│   │   ├── Sample_C_R1_fastqc.zip
│   │   ├── Sample_C_R1.fastq.gz -> ../../raw_data/Sample_C_R1.fastq.gz
│   │   ├── Sample_C_R2_fastqc.html
│   │   ├── Sample_C_R2_fastqc.zip
│   │   ├── Sample_C_R2.fastq.gz -> ../../raw_data/Sample_C_R2.fastq.gz
│   │   ├── Sample_D_R1_fastqc.html
│   │   ├── Sample_D_R1_fastqc.zip
│   │   ├── Sample_D_R1.fastq.gz -> ../../raw_data/Sample_D_R1.fastq.gz
│   │   ├── Sample_D_R2_fastqc.html
│   │   ├── Sample_D_R2_fastqc.zip
│   │   ├── Sample_D_R2.fastq.gz -> ../../raw_data/Sample_D_R2.fastq.gz
│   │   └── testing_name
│   │       ├── testing_name_data
│   │       │   ├── fastqc_adapter_content_plot.txt
│   │       │   ├── fastqc_overrepresented_sequences_plot.txt
│   │       │   ├── fastqc_per_base_n_content_plot.txt
│   │       │   ├── fastqc_per_base_sequence_quality_plot.txt
│   │       │   ├── fastqc_per_sequence_gc_content_plot_Counts.txt
│   │       │   ├── fastqc_per_sequence_gc_content_plot_Percentages.txt
│   │       │   ├── fastqc_per_sequence_quality_scores_plot.txt
│   │       │   ├── fastqc_sequence_counts_plot.txt
│   │       │   ├── fastqc_sequence_duplication_levels_plot.txt
│   │       │   ├── fastqc-status-check-heatmap.txt
│   │       │   ├── fastqc_top_overrepresented_sequences_table.txt
│   │       │   ├── llms-full.txt
│   │       │   ├── multiqc_citations.txt
│   │       │   ├── multiqc_data.json
│   │       │   ├── multiqc_fastqc.txt
│   │       │   ├── multiqc_general_stats.txt
│   │       │   ├── multiqc.log
│   │       │   ├── multiqc.parquet
│   │       │   ├── multiqc_software_versions.txt
│   │       │   └── multiqc_sources.txt
│   │       └── testing_name.html
│   ├── 02_fastp
│   │   ├── fastp_reports
│   │   │   ├── Sample_A.fastp.html
│   │   │   ├── Sample_A.fastp.json
│   │   │   ├── Sample_B.fastp.html
│   │   │   ├── Sample_B.fastp.json
│   │   │   ├── Sample_C.fastp.html
│   │   │   ├── Sample_C.fastp.json
│   │   │   ├── Sample_D.fastp.html
│   │   │   └── Sample_D.fastp.json
│   │   ├── run_fastp.sh
│   │   ├── Sample_A_R1.fastq.gz -> ../01_fastqc/Sample_A_R1.fastq.gz
│   │   ├── Sample_A_R2.fastq.gz -> ../01_fastqc/Sample_A_R2.fastq.gz
│   │   ├── Sample_B_R1.fastq.gz -> ../01_fastqc/Sample_B_R1.fastq.gz
│   │   ├── Sample_B_R2.fastq.gz -> ../01_fastqc/Sample_B_R2.fastq.gz
│   │   ├── Sample_C_R1.fastq.gz -> ../01_fastqc/Sample_C_R1.fastq.gz
│   │   ├── Sample_C_R2.fastq.gz -> ../01_fastqc/Sample_C_R2.fastq.gz
│   │   ├── Sample_D_R1.fastq.gz -> ../01_fastqc/Sample_D_R1.fastq.gz
│   │   ├── Sample_D_R2.fastq.gz -> ../01_fastqc/Sample_D_R2.fastq.gz
│   │   └── trimmed
│   │       ├── multiqc_test2_trimmed
│   │       │   ├── multiqc_test2_trimmed_data
│   │       │   │   ├── fastqc_adapter_content_plot.txt
│   │       │   │   ├── fastqc_per_base_n_content_plot.txt
│   │       │   │   ├── fastqc_per_base_sequence_quality_plot.txt
│   │       │   │   ├── fastqc_per_sequence_gc_content_plot_Counts.txt
│   │       │   │   ├── fastqc_per_sequence_gc_content_plot_Percentages.txt
│   │       │   │   ├── fastqc_per_sequence_quality_scores_plot.txt
│   │       │   │   ├── fastqc_sequence_counts_plot.txt
│   │       │   │   ├── fastqc_sequence_duplication_levels_plot.txt
│   │       │   │   ├── fastqc_sequence_length_distribution_plot.txt
│   │       │   │   ├── fastqc-status-check-heatmap.txt
│   │       │   │   ├── llms-full.txt
│   │       │   │   ├── multiqc_citations.txt
│   │       │   │   ├── multiqc_data.json
│   │       │   │   ├── multiqc_fastqc.txt
│   │       │   │   ├── multiqc_general_stats.txt
│   │       │   │   ├── multiqc.log
│   │       │   │   ├── multiqc.parquet
│   │       │   │   ├── multiqc_software_versions.txt
│   │       │   │   └── multiqc_sources.txt
│   │       │   └── multiqc_test2_trimmed.html
│   │       ├── reads_and_base_count.sh
│   │       ├── Sample_A_R1.trimmed_fastqc.html
│   │       ├── Sample_A_R1.trimmed_fastqc.zip
│   │       ├── Sample_A_R1.trimmed.fastq.gz
│   │       ├── Sample_A_R2.trimmed_fastqc.html
│   │       ├── Sample_A_R2.trimmed_fastqc.zip
│   │       ├── Sample_A_R2.trimmed.fastq.gz
│   │       ├── Sample_B_R1.trimmed_fastqc.html
│   │       ├── Sample_B_R1.trimmed_fastqc.zip
│   │       ├── Sample_B_R1.trimmed.fastq.gz
│   │       ├── Sample_B_R2.trimmed_fastqc.html
│   │       ├── Sample_B_R2.trimmed_fastqc.zip
│   │       ├── Sample_B_R2.trimmed.fastq.gz
│   │       ├── Sample_C_R1.trimmed_fastqc.html
│   │       ├── Sample_C_R1.trimmed_fastqc.zip
│   │       ├── Sample_C_R1.trimmed.fastq.gz
│   │       ├── Sample_C_R2.trimmed_fastqc.html
│   │       ├── Sample_C_R2.trimmed_fastqc.zip
│   │       ├── Sample_C_R2.trimmed.fastq.gz
│   │       ├── Sample_D_R1.trimmed_fastqc.html
│   │       ├── Sample_D_R1.trimmed_fastqc.zip
│   │       ├── Sample_D_R1.trimmed.fastq.gz
│   │       ├── Sample_D_R2.trimmed_fastqc.html
│   │       ├── Sample_D_R2.trimmed_fastqc.zip
│   │       └── Sample_D_R2.trimmed.fastq.gz
│   ├── 03_bwamem2
│   │   ├── multiqc_test2_post-bam_data
│   │   │   ├── llms-full.txt
│   │   │   ├── multiqc_citations.txt
│   │   │   ├── multiqc_data.json
│   │   │   ├── multiqc_general_stats.txt
│   │   │   ├── multiqc.log
│   │   │   ├── multiqc.parquet
│   │   │   ├── multiqc_samtools_coverage.txt
│   │   │   ├── multiqc_samtools_flagstat.txt
│   │   │   ├── multiqc_samtools_stats.txt
│   │   │   ├── multiqc_software_versions.txt
│   │   │   ├── multiqc_sources.txt
│   │   │   ├── samtools_alignment_plot.txt
│   │   │   ├── samtools-coverage_Bases.txt
│   │   │   ├── samtools-coverage_BQ.txt
│   │   │   ├── samtools-coverage_Coverage.txt
│   │   │   ├── samtools-coverage_Mean_depth.txt
│   │   │   ├── samtools-coverage_MQ.txt
│   │   │   ├── samtools-coverage_Reads.txt
│   │   │   ├── samtools-coverage-table.txt
│   │   │   ├── samtools-flagstat-pct-table.txt
│   │   │   ├── samtools-flagstat-table.txt
│   │   │   ├── samtools_insert_size.txt
│   │   │   └── samtools-stats-dp.txt
│   │   ├── multiqc_test2_post-bam.html
│   │   ├── sacCer3.fa -> ../../../../test2/ref/sacCer3.fa
│   │   ├── sacCer3.fa.0123 -> ../../../../test2/ref/sacCer3.fa.0123
│   │   ├── sacCer3.fa.amb -> ../../../../test2/ref/sacCer3.fa.amb
│   │   ├── sacCer3.fa.ann -> ../../../../test2/ref/sacCer3.fa.ann
│   │   ├── sacCer3.fa.bwt.2bit.64 -> ../../../../test2/ref/sacCer3.fa.bwt.2bit.64
│   │   ├── sacCer3.fa.fai -> ../../../../test2/ref/sacCer3.fa.fai
│   │   ├── sacCer3.fa.pac -> ../../../../test2/ref/sacCer3.fa.pac
│   │   ├── Sample_A.coverage.txt
│   │   ├── Sample_A.flagstat.txt
│   │   ├── Sample_A.sorted.bam
│   │   ├── Sample_A.sorted.bam.bai
│   │   ├── Sample_A.stats.txt
│   │   ├── Sample_B.coverage.txt
│   │   ├── Sample_B.flagstat.txt
│   │   ├── Sample_B.sorted.bam
│   │   ├── Sample_B.sorted.bam.bai
│   │   ├── Sample_B.stats.txt
│   │   ├── Sample_C.coverage.txt
│   │   ├── Sample_C.flagstat.txt
│   │   ├── Sample_C.sorted.bam
│   │   ├── Sample_C.sorted.bam.bai
│   │   ├── Sample_C.stats.txt
│   │   ├── Sample_D.coverage.txt
│   │   ├── Sample_D.flagstat.txt
│   │   ├── Sample_D.sorted.bam
│   │   ├── Sample_D.sorted.bam.bai
│   │   ├── Sample_D.stats.txt
│   │   ├── sam_to_bam.sh
│   │   └── stats_check.sh
│   ├── 04_bcftools
│   │   ├── bam_list.txt
│   │   ├── mpileup.sh
│   │   ├── test2_data.gentle_filter.stats.txt
│   │   ├── test2_data.gentle_filter.vcf.gz
│   │   ├── test2_data.raw.stats.txt
│   │   ├── test2_data.raw.vcf.gz
│   │   ├── test2_data.stringent_filter.stats.txt
│   │   └── test2_data.stringent_filter.vcf.gz
│   └── 05_evaluation
│       ├── evaluate_snps_v4.py -> ../../../../test2/check/evaluate_snps_v4.py
│       ├── Sample_A_gentle_false_negatives.tsv
│       ├── Sample_A_gentle_false_positives.tsv
│       ├── Sample_A_gentle_genotype_confusion.tsv
│       ├── Sample_A_raw_false_negatives.tsv
│       ├── Sample_A_raw_false_positives.tsv
│       ├── Sample_A_raw_genotype_confusion.tsv
│       ├── Sample_A_stringent_false_negatives.tsv
│       ├── Sample_A_stringent_false_positives.tsv
│       ├── Sample_A_stringent_genotype_confusion.tsv
│       ├── Sample_B_gentle_false_negatives.tsv
│       ├── Sample_B_gentle_false_positives.tsv
│       ├── Sample_B_gentle_genotype_confusion.tsv
│       ├── Sample_B_raw_false_negatives.tsv
│       ├── Sample_B_raw_false_positives.tsv
│       ├── Sample_B_raw_genotype_confusion.tsv
│       ├── Sample_B_stringent_false_negatives.tsv
│       ├── Sample_B_stringent_false_positives.tsv
│       ├── Sample_B_stringent_genotype_confusion.tsv
│       ├── Sample_C_gentle_false_negatives.tsv
│       ├── Sample_C_gentle_false_positives.tsv
│       ├── Sample_C_gentle_genotype_confusion.tsv
│       ├── Sample_C_raw_false_negatives.tsv
│       ├── Sample_C_raw_false_positives.tsv
│       ├── Sample_C_raw_genotype_confusion.tsv
│       ├── Sample_C_stringent_false_negatives.tsv
│       ├── Sample_C_stringent_false_positives.tsv
│       ├── Sample_C_stringent_genotype_confusion.tsv
│       ├── Sample_D_gentle_false_negatives.tsv
│       ├── Sample_D_gentle_false_positives.tsv
│       ├── Sample_D_gentle_genotype_confusion.tsv
│       ├── Sample_D_raw_false_negatives.tsv
│       ├── Sample_D_raw_false_positives.tsv
│       ├── Sample_D_raw_genotype_confusion.tsv
│       ├── Sample_D_stringent_false_negatives.tsv
│       ├── Sample_D_stringent_false_positives.tsv
│       ├── Sample_D_stringent_genotype_confusion.tsv
│       ├── summary_all_runs.csv
│       ├── summary_all_runs.tsv
│       └── truth_snps.vcf -> ../../../../test2/check/truth_snps.vcf
└── raw_data
    ├── Sample_A_R1.fastq.gz
    ├── Sample_A_R2.fastq.gz
    ├── Sample_B_R1.fastq.gz
    ├── Sample_B_R2.fastq.gz
    ├── Sample_C_R1.fastq.gz
    ├── Sample_C_R2.fastq.gz
    ├── Sample_D_R1.fastq.gz
    └── Sample_D_R2.fastq.gz
```

### Results

| Sample | Filter | TP | FP | FN | Precision | Recall | F1 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| A | raw | 4875 | 0 | 125 | 1 | 0.9750 | 0.9873|
| A | gentle | 4864 | 0 | 136 | 1 | 0.9728 | 0.9862 |
| A | strict | 4772 | 0 | 228 | 1 |	0.9544 | 0.9767 |
| B | raw | 4422 | 23 | 578 | 0.9948 | 0.8844 | 0.9364|
| B | gentle | 4419 | 0 | 581 | 1 | 0.8838 | 0.9383 |
| B | strict | 4347 | 0 | 653 | 1 | 0.8694 | 0.9301 |
| C | raw | 4871 | 2 | 129 | 0.9996 | 0.9742 | 0.9867
| C | gentle | 4865 | 0 | 135 | 1 | 0.9730 | 0.9863 |
| C | strict | 4772 | 0 | 228 | 1 | 0.9544 | 0.9767 |
| D | raw | 4859 | 15 | 141 | 0.9969 | 0.9718 | 0.9842 |
| D | gentle | 4851 | 1 | 149 | 0.9998 | 0.9702 | 0.9848 |
| D | strict | 4761 | 0 | 239 | 1 | 0.9522 | 0.9755 |

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


