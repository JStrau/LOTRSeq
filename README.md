
# LOTR‑Seq Analysis Pipeline

 
This repository contains scripts used for the analysis of LOTR‑Seq (Long-read and Transcriptome sequencing) data. The pipeline enables integration of single-cell RNA sequencing (scRNA-seq) with long-read Nanopore sequencing for transcript-based genotyping.

The workflow processes raw Nanopore sequencing data through basecalling, barcode recovery, alignment, and variant calling, enabling linkage of genotypic information to single-cell transcriptomes.

Please cite the following publication when using this pipeline:

Julian Grabek*, Jasmin Straube* et al ; Single cell long read genotyping of transcripts reveals discrete mechanisms of clonal evolution in post-MPN AML. Blood Adv 2026; ttps://doi.org/10.1182/bloodadvances.2025018902

A detailed STAR Protocol describing the LOTR‑Seq workflow will be available soon.

The LOTR‑Seq analysis pipeline consists of the following key steps:

**1. Convert FAST5 to POD5**

Convert raw Nanopore signal data from FAST5 to POD5 format:
 - Script: ConvertFast5ToPod5.pbs

**2. Basecalling**

Convert POD5 signal data into FASTQ files:
- Script: DoradoBaseCalling.pbs

Important:
Select the appropriate Dorado model based on platform and flow cell e.g.:

- MinION (FLO-MIN114): dna_r10.4.1_e8.2_260bps_sup@v4.1.0
- MinION (FLO-MIN106): dna_r9.4.1_e8_sup@v3.6
- PromethION (FLO-PRO114M): dna_r10.4.1_e8.2_400bps_sup@v5.0.

Always include:
```bash
--no-trim
```
to retain barcode information in the polyA region.


**3. Chimeric Read Splitting**

Remove adapters and split chimeric reads:

- Script: Porechop.pbs

**Note:**

Custom 10x adapter sequences must be added to Porechop.

**4. Barcode Recovery**

We evaluated several tools (BLAZE, scTagger, SiCeLORE) and found FLAMES2 to provide the highest sensitivity and specificity for barcode recovery.
- Assign barcodes:
     - Script: FLAMES.pbs
     - Tool: match_cell_barcode
     - Parameters:
         - Allow up to 2 mismatches
         - UMI length: 12

- Split reads by barcode:
   - Script: FLAMES_BC_Split.pbs

**5. Read Alignment and Filtering**

- Script: Minimap_and_Filter.pbs
Steps:
- Align reads to the human reference genome using minimap2
- Remove:
  - secondary alignments
  - supplementary alignments
  - chimeric reads
using samtools and sambamba

**6. Variant Calling**

- Script: SNPBasePileUP.pbs
- SNV detection using bcftools mpileup

**Important Notes**

- Use --no-trim during basecalling to preserve polyA tails and barcode regions
- Porechop requires custom addition of 10x adapter sequences
- Barcode recovery is typically ~50% and may improve with updated basecalling models
- Filtering supplementary alignments improves variant accuracy
- Sequencing depth and gene expression strongly influence genotyping success
