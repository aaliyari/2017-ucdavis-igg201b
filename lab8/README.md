# Lab 8 - RNAseq expression analysis, revisited

Learning objectives:

1. Dig into some statistical thoughts.

2. Think about the pipeline, and maybe the github.

## Running a slightly larger RNAseq analysis than before.

0. Start up a new cloud instance (as usual; see [lab 5](../lab5/README.md)).
   (m4.large is probably fine for this.)

1. Connect to Terminal. Run `bash`.

2. Install edgeR using [this script](https://github.com/ctb/2017-ucdavis-igg201b/blob/master/lab7/install-edgeR.R):

        cd
        git clone https://github.com/ctb/2017-ucdavis-igg201b.git

        sudo Rscript --no-save ~/2017-ucdavis-igg201b/lab7/install-edgeR.R

3. Install [salmon](https://salmon.readthedocs.io):

        cd
        curl -L -O https://github.com/COMBINE-lab/salmon/releases/download/v0.8.0/Salmon-0.8.0_linux_x86_64.tar.gz
        tar xzf Salmon-0.8.0_linux_x86_64.tar.gz
        export PATH=$PATH:$HOME/Salmon-latest_linux_x86_64/bin

4. Run:

        mkdir yeast
        cd yeast

5. Download some data from [Schurch et al, 2016](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4878611/):

        curl -O ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR458/ERR458500/ERR458500.fastq.gz
        curl -O ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR458/ERR458501/ERR458501.fastq.gz
        curl -O ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR458/ERR458502/ERR458502.fastq.gz

        curl -O ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR458/ERR458493/ERR458493.fastq.gz
        curl -O ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR458/ERR458494/ERR458494.fastq.gz
        curl -O ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR458/ERR458495/ERR458495.fastq.gz

6. Download the yeast reference transcriptome:

        curl -O http://downloads.yeastgenome.org/sequence/S288C_reference/orf_dna/orf_coding.fasta.gz

7. Index the yeast transcriptome:

        salmon index --index yeast_orfs --type quasi --transcripts orf_coding.fasta.gz

8. Run salmon on all the samples:

        for i in *.fastq.gz
        do
            salmon quant -i yeast_orfs --libType U -r $i -o $i.quant --seqBias --gcBias
        done

   What do you think all this stuff with the bias is about?

   Read up on [libtype, here](https://salmon.readthedocs.io/en/latest/salmon.html#what-s-this-libtype).

9. Collect all of the sample counts using [this Python script](https://github.com/ngs-docs/2016-aug-nonmodel-rnaseq/blob/master/files/gather-counts.py):

        curl -L -O https://github.com/ngs-docs/2016-aug-nonmodel-rnaseq/raw/master/files/gather-counts.py
        python2 gather-counts.py


10. Run edgeR (in R) using [this script](https://github.com/ctb/2017-ucdavis-igg201b/blob/master/lab8/yeast.salmon.R) and take a look at the output:

        Rscript --no-save ~/2017-ucdavis-igg201b/lab8/yeast.salmon.R

   (Note that this script has been updated from lab 7 to add two new samples.)

   This will produce two plots, `yeast-edgeR-MA-plot.pdf` and
   `yeast-edgeR-MDS.pdf`. You can view them by going to your Jupyter
   console and looking in the directory `yeast`.

   The `yeast-edgeR.csv` file contains the fold expression & significance information in a spreadsheet.

## Questions to ask/address

1. What is the point or value of the [multidimensional scaling (MDS)](https://en.wikipedia.org/wiki/Multidimensional_scaling) plot?

2. Why does the MA-plot have that shape?

   Related: Why can't we just use fold expression to select the things we're interested in?

   Related: How do we pick the FDR (false discovery rate) threshold?

3. How do we know how many replicates (bio and/or technical) to do?

   Related: what confounding factors are there for RNAseq analysis?

   Related: what is our false positive/false negative rate?

## More reading

"How many biological replicates are needed in an RNA-seq experiment and which differential expression tool should you use?" [Schurch et al., 2016](http://rnajournal.cshlp.org/content/22/6/839).

"Salmon provides accurate, fast, and bias-aware transcript expression estimates using dual-phase inference" [Patro et al., 2016](http://biorxiv.org/content/early/2016/08/30/021592).

Also see [seqanswers](http://seqanswers.com/) and [biostars](https://www.biostars.org/).
