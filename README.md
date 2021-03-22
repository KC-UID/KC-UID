README
======

Introduction
=============

The `kcUID` software suite is used to process UID library RNA-seq reads in SeqHealth. `kcUID` suite have been compiled with C++ and are executable in the directory. If any questions, please contract SeqHealth.

Depends
========

The suite includes `kcUID`, `IdentifyUIDs`, `ClusterUID`, `CallConsensus`, `CorrectUID`, `Line2Fast`, `FastCount` and `UIDSummary`. 

It will also invoke some Linux commands, such as `pandoc`, `awk`, `wc`, `sort` and `rm` and please make sure these commands are in your PATH. 

The suite also relies on `R` to generate figs and html report. The `kcUIDReport_0.1.1.tar.gz` is the R package for `kcuid`. Before analysis, the `kcUIDReport` should be first installed in R which depends on `ggplot2`, `reshape2`, `rmarkdown`, `htmltools`, `readr`, `DT`, `knitr` and `kableExtra` packages.

Usage
======

The **kcUID** executable program is **the main entry point**. It will invoke the follow-up analyses and control the whole process. Try `kcUID --help` to see more helps.

The steps which `kcUID` will invoke are shown below.

1. Identify UID in reads: This step invokes `IdentifyUIDs` to identify UIDs from every read1.

    Commands as: 

	  	IdentifyUIDs -r1 test.clean.R1.fastq -r2 test.clean.R2.fastq -o test.uid.txt  -ur1 test.not.R1.fastq -ur2 test.not.R2.fastq -n 5

    The output 'test.uid.txt' will include UID, ReadID, Read1 with UID removed and Read2 if r2 is provided.

2. Sort UID: This step will sort the UIDs in the uid output.

    Command as: 

		sort --parallel 4 --buffer-size G -k 1,1 -o test.sort.uid.txt test.uid.txt


3. Cluster UID in every UID. A uid may contain some different molecules, so the step is to cluster similar sequences to form clusters within the same UID. 

	Command as
 
		ClusterUID -i test.sort.uid.txt -o test.clu.txt -t 10 -n 5'

    The output will include UID, sub-cluster ID in the same UID, Read1 and Read2 if r2 is provided.

4. `UIDSummary` is used to summay UID usage and cluster size distribution.

5. Get consensus sequence in every UID sub-cluster. For every sub-cluster, we will conduct multiple sequence alignment (MSA) and get an consensus sequence.

	Command as: 

		CallConsensus -i test.clu.txt -o test.con.txt -n 5

6. During PCR or sequencing, errors may happen in UIDs. So after consensus generation, we merged all of the identical consensus to form unique consensus sequences among similar uids.

	Command as: 

		sort --parallel 4 --buffer-size G -k 3,4 -o test.sort.con.txt test.con.txt
        CorrectUID -i test.sort.con.txt -o test.cor.con.txt

7. The output in the previous step is one line including read1 consensus and read2 consensus. But we always need fasta or fastq for downstream analysis. So change txt to fasta/fastq.

	Command as: 

		Line2Fast -i test.cor.con.txt -p test.con -type fasta
        or  
		Line2Fast -i test.cor.con.txt -p test.con -type fastq

8. `FastCount` is used to get reads information such as length distribution, base distribution, duplication level. It reads fasta or fastq files and algorithms are similar with FastQC.

Result
========

After running `kcUID`, you will get such results:

1. *.dedup.R1.fastq/fasta: The reads1 that have been dedup.

2. *.dedup.R2.fastq/fasta: The reads2 that have been dedup (if read2 has been provided).

3. *.noUID.R1.fastq/fasta：The reads1 that don't contain UIDs.

4. *.noUID.R2.fastq/fasta：The reads2 that don't contain UIDs.

5. *.kcUID.report.html: The kcUID html report.



<center>Copyright ©2019 SeqHealth, WuHan, China</center>
