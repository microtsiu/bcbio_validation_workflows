# bcbio validation workflows

[bcbio](http://bcb.io) validation workflows in [Common Workflow Language (CWL)](http://www.commonwl.org/)
generated using [bcbio's CWL support](http://bcbio-nextgen.readthedocs.io/en/latest/contents/cwl.html).
These aim to be compatible with multiple CWL runners.

## Available workflows

### Joint calling validation workflow with Genome in a Bottle samples

`giab-joint` -- Run a germline [GATK4 joint variant calling
workflow](https://software.broadinstitute.org/gatk/) using three diverse Genome
in a Bottle samples from different sequencing technologies:

- NA12878: (Caucasian daughter): [65x NovaSeq TruSeq Nano](https://basespace.illumina.com/datacentral)
- NA24385 (Ashkenazi Jewish son): [50x HiSeq x10 dataset from 10x genomics](https://support.10xgenomics.com/de-novo-assembly/datasets)
- NA24631 (Chinese son) -- [55x Illumina HiSeq 2500 2x250bp](http://bit.ly/NA24631-readme)

Ths original whole genome inputs were subset to exome regions and chr20, maintaining
parallelization and whole genome challenges while trying to avoid long runtimes.

This calls each independently using GATK HaplotypeCaller, generating gVCF as input
to a combined joint callset.

### Synthetic Diploid CHM and Genome in a Bottle

`giab-chm` -- This validation expands on the Genome in a Bottle datasets to include
[a synthetic diploid input](https://gatkforums.broadinstitute.org/gatk/discussion/10912/what-is-truth-or-how-an-accident-of-nature-can-illuminate-our-path)
derived from two haploid cell lines from Complete Hydatidiform Moles (CHM).
This truth set uses PacBio sequencing and is orthogonal to the Genome in a
Bottle Truth sets, helping identify short read and Illumina specific bias in those
inputs.

We subset the [initial read input from ERA596361](ftp://ftp.sra.ebi.ac.uk/vol1/ERA596/ERA596361/bam/CHM1_CHM13_2.bam)
to exome regions and chromosome 20. The truth set is prepared from
the [version 0.2 20161018 tarball](https://github.com/lh3/CHM-eval/releases/download/v0.2/CHM-evalkit-20161018.tar)
by removing polyA10 regions and regions within 10bp of problematic PacBio indels
(1bp and >50bp) using the `prep_chm_truth.py` script. The inputs are in the same
synapse project as the Genome in a Bottle joint calling validation above.

### Somatic Genome in a Bottle mixture

`somatic-giab-mix` -- Somatic tumor/normal variant calling on [a sequenced
mixture dataset of two Genome in a Bottle
samples](ftp://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/use_cases/mixtures/UMCUTRECHT_NA12878_NA24385_mixture_10052016/)
simulating a lower frequency set of calls. It has a 90x tumor genome consisting
of 30% NA12878 (tumor) and 70% NA24385 (germline) and a 30x normal genome of
NA24385. Unique NA12878 variants are somatic variations at 15% and 30%.
Since this has well resolved truth sets and is non-synthetic, it provides a
minimum baseline for calling lower frequency somatic variant. The HiSeq x10
sequenced data also has some 3' quality issues which help identify false
positive and long runtime issues in different calling methods.

The dataset is subset to chr20 and exome regions, similar to the `giab-joint`
and `giab-chm` examples above.

### NA12878 chromosome 20 for GA4GH-DREAM tool execution challenge

`NA12878-chr20` -- A germline variant calling workflow running a single chromosome subset of the
[Genome in a Bottle NA12878 sample](http://jimb.stanford.edu/giab) from
[Illumina's Platinum Genomes](https://www.illumina.com/platinumgenomes.html).
It runs alignment, variant calling with multiple methods (
[GATK4 HaplotypeCaller](http://gatkforums.broadinstitute.org/gatk/categories/gatk-4-alpha)
[FreeBayes](https://github.com/ekg/freebayes),
[Platypus](https://github.com/andyrimmer/Platypus),
[samtools](https://github.com/samtools/samtools)) and quality control,
validating outputs against reference standards. It runs in ~4 hours on a 8
core machine, using ~50Gb of disk space (35Gb for inputs, 15Gb for analyses).

The NA12878 chr20 example is part of the
[GA4GH-DREAM tool execution challenge](https://www.synapse.org/#!Synapse:syn8507134/wiki/416001)
and meant to be easily run on multiple CWL runners. The
[Synapse bcbio workflow directory](https://www.synapse.org/#!Synapse:syn9725771)
mirrors the CWL in this repository and also contains the biological data to run
the workflow.

The data for this run is self-contained within synapse:

        synapse get -r syn9725771

### Workflows in progress

- `NA24385-sv` -- Structural variant calling on Genome in a Bottle NA24385 (HG002) Ashkenazi
  sample, compared against [the v0.5.0 combined validation set](ftp://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/AshkenazimTrio/analysis/NIST_UnionSVs_12122017/).

- `pgp` -- Variant, HLA and structural variant calling on [Personal Genome
  Project](http://www.personalgenomes.org/us) genomes using the [Arvados public
  cloud](https://workbench.su92l.arvadosapi.com/).

- `SGDP-recall-CGC` -- Germline recalling on the
  [Cancer Genomics Cloud](http://www.cancergenomicscloud.org/) using the public
  [Simons Genome Diversity Project Dataset](https://www.simonsfoundation.org/life-sciences/simons-genome-diversity-project-dataset/).

## General instructions

### Retrieving data

The [GA4GH-DREAM Workflow Execution
Challenge](https://www.synapse.org/#!Synapse:syn8507133/wiki/415976) hosts the
data for these challenges on [Synapse](https://www.synapse.org) in the bcbio
[biodata](https://www.synapse.org/#!Synapse:syn10468187) folder.

Use the [Synapse python
client](https://github.com/Sage-Bionetworks/synapsePythonClient#installation) to
download the data. You can install this yourself via pip or conda:

    conda install -c conda-forge -c bioconda synapseclient

or it's included with a bcbio-vm installation.

Download requires a free account on [Synapse](https://www.synapse.org) to obtain
access. First login to Synapse with your credentials:

    syanpse login

and you can retrieve the full biodata folder with sample data for multiple
validations with:

    synapse get -r syn10468187

or use the `download_data.sh` shell script link in each validation to get only
the data for that run.

### Running

Each validation contains a ready to run Common Workflow Language description of
the workflow, along with a description of samples pointing to local files from
the downloaded biodata directory. If you're an experienced user of an analysis
platform this is all you need to run an analysis.

If you'd like to explore more or are not sure where to get started, bcbio-vm
contains wrappers and automated tools to help with regenerating CWL from input
YAML files and running this CWL on different plaforms. The [bcbio CWL
documentation](http://bcbio-nextgen.readthedocs.io/en/latest/contents/cwl.html)
contains details about installing and running on each platform, but in brief,
[Install bcbio-vm](https://github.com/chapmanb/bcbio-nextgen-vm#installation)
with:

    wget https://repo.continuum.io/miniconda/Miniconda2-latest-Linux-x86_64.sh
    bash Miniconda-latest-Linux-x86_64.sh -b -p ~/install/bcbio-vm/anaconda
    ~/install/bcbio-vm/anaconda/bin/conda install --yes -c conda-forge -c bioconda bcbio-nextgen-vm
    export PATH=~/install/bcbio-vm/anaconda/bin:$PATH

The individual tool run directories contains starter shell scripts for different
CWL-enabled tools like [Toil](https://github.com/BD2KGenomics/toil) or and
[rabix bunny](https://github.com/rabix/bunny). Run with these can either use
a local bcbio installation of tools or [bcbio Docker
containers](https://github.com/bcbio/bcbio_docker).

### Adjusting resources

You can adjust `bcbio_system.yaml`in each directory to fit a pipeline run
to your current system. This includes options for changing cores, memory usage
and locations of input files. After changing, use `run_generate_cwl.sh` to
create a new CWL matching your input specifications.
