---
layout: tutorial_hands_on

title: "EWAS data analysis of 450k data"
zenodo_link: "https://zenodo.org/record/1251211#.WwREQ1Mvz-Y"
enable: "false"
questions:
  - "What is a epigenome-wide association studies?"
  - "Why is a EWAS analysis useful?"
  - "Why 450k data?"
time_estimation: "1h"
objectives:
  - "Learn how to perform reproducible EWAS analysis"
key_points:
  - "EWAS analyse changes in DNA methylation in cohorts of different individuals effected by diseases"
  - "DNA methylation changes can be triggered by environment (i.e. sun exposure, diet) and effect the way how our cells works and in long term cause cancer"
contributors:
  - kpbioteam
  - kpoterlowicz

---

# Introduction
{:.no_toc}

The field of cancer genomics has demonstrated the power of massively parallel sequencing techniques to inform on genes and specific alterations that drive tumor onset and progression. Although large comprehensive sequence data sets continue to be made increasingly available, data analysis remains an ongoing challenge, particularly for laboratories lacking dedicated resources and bioinformatics expertise. To address this, we have provide training based on Galaxy tools EWAS suite that represent many popular algorithms for detecting somatic genetic alterations from cancer genome and exome data.

![epimechanism](../../images/Epigenetic_mechanisms.jpg "How epigenetics mechanism can effect health (adapted from https://commonfund.nih.gov/epigenomics/figure)")

This exercise uses datasets from the Cell publication by Hugo, Willy, et al., 2015 {% cite Hugo2015 %}. with the goal being to identify differentially methylated regions and positions associated with treatment resistant melanomas.
Datasets include the Illumina 450K Methylation array performed in melanoma tumors in a sample of patients pre and post MAPKi and BRAFi treatment with different outcomes (sensitive and resistant). For each sample there is raw green and red colour arrays containing the summarized bead information generated by the Illumina 450k scanner, i.e. there are 8 files in total:


The Infinium Methylation Assay 450k uses two different bead types to detect changes in DNA methylation levels. In the figure we can see M - methylated and U - unmethylated bead types. In our study unmethylated and methylated bead signals are reported as green and red colors respectively.

![methassay](../../images/methylationassay.png "Infinium Methylation Assay Overview (adapted from https://emea.illumina.com/science/technology/beadarray-technology/infinium-methylation-assay.html?langsel=/gb/)")


Accession  | Sensitivity | Treatment
--- | ---  | ---
GSM1588704 | baseline    | pre-treatment
GSM1588705 | baseline    | pre-treatment
GSM1588706 | resistant   | BRAFi
GSM1588707 | resistant   | BRAFi

> ### Agenda
>
> 1. TOC
> {:toc}
>
{: .agenda}

# 450K Array Data Upload

The first step of EWAS data analysis is the raw methylation data loading (intensity information files for each two color micro array)

