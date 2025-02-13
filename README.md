![logo_dark_white](https://user-images.githubusercontent.com/12270542/136858030-a9df2ec0-90af-472e-91f5-b7a224dd7289.png)


Stitch together Nanopore tiled amplicon data using a reference guided approach

Tiled amplicon data, like those produced from primers designed with [primal scheme](https://github.com/aresti/primalscheme), are typically assembled using methods that involve aligning them to a reference and polishing the reference to obtain a consensus of the reads based on the reference. This works very well for obtaining a genome with SNPs and small indels representative of the reads. However in cases where the reads cannot be mapped well to the reference (e.g. genomes containing hypervariable regions between primers) or in cases where large structrual variants are expected this method may fail as polishing tools expect the reference to originate from the reads.

Lilo uses a reference only to assign reads to the amplicon they originated from and to order and orient the polished amplicons, no reference sequence is incorporated into the final assembly. Once assigned to an amplicon, a read with high average base quality of roughly median length for that amplicon is selected as a reference and polished with up to 300x coverage three times with medaka. The polished amplicons have primers removed with porechop (fork: https://github.com/sclamons/Porechop-1) and are then assembled with [CAP3](https://faculty.sites.iastate.edu/xqhuang/cap3-and-pcap-sequence-and-genome-assembly-programs). 

Lilo has been tested on SARS-CoV-2 with artic V3 primers. It has also been tested on 7kb and 4kb amplicons with ~100-1000bp overlaps for ASFV, PRRSV-1 and PRRSV-2, schemes for which will be made available in the near future.

LILO-CaPV, has been tested on 7.5kb amplicons with ~1 kb overlaps for CaPV (as described by [Mathijs et al., 2022](https://www.sciencedirect.com/science/article/pii/S0166093422000118)).

## Requirments
Install Conda :)   

## Installation
```
git clone https://github.com/el-mat/LILO-CaPV.git
cd LILO-CaPV
conda env create --file LILO-CaPV.yaml 
conda activate LILO-CaPV
pip3 install git+https://github.com/sclamons/Porechop-1.git
```

## Usage
Lilo assumes your reads are in a folder called *raw/* in the current working directory and have the suffix *.fastq.gz.*. Multiple samples can be processed at the same time.  
Lilo requires a config file detailing the location of a reference, a primer scheme (in the form of a primal scheme style bed file), and a primers.csv file (described below).
```
conda activate LILO-CaPV
snakemake -k -s /path/to/LILO-CaPV --configfile /path/to/config.file --cores N
```
It is recommended to run with -k so that one sample with insufficient coverage will not stop the other jobs completing. N should be adapted to the amount of threads available for the analysis.

A folder called *raw/* containing a CaPV dataset *test.fastq.gz* is provided to allow you to test the installation of the Lilo pipeline. The pipeline should generate a ~150 kb contig in *test/polished_trimmed.fa.cap.contigs*

## Input specifications
* **config.file**: an example config file has been provided.  
* **Primer scheme**: As output by primal scheme, **with alt primers removed**. Bed file of primer alignment locations. Columns: reference name, primer start, primer end, primer name, pool (must end with 1 or 2).  
* **Primers.csv**: Comma delimited, includes alt primers, **with header line**. Columns: amplicon_name, F_primer_name, F_primer_sequence, R_primer_name, R_primer_sequence. If there are a lot of degenerate bases in any of the primers it is recommended to expand these, the script expand.py will expand the described csv into a longer csv with IUPAC codes expanded.
* **reference.fasta** Same reference used to make the scheme file.

## Output
Lilo uses the names from raw/ to name the output file. For a file named "test.fastq.gz", files produced during the pipeline will be in a folder called "test", and the final assembly will be named "polished_trimmed.fa.cap.contigs". The output will contain amplicons that had at least 40X full length coverage. Missing amplicons will be represented by Ns. Any ambiguity at overlaps will be indicated with IUPAC codes.

## Note
* Use of the wrong fork for porechop will cause the pipeline to fail.  
* Currently Lilo only works on genomes with a single chromosome, but the edit to fix this is relatively simple and I will get to it.
* Lilo is a work in progress and has been tested on a limited number of references, amplicon sizes, and overlap sizes, I recommend checking the results carefully for each new scheme.    
* The pipeline currently assumes that any structural variants are contained between the primers of an amplicon and do not change the length of the amplicon by more than 5%. If alt amplicons produce a product of a different length to the original amplicon they may not be allocated to their amplicon. Editing it to work better with alt amplicons is on my to do list.  
* Should not be used with reads produced with rapid kits, the pipeline assumes the reads are the length of the amplicons.
* Do let me know if it destroys any cities or steals everyone's left shoe.
