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