> ### {% icon hands_on %} Hands-on: Data Loading
>
> 1. Create a new history for this tutorial and give it a proper name
>
>    {% include snippets/create_new_history.md %}
>
> 2. Import the following IDAT files from [Zenodo](https://zenodo.org/record/1251211#.WwREQ1Mvz-Y) or from the data library (ask your instructor)
>    - `GSM1588704_8795207135_R01C02_Red.idat`
>    - `GSM1588705_8795207119_R05C02_Red.idat`
>    - `GSM1588706_8795207135_R02C02_Red.idat`
>    - `GSM1588707_8795207119_R06C02_Red.idat`
>    - `GSM1588704_8795207135_R01C02_Grn.idat`
>    - `GSM1588705_8795207119_R05C02_Grn.idat`
>    - `GSM1588706_8795207135_R02C02_Grn.idat`
>    - `GSM1588707_8795207119_R06C02_Grn.idat`
>
>    > ### {% icon details %} List of Zenodo URLs
>    > ```
>    > https://zenodo.org/record/1251211/files/GSM1588704_8795207135_R01C02_Red.idat
>    > https://zenodo.org/record/1251211/files/GSM1588706_8795207135_R02C02_Red.idat
>    > https://zenodo.org/record/1251211/files/GSM1588705_8795207119_R05C02_Red.idat
>    > https://zenodo.org/record/1251211/files/GSM1588707_8795207119_R06C02_Red.idat
>    > https://zenodo.org/record/1251211/files/GSM1588704_8795207135_R01C02_Grn.idat
>    > https://zenodo.org/record/1251211/files/GSM1588706_8795207135_R02C02_Grn.idat
>    > https://zenodo.org/record/1251211/files/GSM1588705_8795207119_R05C02_Grn.idat
>    > https://zenodo.org/record/1251211/files/GSM1588707_8795207119_R06C02_Grn.idat
>    > ```
>    {: .details}
>
>    {% include snippets/import_via_link.md %}
>
>    {% include snippets/import_from_data_library.md %}
>
> 3. Run **minfi read450k** {% icon tool %} with the following parameters:
>    - {% icon param-files %} *"red channel files"*: all files ending in `_Red`
>    - {% icon param-files %} *"green channel files"*: all files ending in `Grn`
>
> 4. Inspect the generated set of data
{: .hands_on}

> ### {% icon question %} Questions
>
> How are the Green and Red signals stored?
>
> > ### {% icon solution %} Solution
>    > Green and Red micro arrays are built up into `RGChannelSet`
> {: .solution}
>
{: .question}

# Preprocessing and  Quality Assessment

Preprocessing and data quality assurance is an important step in Infinium Methylation Assay analysis.

`RGChannelSet` represents two colour data with a green and a red channel and can be converted into methylated and unmethylated signals assigned to `MethylSet` or into Beta values build in `RatioSet`. Users can convert from `RGChannelSet` into `MethylSet` using the **minfi mset** {% icon tool %} or compute Beta values using **minfi set** {% icon tool %}. The **minfi qc** {% icon tool %} tool extracts and plots the quality control data frame with two columns mMed and uMed which are the medians of `MethylSet` signals (Meth and Unmeth). Comparing them against one another allows users to detect and remove low-quality samples.

> ### {% icon hands_on %} Hands-on: Preprocessing
> 1. Run **minfi mset** {% icon tool %} on the output from minfi_read450K to create `MethylSet` object
> 2. Run **minfi qc** {% icon tool %}  on the 'MethylSet' object created in the previous step to estimate sample-specific quality control. Once it is done click on the {% icon galaxy-eye %} (eye) icon next to the QC filename in your history to view QC image.
> 3.  Once you have inspected the quality control run **minfi rset** {% icon tool %} on the  'MethylSet'output from minfi_read450K  to convert methylation data from the `MethylSet` to a ratio of the methylated probe intensity and the overall intensity (sum of methylated and unmethylated probe intensities)
> 4. Run **minfi maptogenome** {% icon tool %} on the output from the previous step to map ratio data to the genomic positions
>
> > ### {% icon comment %} Preprocess and Normalize data
> > If your files require normalisation, you might prefer to use one of the other preprocessing tools provided in EWAS suite i.e. **minfi ppfun** {% icon tool %} or **minfi ppquantile**  {% icon tool %} look for recommendation at (ref).
> >
> {: .comment}
>
{: .hands_on}

![Quality Control](../../images/qcplot.png "Quality control plot")

# Removing probes affected by genetic variation

Incomplete annotation of genetic variations such as single nucleotide polymorphism (SNP) may affect DNA measurements and interfere results from downstream analysis.

Hansen, J. P. 2014 highly recommended to remove the probes that contain either a SNP at the methylated loci interrogation or at the single nucleotide extension

> ### {% icon hands_on %} Hands-on: Removing probes affected by genetic variation
> 1. Run **minfi dropsnp** {% icon tool %}  on the output of minfi_maptogenome to remove the probes that contain either a SNP at the methylated loci interrogation or at the single nucleotide extension
{: .hands_on}

# DMPs and DMRs Identification

The main goal of the EWAS suite is to simplify the way differentially methylated loci sites are detected. The EWAS suite contains **minfi dmp** {% icon tool %} tool detecting differentially methylated positions (DMPs) with respect to a phenotype covariate, and more complex **minfi dmr** {% icon tool %} solution for finding differentially methylated regions (DMRs). Genomic regions that are differentially methylated between two conditions can be tracked using a bumphunting algorithm. The algorithm first implements a t-statistic at each methylated loci location, with optional smoothing, then groups probe into clusters with a maximum location gap and a cutoff size to refer the lowest possible value of genomic profile hunted by our tool.

From the next step we will need an additional file that annotates illumina450K with genomic features, which we can obtain from UCSC.

> ### {% icon hands_on %} Hands-on: Data upload from UCSC
>
> 1. Search for `UCSC Main` in the tool search bar (top left)
>
> 2. Click on `UCSC Main` {% icon tool %}. You will be taken to the **UCSC table browser**
>
> 3. Set the following options:
>     - *"clade"*: `Mammal`
>     - *"genome"*: `Human`
>     - *"assembly"*: `Feb. 2009 (GRCh37/hg19)`
>     - *"group"*: `Regulation`
>     - *"track"*: `HAIB Methyl450`
>     - *"table"*: `GM12878 (wgEncodeHaibMethyl450Gm12878SitesRep1)`
>     - *"region"*: `genome`
>     - *"output format"*: `GTF - gene transfer (limited)`
>     - *"Send output to"*: `Galaxy` (only)
>
> 4. Click on the **get output** button at the bottom of the screen
> 5. On the next page, click on the **Send Query to Galaxy** button
> 6. Wait for the upload to finish
>
{: .hands_on}



> ### {% icon hands_on %} Hands-on: DMPs and DMRs Identification
>
> 1. Import `phenotypeTable.txt` from [Zenodo](https://zenodo.org/record/1251211#.WwREQ1Mvz-Y) or data library:
>    ```
>    https://zenodo.org/record/1251211/files/phenotypeTable.txt
>    ```
>
> 2. Run **minfi dmp** {% icon tool %} on the output of the minfi dropsnp tool adjusting the following parameters
>    - {% icon param-file %} *"Input set"*:`GenomicRatioSet`
>    - {% icon param-file %} *"Phenotype Table"*:`phenotypeTable.txt`
>    - *"Phenotype Type"*:`categorical`
>    - *"qCutoff Size"*:`0.5` (DMPs with an FDR q-value greater than this will not be returned)
>    - *"Variance Shrinkage"*:` TRUE` (This is recommended when sample sizes are small <10}
>    - *"Genome Table"*: `wgEncodeHaibMethyl450 ... `
>
> 3. Run **minfi dmr** {% icon tool %} on the output of minfi_dmp adjusting the following parameters
>    - {% icon param-file %} *"Input set"*:`GenomicRatioSet`
>    - {% icon param-file %} *"Phenotype Table"*:`phenotypeTable.txt`
>    - *"factor1"*: `sensitive`
>    - *"factor2"*: `resistant`
>    - *"maxGap Size"*:`250`
>    - *"coef Size"*:`2`
>    - *"Cutoff Size"*:`0.1`
>    - *"nullMethod"*:`permutation`
>    - *"verbose"*:`TRUE`
>
> 4. Click on the **minfi dmr** {% icon tool %} output in your history to expand it
> 5. Set the database build of your dataset to `Human Feb. 2009 (GRCh37/hg19) (hg19)`
>    {% include snippets/change_dbkey.md dbkey="hg19" %}
> 6. Click on `display at UCSC` towards the bottom of the history item.
>     This will launch UCSC Genome Browser with your Custom Track
{: .hands_on}

![Display at UCSC](../../images/ucsc.png "UCSC genome track showing differentialy methylated regions located on chromosome 6")

> ### {% icon comment %} Phenotype table
> Phenotype table can be in different sizes with different arguments, however the second column is required to contain phenotype covariate information for each sample.
{: .comment}

> ### {% icon question %} Questions
> How do we define phenotype covariate?
> > ### {% icon solution %} Solution
> > Phenotype covariate is the set of observable characteristics of an individual resulting from the gene-environment interactions
> {: .solution}
{: .question}

# Annotation and Visualization

In addition to downstream analysis users can annotate the differentially methylated loci at the promoter regions of genes with gene function description, and relationships between these concepts.

> ### {% icon hands_on %} Hands-on:  Annotate Differentially Methylated Position
> 1. Run **chipeakanno annopeaks** {% icon tool %}on the output of minfi_dmp with the following parameters
>   - {% icon param-file %} *"Differentialy methylated data"*: output of **minfi dmp** {% icon tool %}
>   - *"bindingType"*: `StartSite`
>   - *"bindingRegionStart"*:`-5000`
>   - *"bindingRegionEnd"*:`3000`
>   - *"Additional Column of Score"*:`8`
>
>        Position of column of score optional value if it is required
>
> 2. **Cut** {% icon tool %} on the previous output adjusting the following parameters to cut "gene_name" column from table of annotated peaks and then get a list of geenes
>   - *"Cut columns"*: `c16`
>   - *"Delimited by"*: `Tab`
>   - {% icon param-file %} *"From"*: output of **chipeakanno annopeaks** {% icon tool %}
>
> 3. **Remove beginning** {% icon tool %} of `Gene List` with the following parameters
>   - *"Remove first"*: `1`
>   - {% icon param-file %} *"from"*: output of **Cut** {% icon tool %}
>
> 4. Run  **clusterProfiler bitr** {% icon tool %} on the previous output adjusting the following parameters to convert the list of genes to list of entrez ID
>   - *"Input Type Gene ID"*: `SYMBOL`
>   - *"Output Type Gene ID"*: `ENTREZID`
>
> 5. Use the output of the  clusterProfiler bitr {% icon tool %} to run a GO Enrichment Analysis using **clusterProfiler go**
{: .hands_on}

![Functional annotations](../../images/funcann.jpg "Results of GO enrichments analysis for DMPs")

ID  | Description | pvalue | qvalue | geneID | Count
--- | ---  | --- | --- | --- | ---
GO:0048732 | gland development  | 1.38E-58 | 4.23E-55 | PTGS2 / KCNC1 / FZD1 /SLC22A18 /SLC22A3 (...) | 372
GO:1901652 | response to peptide | 3.99E-57 | 8.13E-54 | SULF1/ LAMA5/ MED1 /CFLAR/ MSX2 (...) | 359
GO:0048545 | response to steroid hormone | 1.38EE-54 | 2.11E-51 | HDAC9/ RAB10/ CFLAR/ WDTC1 (...) | 394

# Conclusion
{:.no_toc}

Epigenetic aberrations which involve DNA modifications give researchers an interest in identifying novel non-genetic factors responsible for complex human phenotypes such as height, weight, and disease. To identify methylation changes researchers need to perform complicated and time consuming computational analysis. Here, the EWAS suite becomes a solution for this inconvenience and provides a simplified downstream analysis available as a ready to run pipline in supplementary materials.

