# Lab 3

2/3/17, Daniel Standage, daniel.standage@gmail.com

Follows from [Lab 2](../lab2/README.md).

Learning objectives:

* understand the output of variant calling / VCF files.

* compare using roll-your-own Python code vs Python libraries vs command-line programs for analyzing variant locations.

## Boot up an Amazon instance

Goal: provide a platform to run stuff on.

1. Boot an AWS m4.large, running image ami-c72d7fa7; you can follow [these instructions](https://2016-feb-aws.readthedocs.io/boot.html).

2. Edit the security group "inbound rules" so that ports 22 and 8000
   are open to all. You can follow [these instructions](https://2016-feb-aws.readthedocs.io/configure-firewall.html).

3. Connect to your machine's Public IP at port 8000 in a Web browser, e.g.
   `http://52.53.yyy.xxx:8000`.

4. Start a `New...` `Terminal`, and then run `bash`.

5. Run:

        git clone https://github.com/ctb/2017-ucdavis-igg201b.git

6. Install samtools:

        sudo apt-get -y install samtools

## Download data

Goal: get the outputs from lab 2, so we don't need to run all that stuff again!

1. Run:

        mkdir ~/work
        cd ~/work
        curl -O https://s3-us-west-1.amazonaws.com/dib-training.ucdavis.edu/2017-ucdavis-igg201b/SRR2584857-bam%2Bvcf.tar.gz

2. Unpack the files:

        tar -xzf SRR2584857-bam%2Bvcf.tar.gz

3. Verify that you can run 'samtools tview':

        samtools tview SRR2584857.sorted.bam ecoli-rel606.fa

   (Use 'q' to exit the viewer)

## Look at the VCF file with Python.

The [official VCF specification](https://samtools.github.io/hts-specs/VCFv4.1.pdf) is a great read...if you're suffering from insomnia.
Let's skip this and just take a quick look at the file.

1. Look at the non-commented lines:

        grep -v ^# variants.vcf

   The first five columns: `CHROM  POS     ID      REF     ALT`.
   It's a little easier to see if you run

        grep -v ^# variants.vcf | less -S

    Use your left and right arrows to scroll, and 'q' to quit.

2. Examine one of the variants with tview:

        samtools tview SRR2584857.sorted.bam ecoli-rel606.fa -p ecoli:920514

   'q' to quit, left arrow to scroll a bit left.

## Parse the GFF3 file.

The GFF3 format also has a [technical specification](https://github.com/The-Sequence-Ontology/Specifications/blob/master/gff3.md), but we can skip this for now too and just take a look at the data.

1. At the command line, copy in the GFF annotation of E. coli rel 606.

        cp ~/2017-ucdavis-igg201b/lab3/ecoli-rel606.gff.gz .

2. Start a new Python 3 notebook.

3. In a cell, write:

        cd work

   and use 'shift-ENTER' to execute.

3. Run (in a new cell, with Shift-ENTER):

        !gunzip -c ecoli-rel606.gff.gz | head

   Note, here you are executing a shell command from within Jupyter Notebook.

4. Write a short GFF3 parser:

```
import gzip

def read_gff3(filename):
    for line in gzip.open(filename, 'rt', encoding='utf-8'):
        if line.startswith('#'): continue   # GFF comment; skip!
        line = line.split('\t')

        feature_type, start, stop, _, strand, _, info = line[2:9]

        # we want 'gene'
        if feature_type != 'gene':
            continue

        start, stop = int(start), int(stop)
        yield feature_type, start, stop, strand, info
```

5. Try running it:

```
for f, start, stop, strand, info in read_gff3('ecoli-rel606.gff.gz'):
    print(f, start, stop, strand, info)
    break
```

6. Take one of the variants and see if we can find it in a gene:

```
# interrogate a particular position
variant_pos = 3931002

for f, start, stop, strand, info in read_gff3('ecoli-rel606.gff.gz'):
    if variant_pos >= start and variant_pos <= stop:
        print(f, start, stop, strand, info)
```

7. Discuss:

   * What happens if you set `variant_pos = 920514`?
     Can you explain this result?
     Is there anything you could change to get a more interesting result?
   * Efficiency of this approach vs indexing approaches :)

## Parse GFF3 with a library

1. Install the `tag` library.

```
!pip install tag
import tag
```

2. Reproduce the previous task using library commands.

```python
reader = tag.GFF3Reader(infilename='ecoli-rel606.gff.gz')
for gene in tag.select.features(reader, type='gene'):
    if gene.contains_point(3931002):
        print(gene.slug)
```

Most libraries like tag come with some kind of documentation.
The tag documentation is at http://tag.readthedocs.io/en/stable/index.html (API documentation at http://tag.readthedocs.io/en/stable/api.html).

## Look at the VCF file with bedtools.

[bedtools docs](https://bedtools.readthedocs.io/en/latest/)

1. Download and build bedtools:

        cd ~/
        curl -O -L https://github.com/arq5x/bedtools2/releases/download/v2.26.0/bedtools-2.26.0.tar.gz
        tar -xzf bedtools-2.26.0.tar.gz

        cd bedtools2
        make
        sudo make install

2. Go back to work:

        cd ~/work

3. Run bedtools intersect:

        bedtools intersect -a ecoli-rel606.gff.gz -b variants.vcf -wa -u

   [Documentation for bedtools intersect](https://bedtools.readthedocs.io/en/latest/content/tools/intersect.html)

## Compare bedtools and Python.

* Writing your own Python code is flexible and customizable.
* Python libraries can save you lots of work, but can also have a learning curve.
* Bedtools is fast, general, well supported, less likely to be erroneous.
* Use all three :)

## Extract reads with samtools.

1. Execute:

        samtools view SRR2584857.sorted.bam 'ecoli:920514-920514' > out.bam
        wc -l out.bam

and this will give you the coverage of the relevant position.
