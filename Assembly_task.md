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
```bash
mamba install -c bioconda bowtie2 # tested with 2.5.1
bowtie2-build k39/platanus_k39_contig.fa index/platanus_k39
bowtie2-build k49/platanus_k49_contig.fa index/platanus_k49
bowtie2-build k63/platanus_k63_contig.fa index/platanus_k63
bowtie2-build spades/scaffolds.fasta index/spades
```
## 2.5 mapping reads on assembly results
```bash
bowtie2 -x index/platanus_k39 -1 ../7_S4_L001_R1_001.fastq -2 ../7_S4_L001_R2_001.fastq -S align/platanus_k39.sam #88.02% overall alignment rate
bowtie2 -x index/platanus_k49 -1 ../7_S4_L001_R1_001.fastq -2 ../7_S4_L001_R2_001.fastq -S align/platanus_k49.sam #89.38% overall alignment rate
bowtie2 -x index/platanus_k63 -1 ../7_S4_L001_R1_001.fastq -2 ../7_S4_L001_R2_001.fastq -S align/platanus_k63.sam #88.60% overall alignment rate
bowtie2 -x index/spades -1 ../7_S4_L001_R1_001.fastq -2 ../7_S4_L001_R2_001.fastq -S align/spades.sam #90.81% overall alignment rate
```
## 2.6 prepare environment for ALE  
Here is the [documentation](https://portal.nersc.gov/dna/RD/Adv-Seq/ALE-doc/) for ALE
```bash
mkdir ALE_results
```
## 2.7 run ALE on platanus results with parameter k=39
```bash
git clone https://github.com/sc932/ALE.git # get latest ALE 
cd ALE/src
make # build ALE
cd ../../genome_assembly
../ALE/src/ALE align/platanus_k39.sam k39/platanus_k39_contig.fa ALE_results/platanus_k39.ale #Total ALE Score: -2308412.023487
```
## 2.8 run ALE on platanus results with parameter k=49
```bash
../ALE/src/ALE align/platanus_k49.sam k49/platanus_k49_contig.fa ALE_results/platanus_k49.ale #Total ALE Score: -2047680.789728
```
## 2.9 run ALE on platanus results with parameter k=63
```bash
../ALE/src/ALE align/platanus_k63.sam k63/platanus_k63_contig.fa ALE_results/platanus_k63.ale #Total ALE Score: -2104810.141290
```
## 2.10 run ALE on spades results
```bash
../ALE/src/ALE align/spades.sam spades/scaffolds.fasta ALE_results/spades.ale #Total ALE Score: -2301561.952032
```
**Try to improve assembly by using platanus assembler. Send me quast results, ale results and used platanus parameters for additional assemblies by email or telegram.**
My email: artem.kasianov@gmail.com
# improving assembly using platanus assembler
Change parameters?

Сначала решила запустить platanus с дефолтным значением initial k-mer size (-k) = 32
```bash
./platanus assemble -s 0 -k 32 -o genome_assembly/k32/platanus_k32 -f 7_S4_L001_R1_001.fastq 7_S4_L001_R2_001.fastq
```
Соберём статистику
```bash
quast.py -o assembly_analysis/platanus_spades -m 0 --threads 1 k32/platanus_k32_contig.fa k39/platanus_k39_contig.fa k49/platanus_k49_contig.fa k63/platanus_k63_contig.fa spades/scaffolds.fasta
```
Получилось увеличить N50 (до 335), total length (до 8199) - хотя всё равно меньше, чем в SPAdes; уменьшить количество контигов до 41.  
Вообще, судя по статьям, SPAdes может собирать лучше, но при этом имеет большую вероятность ошибок, чем platanus (т.е. platanus более аккуратный)  

Решила остановиться на k32 и попробовать сменить другие параметры.  
step size of k-mer extension - сменим на 10, а то раньше пробовали вообще с 0 (а вот в [Platanus-allee](http://platanus.bio.titech.ac.jp/platanus2) вообще сделали дефолтными 20)  
```bash

./platanus assemble -s 10 -k 32 -o genome_assembly/k32_s10/platanus_k32_s10 -f 7_S4_L001_R1_001.fastq 7_S4_L001_R2_001.fastq
quast.py -o assembly_analysis/platanus_spades -m 0 --threads 1 k32_s10/platanus_k32_s10_contig.fa k32/platanus_k32_contig.fa k39/platanus_k39_contig.fa k49/platanus_k49_contig.fa k63/platanus_k63_contig.fa spades/scaffolds.fasta
```
=> N50 (=489) и количество контигов (=14) стали классными, а вот total length поплохела (=5830). C

Для вируса гриппа ожидаем total length в районе 13500bp??

Попробуем вручную проставить initial k-mer coverage cutoff (флаг -n). Выбрала значение 14, потому что это половина от выставляемой platanus автоматически (вдохновлялась статьёй про [Platanus-allee](http://platanus.bio.titech.ac.jp/platanus2)).
```bash
./platanus assemble -s 0 -k 32 -n 14 -o genome_assembly/k32_n14/platanus_k32_n14 -f 7_S4_L001_R1_001.fastq 7_S4_L001_R2_001.fastq
quast.py -o assembly_analysis/platanus_spades -m 0 --threads 1 k32_n14/platanus_k32_n14_contig.fa k32_s10/platanus_k32_s10_contig.fa k32/platanus_k32_contig.fa k39/platanus_k39_contig.fa k49/platanus_k49_contig.fa k63/platanus_k63_contig.fa spades/scaffolds.fasta
```
Получился по сути эффект обратный повышению step size of k-mer extension - total length выросла, а N50 сильно понизилось, количество контигов сильно выросло (=82, это не дело).

А если объединим?  
```bash
./platanus assemble -s 10 -k 32 -n 14 -o genome_assembly/k32_s10_n14/platanus_k32_s10_n14 -f 7_S4_L001_R1_001.fastq 7_S4_L001_R2_001.fastq
quast.py -o assembly_analysis/platanus_spades -m 0 --threads 1 k32_s10_n14/platanus_k32_s10_n14_contig.fa k32_n14/platanus_k32_n14_contig.fa k32_s10/platanus_k32_s10_contig.fa k32/platanus_k32_contig.fa k39/platanus_k39_contig.fa k49/platanus_k49_contig.fa k63/platanus_k63_contig.fa spades/scaffolds.fasta
```
Воот, вот это мне больше нравится по N50 (=434), контигам (n=20), правда, total length получилась неидеальная - 7566bp. Есть, куда улучшать.

Возможно можно как-то увеличить качество ридов? Пробовала триммить с помощью Trim-galore, но особо не помогло (возможно, нужны какие-то специфические тулы? Musket, например - https://musket.sourceforge.net/homepage.htm)  

Прочитала статью https://www.nature.com/articles/s41467-019-09575-2 и скачала [Platanus-allee](http://platanus.bio.titech.ac.jp/platanus2)
```bash
tar zxvf Platanus_allee_v2.2.2_Linux_x86_64.tgz
./platanus_allee assemble -k 32 -o ../genome_assembly/k32/platanus_allee_k32 -f ../7_S4_L001_R1_001.fastq ../7_S4_L001_R2_001.fastq

quast.py -o assembly_analysis/platanus_spades -m 0 --threads 2 k32/platanus_allee_k32_contig.fa k32_s10_n14/platanus_k32_s10_n14_contig.fa k32_n14/platanus_k32_n14_contig.fa k32_s10/platanus_k32_s10_contig.fa k32/platanus_k32_contig.fa k39/platanus_k39_contig.fa k49/platanus_k49_contig.fa k63/platanus_k63_contig.fa spades/scaffolds.fasta
```
=> очень плохо, скорее всего, потому что больше подходит для диплоидных (?), но пусть будет.

## дозапустим bowtie и ALE
```bash
bowtie2-build k32/platanus_k32_contig.fa index/platanus_k32
bowtie2 -x index/platanus_k32 -1 ../7_S4_L001_R1_001.fastq -2 ../7_S4_L001_R2_001.fastq -S align/platanus_k32.sam # 87.43% overall alignment rate
../ALE/src/ALE align/platanus_k32.sam k32/platanus_k32_contig.fa ALE_results/platanus_k32.ale # Total ALE Score: -2415113.221947

bowtie2-build k32_n14/platanus_k32_n14_contig.fa index/platanus_k32_n14
bowtie2 -x index/platanus_k32_n14 -1 ../7_S4_L001_R1_001.fastq -2 ../7_S4_L001_R2_001.fastq -S align/platanus_k32_n14.sam #80.92% overall alignment rate
../ALE/src/ALE align/platanus_k32_n14.sam k32_n14/platanus_k32_n14_contig.fa ALE_results/platanus_k32_n14.ale # Total ALE Score: -3842622.858044

bowtie2-build k32_s10/platanus_k32_s10_contig.fa index/platanus_k32_s10
bowtie2 -x index/platanus_k32_s10 -1 ../7_S4_L001_R1_001.fastq -2 ../7_S4_L001_R2_001.fastq -S align/platanus_k32_s10.sam #87.27% overall alignment rate
../ALE/src/ALE align/platanus_k32_s10.sam k32_s10/platanus_k32_s10_contig.fa ALE_results/platanus_k32_s10.ale # Total ALE Score: -2122450.542837

bowtie2-build k32_s10_n14/platanus_k32_s10_n14_contig.fa index/platanus_k32_s10_n14
bowtie2 -x index/platanus_k32_s10_n14 -1 ../7_S4_L001_R1_001.fastq -2 ../7_S4_L001_R2_001.fastq -S align/platanus_k32_s10_n14.sam #89.68% overall alignment rate
../ALE/src/ALE align/platanus_k32_s10_n14.sam k32_s10_n14/platanus_k32_s10_n14_contig.fa ALE_results/platanus_k32_s10_n14.ale # Total ALE Score: -1889963.774072

bowtie2-build k32/platanus_allee_k32_contig.fa index/platanus_allee_k32
bowtie2 -x index/platanus_allee_k32 -1 ../7_S4_L001_R1_001.fastq -2 ../7_S4_L001_R2_001.fastq -S align/platanus_allee_k32.sam #62.16% overall alignment rate
../ALE/src/ALE align/platanus_allee_k32.sam k32/platanus_allee_k32_contig.fa ALE_results/platanus_allee_k32.ale # Total ALE Score: -7767413.465383
```
В итоге, получилось, что в тройку лучших входят сборки **platanus_k32_s10_n14**, platanus_k32 и SPAdes. Хотя, возможно, platanus надо как-то ещё подрегулировать, чтобы получить total length побольше.  