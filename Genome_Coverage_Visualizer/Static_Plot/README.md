# Coverage Plot for Metagenomics

## About:
1. The script of "MiCoP_MakeCoveragePlot.sh" takes a single ".bam" file that is the output of "bwa-mem -a".
2. It extracts the unique reads that are mapped once and only once to the entire genome database.
3. It sorts the unique read list based on the genome.
4. For each genome, it takes the unique reads and builds coverage information and then generates the coverage plot.
5. The coverage information includes the genome name, bp index, and number of reads that are mapped to the bp located at the bp index.
6. All the coverage plots will be saved in "CoveragePlots" folder in the same working directory.

## Step 1: Prepare reference database:
1. The reference database contains many contigs for each organism. If you are interested in mapping at organism-level rather than contig-level, you need to pre-process the reference database.
2. The code below prepares the database (e.g., fungi.fa), such that for each organism, it concatenates all contigs and generates a single fasta output. It also updates the organism name and the length of the concatenated contigs in the header field of each organism. 
```
$ grep '>' fungi.fa | awk -F "|" '{print $2}' | uniq > fungi_RefList.txt
$ python ConcatContigs.py fungi_RefList.txt fungi.fa > fungi_ConcatContigs.fa
```
To execute the algorithm on UCLA-Huffman2 cluster:
```
$ cd /u/project/zarlab/malser/MiCoP/Scripts
$ qsub submit-Concat-Contigs.sh
```
Or execute the following commands:
```
$ cd /u/project/zarlab/malser/MiCoP/Scripts
$ grep '>' /u/project/zarlab/serghei/eupathdb/eupathdbFasta/ameoba.fa | awk -F "|" '{print $2}' | uniq > /u/project/zarlab/malser/MiCoP/eupathdbFasta_ConcatContigs/ameoba_RefList.txt
$ module load python/3.6.1
$ python3 ConcatContigs.py /u/project/zarlab/malser/MiCoP/eupathdbFasta_ConcatContigs/ameoba_RefList.txt /u/project/zarlab/serghei/eupathdb/eupathdbFasta/ameoba.fa > /u/project/zarlab/malser/MiCoP/eupathdbFasta_ConcatContigs/ameoba_ConcatContigs.fa
```
The output concatenated contigs of each organism can be found at:
/u/project/zarlab/malser/MiCoP/eupathdbFasta_ConcatContigs

## Step 2: BWA-MEM Mapping:
1. Build Index of the reference.
2. Align the read set to the reference database and Remove the reads that has no reference name in the third column.
3. Sort the outputbased on the read ID and the reference Name.
```
$ bwa index fungi_ConcatContigs.fa
$ bwa mem -a fungi_ConcatContigs.fa /u/home/galaxy/collaboratory/serghei/MetaSUB-Inter-City-Challenge/data/SRR3546361.fastq | awk '$3!="*"' > SRR3546361_MergedContigs_Filtered.sam
$ sort -t$'\t' -k 1,1 -V -k 3,3 SRR3546361_MergedContigs_Filtered.sam > SRR3546361_MergedContigs_Sorted.sam
```
## Step 3: SAM to BAM [optional]:
Convert .sam to .bam to maintain a compressed data for efficient storage.
```
$ samtools view -bS SRR3546361_MergedContigs_Sorted.sam > SRR3546361_MergedContigs_Sorted.bam
```
## Step 4: Extract Unique Reads:
1. Extract the name and length of each reference in the database.
2. Extract the Unique Reads.
3. Build the Coverage Plot (it takes the plot window size and the read classifier mode).
```
$ grep '>' fungi_ConcatContigs.fa > GenomeInformation.txt
$ python ReadClassifier.py SRR3546361_MergedContigs_Sorted.sam 1 > Read_FullList.sam
$ sort -t$'\t' -V -k 3,3 Read_FullList.sam > Read_FullList_Sorted.sam
$ python CoveragePlot.py Read_FullList_Sorted.sam GenomeInformation.txt 100 1
```
This command also extracts the uniquereads.
```
$ samtools view SRR3546361_MergedContigs_Sorted.bam | awk 'BEGIN { FS="\t" } { c[$1]++; l[$1,c[$1]]=$0 } END { for (i in c) { if (c[i] == 1) for (j = 1; j <= c[i]; j++) print l[i,j] } }' | sort -t$'\t' -k 3,3 -V -k 1,1 > Read_FullList.sam
```
![alt text](https://github.com/smangul1/miCoP/blob/master/Genome_Coverage_Visualizer/Static_Plot/Malassezia_globosa_CBS_7966.png)

## Step 5: Extract MultiMapped Reads (within-genome):
1. Extract the name and length of each reference in the database.
2. Extract the Unique Reads.
3. Build the Coverage Plot (it takes the plot window size and the read classifier mode).
```
$ grep '>' fungi_ConcatContigs.fa > GenomeInformation.txt
$ python ReadClassifier.py SRR3546361_MergedContigs_Sorted.sam 2 > Read_FullList.sam
$ sort -t$'\t' -V -k 3,3 Read_FullList.sam > Read_FullList_Sorted.sam
$ python CoveragePlot.py Read_FullList_Sorted.sam GenomeInformation.txt 100 2
```
![alt text](https://github.com/smangul1/miCoP/blob/master/Genome_Coverage_Visualizer/Static_Plot/Malassezia_globosa_CBS_7966_MM.png)

By Mohammed Alser
