# Sg339ss1
Genome assembly of Pyricularia oryzae (Isolate Sg339ss1) 
## MyGenome - Sg339ss1
**Repository URL:** https://github.com/OgheneyomaIkpeni/Sg339ss1/

This repository consists bash command line codes for sequence quality assessment and trimming, genome and transcriptome assembly, gene prediction, expression analysis and data visualization.

---

### Table of Contents
1. [Sequence Data, Quality Assessment and Trimming](#sequence-data-quality-assessment-and-trimming)
2. [Genome Assembly](#genome-assembly)
3. [BLAST](#Blast)

---

#### Sequence Data, Quality Assessment and Trimming

    This contains:
    1. code used to download data, assess quality and perform trimming 
    2. Summary of FASTQC metrics and note warnings and failures.
    3. Images of Summary and Adaptor Content tabs (before and after trimming) 


##### Copy sequence data to the working directory (sequence) on the VM


1. Log into the VM

   ```
   ssh opik222@opik222.cs.uky.edu
   ```
2. cd into the sequences directory
    ```
   cd sequences
   ```
3. Transfer the directory that contains the fastq data into the current working directory:

```
 scp -r fastq opik222@opik222.cs.uky.edu:~/sequences/Sg339ss1

```

### Assess Sequence Quality with fastqc

1. Run FastQC on the forward and reverse reads:

```
 fastqc -o ~/sequences/Sg339ss1 Sg339ss1_1.fq.gz Sg339ss1_2.fq.gz
```

2. Confirm the FastQC reports were created
   ```
 ls -lh *_fastqc.html *_fastqc.zip
```
You should see 4 new files (2 html + 2 zip).2. 
<Sg339ss1_1_fastqc.html
Sg339ss1_1_fastqc.zip
Sg339ss1_2_fastqc.html
Sg339ss1_2_fastqc.zip>

3. Logout of the VM

 ```
 exit
```

4. Open a terminal on the local machine

5. Transfer the two html files to your local machine

 ```
scp opik222@opik222.cs.uky.edu:~/sequences/Sg339ss1/*_fastqc.html .
```

6. Navigate to the downloaded html files using the file browser

7. Open the FASTQC reports by double-clicking on the files and browse the various metrics by clicking on the tabs under summary.


###### Summary of FASTQC metrics and note warnings and failures
Total sequences (#raw reads (single end)) = 6,623,435
Total Bases = 993.5Mbp
Sequence length = 150
%GC = 50


#Images of adaptor content tab before trimming

<img width="1110" height="600" alt="image" src="https://github.com/user-attachments/assets/f703a578-e45f-4ebe-bfd2-a751c04c0ac7" />


#Images of summary tab

<img width="818" height="476" alt="image" src="https://github.com/user-attachments/assets/97b00f3d-3671-44df-9681-c1a2addbba36" />


####### Sequence Trimming with Trimmomatic

1. Go to the sequences directory on the VM
```
cd ~/sequences
```

2. Use nano to edit the adaptors.fa
```
nano adaptors.fa
```
3. Add a new fasta sequence record named polyG and with the sequence G x 20

4.Run trimmomatic on the new dataset

```
java -jar trimmomatic-0.38.jar PE -threads 2 -phred33 -trimlog Sg339ss1_errorlog.txt Sg339ss1/Sg339ss1_1.fq.gz Sg339ss1/Sg339ss1_2.fq.gz Sg339ss1_1_paired.fastq Sg339ss1_1_unpaired.fastq Sg339ss1_2_paired.fastq Sg339ss1_2_unpaired.fastq ILLUMINACLIP:adaptors.fa:2:30:10 SLIDINGWINDOW:20:20 MINLEN:125

```


5. Run FASTQC on the four trimmed files
```
fastqc Sg339ss1_*paired.fastq Sg339ss1_*unpaired.fastq

```

6. Secure copy trimmed files to the local machine

Run in powershell
```
scp opik222@opik222.cs.uky.edu:~/sequences/Sg339ss1_*fastqc.html .

```


#Summary of Statistics on trimmed reads

Total sequences (#raw reads (single end)) = 5,750,270
Total Bases = 1720.6Mbp
Sequence length = 125 - 150
%GC = 50

#images of summary statistics of paired reads after trimming

<img width="862" height="502" alt="image" src="https://github.com/user-attachments/assets/e66339c7-eb05-49dc-950a-fa670c426336" />

#images of adaotor content of paired reads after trimming

<img width="1110" height="600" alt="image" src="https://github.com/user-attachments/assets/261db49d-8b8b-4ef9-8099-16c3715cf8b8" />




---
#Genome Assembly

##Prepare genome assembly workspace

1. Log onto MCC

```
ssh opik222@mcc.uky.edu 
```
2. Create a directory inside /project/farman_s26abt480/ named opik222

```
cd /project/farman_s26abt40
mkdir opik222

```
3. Create a genome project directory inside /project/farman_s26abt480/ named opik222

```
cd opik222
mkdir Sg339ss1

```
4. Securecopy trimmed reads (paired and unpaired) from the VM into the project directory (Sg339ss1)

```
exit
ssh opik222@opik222.cs.uky.edu
scp Sg339ss1_1_paired.fastq \
Sg339ss1_1_unpaired.fastq \
Sg339ss1_2_paired.fastq \
Sg339ss1_2_unpaired.fastq \
opik222@mcc.uky.edu:/project/farman_s26abt480/opik222/Sg339ss1/

enter password

```

#Edit the assembly script

1. Copy the velvetoptimiser.sh script from /project/farman_s26abt40/SLURM_SCRIPTS to the project folder

``
exit
ssh opik222@mcc.uky.edu 
cd project/farman_s26abt480/opik222/Sg339ss1
cp project/farman_s26abt480/SLURM_SCRIPTS/velvetoptimiser.sh .

```

2. Edit the velvetoptimiser.sh script and change the myLinkBlueID@uky.edu entry to opik222@uky.edu (your email address)
``
nano velvetoptimiser.sh

```
3.Run velvet using a range of k-mer values
``
sbatch projsbatch velvetoptimiser.sh Sg339ss1 111 191 10 

```
4. After completion of run, examine the velevt_Sg339ss1_111_191-noclean output directory which contains eight output files: contigs.fa, Graph; Graph2; Log PreGraph; Sequences; stats.txt: and "Logfile.txt"

5.Find the information on the final optimized assembly from the end of the velvetoptimiser runtime message

#Summary of genome assembly metrics using velvet

Genome size (total bases in contigs) : 41,702,572 bp
Number of scaffolds (contigs)* : 5,832
Longest scaffold: 178,639
N50: 28,988

#Further optimize the velvet assembly

1. Select new start and end kmer sizes (using a step size of 2- 71, 121)

``
sbatch velvetoptimiser.sh Sg339ss1 71 121 10 

```
   
3. Run velvet using new range of k-mer values as done in the first run
4. Compare final metrics using the end of the velvetoptimiser runtime message or the new Logfile.txt file

#Summary of optimized genome assembly metrics using velvet

Genome size (total bases in contigs) : 41,572,814 bp
Number of scaffolds (contigs)* : 4,009
Longest scaffold: 325,177 bp
N50: 69,586

#Generate new assembly using SPAdes
1. cd into the genome project directrory

``
cd project/farman_s26abt480/opik222/Sg339ss1

```

2. Copy the spades.sh script from /project/farman_s26abt40/SLURM_SCRIPTS to the project directory

```
scp opik222@mcc.uky.edu:/project/farman_s26abt480/SLURM_SCRIPTs/spades.sh .

```
3. Edit the spades.sh script and  change the myLinkBlueID@uky.edu entry to opik222@uky.edu (your email address)

4. Run the spades.sh script using default kmer values (21, 33, 55, 77)
```
sbatch spades.sh Sg339ss1_1_paired.fastq Sg339ss1_2_paired.fastq MyGenomeID_spades_assembly 21 33 55 77

```
5. Determine the number of contigs in the finsl assembly (scaffolds.fasta)
   
```
grep -v ">" Sg339ss1_spades_assembly/scaffolds.fasta | wc -c

awk '/^>/ {if (seqlen){print seqlen}; seqlen=0; next} {seqlen+=length($0)} END{print seqlen}' Sg339ss1_spades_assembly/scaffolds.fasta | sort -nr | awk '{sum+=$1; arr[NR]=$1} END{for(i=1;i<=NR;i++){cumsum+=arr[i]; if(cumsum>=sum/2){print arr[i]; break}}}'

```
    
   
#Summary of optimized genome assembly metrics using SPAdes

Genome size: 42,292,228 bp
No of Contigs (scaffolds): 5,897
N50: 100,494 bp


#Visualization of Optimized genome assembly using Bandage







#### Sg339ss1 Genome Gene Prediction

    This section contains:
    1. Steps used to generate the HMM file used for gene prediction
    2. Summary of outputs from snap and AUGUSTUS and MAKER gene predictions (#s of gene identified)
    3. Screenshot of an IGV window showing a few MAKER gene models 


##### Generate HMM file to be used for gene prediction 

1. Log into the VM

   ```
   ssh opik222@opik222.cs.uky.edu
   ```

2. Start a screen session

   ```
   screen -S genes bash -l
   ```
3. List available models

   ```
   echo $ZOE
   ```
4. Look inside the directory

   ```
   ls $ZOE
   ```



   ##### Generate training data for SNAP and AUGUSTUS


1. Change to the snap directory under ~/genes where the intermediate results and output will be kept.

   ```
   cd ~/genes/snap
   ```

2. Download the B71ref2.fasta genome and B71Ref2_a0.3.gff3 annotation file from the /project/farman_26abt480/RESOURCES directory.

   ```
 scp opik222@mcc.uky.edu:/project/farman_s26abt480/RESOURCES/B71Ref2.fasta . 
 scp opik222@mcc.uky.edu:/project/farman_s26abt480/RESOURCES/B71Ref2_a0.3.gff3 .
   ```
3. Append the genome fasta sequence to the end of the gff3

   ```
  echo '##FASTA' | cat B71Ref2_a0.3.gff3 - B71Ref2.fasta > B71Ref2.gff3
   ```
4. Check that the B71Ref2.gff3 file has the correct format

   ```
   grep '##FASTA' -B 5 -A 5 B71Ref2.gff3
   ```

5. Convert the MAKER annotations to ZFF for SNAP

   ```
  maker2zff B71Ref2.gff3
   ```

6. Using -gene-stats,get information on the annotations. #This subcommand outputs to the screen the number of sequences (contigs), the number of genes annotated, GC content, average intron and exon lengths, and more.

   ```
  fathom genome.ann genome.dna -gene-stats
   ```
7. Extract the genome regions containing unique genes using the -categorize subcommand. #This subcommand creates a number of pairs of ZFF and FASTA files: one pair for regions with errors, one for regions with overlapping genes, and so on. We will ask for up to 1000 base pairs of intergenic sequence on both sides of each gene to help train the HMM about what sequences are likely to occur near genes.

   ```
   fathom genome.ann genome.dna -categorize 1000
   ```

8. Use annotations found in uni.ann to train SNAP. #Training SNAP  usually works best with unique, non-overlapping genes without alternative splicing; these annotations may be found in uni.ann, and the accompanying sequences in uni.dna. look at their statistics using

   ```
  fathom uni.ann uni.dna -gene-stats
   ```

9. use the -export subcommand to extract the genome, transcript, and protein sequences from these genes. We must tell -export as well to keep 1000 base pairs of context and also (with -plus) to flip genes that are on the reverse strand (that is, with their 3' end nearer the beginning of the contig than their 5'). The output is in four files:
#export.ann ZFF annotations for the exons of each gene.
#export.dna DNA sequence for each gene, including introns and flanking regions.
#export.tx DNA sequence for each transcript.
#export.aa Protein sequence for each gene.

   ```
   fathom uni.ann uni.dna -export 1000 -plus
   ```

10. The data is now in suitable format to train the HMM using the forge tool.
   ```
  forge export.ann export.dna
   ```


11.Use SNAP tool, hmm-assembler.pl, to condense the large  number of .model and .count files generated from using the forge tool into a single file for use with runs of SNAP.
   ```
 hmm-assembler.pl Moryzae . > Moryzae.hmm
   ```

12.Peek at the Moryzae.hmm file to get a sense of how the gene statistics are represented.
   ```
 head Moryzae.hmm
   ```
13.Copy the Moryzae.hmm file to the Sg339ss1 directory on MCC. We will need it for running MAKER later
   ```
scp ~/genes/snap/Moryzae.hmm opik222@mcc.uky.edu:/project/farman_s26abt480/opik222/Sg339ss1/
   ```




##### Summary of outputs from snap and AUGUSTUS and MAKER gene predictions (#s of gene identified)
1. Number of genes identified using SNAP = 12, 737
2. Number of genes identified using AUGUSTUS = 5366
3. Number of genes identified using MAKER = 12,849


###### Screenshot of an IGV window showing a few MAKER gene models







# Blasting the Sg339ss1 genome
A BLAST comparison between Sg339ss1  and B71 reference was performed using BLASTN (e-value ≤ 1e-100).

 This section contains
1. Code used to identify mitochondrial contigs in Sg339ss1 assembly
2. Code used to align Sg339ss1 Genome with B71 reference genome
3. Record of the number of contigs in Sg339ss1 assembly that lack matches in B71 reference genome
4. gff file of Sg339ss1 genome aligned against B1 reference
5. Screenshot of a B71 chromosome that contains a large block of sequence absent from Sg339ss1 genome.



 ## Identify mitochondrial contigs in Sg339ss1 assembly

1. Login to the VM and cd into the blast directory
   ```
   myVM
   cd blast
   ```

2. Copy the final genome assembly from MCC into the blast directory
   ```
 scp opik222@mcc.uky.edu:/project/farman_s26abt480/opik222/Sg339ss1/Sg339ss1_final.fasta . 

   ```
3. Copy the B71.fasta file from /project/farman_s26abt480/BLAST into the blast directory

   ```
  scp opik222@mcc.uky.edu:/project/farman_s26abt480/BLAST/B71.fasta .
   ```
4. Perform a BLAST search using the B71 genome as a query and MyGenome as the subject:

   ```
   blastn -query B71.fasta -subject Sg339ss1_final.fasta -evalue 1e-100 -outfmt 7 > B71.Sg339ss1.BLAST
   ```

5.Peek at the output file and interpret each alignment

   ```
  head -n 50 B71.Sg339ss1.BLAST
   ```


#### Summary of findings:
- Total contigs in assembly: 3161
- Contigs with B71 matches: 1461
- Contigs with no detectable B71 match: 170















###### Outputs of the command line queries of the Sg339ss1 genome


The following queries were answered using command line tools
1. Are there any contigs in the Sg339ss1 genome assembly that have no corresponding matches in the B71 reference genome?
2. Create a list of Sg339ss1 genome contigs that lack matches in the B71 reference
3. Are there any large chromosome segments in the B71 reference genome that are missing in Sg339ss1 genome?

## To check if any contigs in Sg339ss1 assembly don't have corresponding matches in the B71 reference genome
   

1. Get all contigs in the assembly
   ```
 grep ">" Sg339ss1_final.fasta | sed 's/>//' | cut -d' ' -f1 | sort > all_contigs.txt

   ```
2. Extract contigs that have BLAST hits

   ```
  grep -v "^#" B71.Sg339ss1.BLAST | cut -f2 | sort | uniq > matched_contigs.txt
   ```
3. Find contigs with no matches:

   ```
   comm -23 all_contigs.txt matched_contigs.txt > no_match_contigs.txt
   ```

5.Confirm findings by counting no match contigs

   ```
 wc -l no_match_contigs.txt
   ```

#OUTPUT
1700 no_match_contigs.txt


#### To create a list of Sg339ss1 genome contigs that lack matches in the B71 reference

1.Display all contigs
   ```
 cat no_match_contigs.txt
 less no_match_contigs.txt

   ```
2. Copy to a new file Sg339ss1_no_B71_hits.txt

   ```
  cp no_match_contigs.txt Sg339ss1_no_B71_hits.txt
   ```
3. Find contigs with no matches:

   ```
   comm -23 all_contigs.txt matched_contigs.txt > no_match_contigs.txt
   ```

5.Confirm findings by counting no match contigs

   ```
 wc -l no_match_contigs.txt
   ```




