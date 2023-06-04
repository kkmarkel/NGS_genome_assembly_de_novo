# 1 assembly de novo
## 1.1 prepare environment for assembly de novo
```bash
mkdir genome_assembly
mkdir genome_assembly/k39
mkdir genome_assembly/k49
mkdir genome_assembly/k63
mkdir genome_assembly/spades

conda create -n genome_assembly
conda activate genome_assembly
conda install mamba
```
## 1.2 run spades
```bash
mamba install -c bioconda spades # tested with 3.15.5
spades.py --careful -1 7_S4_L001_R1_001.fastq -2 7_S4_L001_R2_001.fastq -o genome_assembly/spades
```
## 1.3 run platanus with start k-mer value 39
Download platanus from http://platanus.bio.titech.ac.jp/platanus-assembler/platanus-1-2-4
```bash
./platanus assemble -s 0 -k 39 -o genome_assembly/k39/platanus_k39 -f 7_S4_L001_R1_001.fastq 7_S4_L001_R2_001.fastq
```
## 1.4 run platanus with start k-mer value 49
```bash
./platanus assemble -s 0 -k 49 -o genome_assembly/k49/platanus_k49 -f 7_S4_L001_R1_001.fastq 7_S4_L001_R2_001.fastq
```
## 1.5 run platanus with start k-mer value 63
```bash
./platanus assemble -s 0 -k 63 -o genome_assembly/k63/platanus_k63 -f 7_S4_L001_R1_001.fastq 7_S4_L001_R2_001.fastq
```
# 2 genome assembly quality control
## 2.1 prepare environment for assembly quality control
```bash
cd genome_assembly
mkdir assembly_analysis
mamba install -c bioconda quast python # tested with quast v.5.2.0 which needed python v.3.10.* - tested with v.3.10.11
```
## 2.2 run quast
```bash
quast.py -o assembly_analysis/platanus_spades -m 0 --threads 1 k39/platanus_k39_contig.fa k49/platanus_k49_contig.fa k63/platanus_k63_contig.fa spades/scaffolds.fasta
```
## 2.3 prepare environment for reads mapping
```bash
mkdir index
mkdir align
```
## 2.4 generate index for mapping
bowtie2-build k39/platanus_k39_contig.fa index/platanus_k39
bowtie2-build k49/platanus_k49_contig.fa index/platanus_k49
bowtie2-build k63/platanus_k63_contig.fa index/platanus_k63
bowtie2-build spades/scaffolds.fasta index/spades
## 2.5 mapping reads on assembly results
bowtie2 -x index/platanus_k39 -1 /home/artem/NGSCourse/2020/assembly/reads/7_S4_L001_R1_001.fastq -2 /home/artem/NGSCourse/2020/assembly/reads/7_S4_L001_R2_001.fastq -S align/platanus_k39.sam
bowtie2 -x index/platanus_k49 -1 /home/artem/NGSCourse/2020/assembly/reads/7_S4_L001_R1_001.fastq -2 /home/artem/NGSCourse/2020/assembly/reads/7_S4_L001_R2_001.fastq -S align/platanus_k49.sam
bowtie2 -x index/platanus_k63 -1 /home/artem/NGSCourse/2020/assembly/reads/7_S4_L001_R1_001.fastq -2 /home/artem/NGSCourse/2020/assembly/reads/7_S4_L001_R2_001.fastq -S align/platanus_k63.sam
bowtie2 -x index/spades -1 /home/artem/NGSCourse/2020/assembly/reads/7_S4_L001_R1_001.fastq -2 /home/artem/NGSCourse/2020/assembly/reads/7_S4_L001_R2_001.fastq -S align/spades.sam
## 2.6 prepare environment for ALE
mkdir ALE_results
## 2.7 run ALE on platanus results with parameter k=39
/home/artem/soft/ALE-master/ALE-master/src/ALE align/platanus_k39.sam k39/platanus_k39_contig.fa ALE_results/platanus_k39.ale
## 2.8 run ALE on platanus results with parameter k=49
/home/artem/soft/ALE-master/ALE-master/src/ALE align/platanus_k49.sam k49/platanus_k49_contig.fa ALE_results/platanus_k49.ale
## 2.9 run ALE on platanus results with parameter k=63
/home/artem/soft/ALE-master/ALE-master/src/ALE align/platanus_k63.sam k63/platanus_k63_contig.fa ALE_results/platanus_k63.ale
## 2.10 run ALE on spades results
/home/artem/soft/ALE-master/ALE-master/src/ALE align/spades.sam spades/scaffolds.fasta ALE_results/spades.ale

Try to improve assembly by using platanus assembler. Send me quast results, ale results and used platanus parameters for additional assemblies by email or telegram.
My email: artem.kasianov@gmail.com