# scRNA Sequencing and Data Pre-processing

> Source: course slides

---

## 1. Library Construction

The following describes the DNBelab C-Series high-throughput single-cell RNA (scRNA) library construction and sequencing workflow. First, a high-quality single-cell suspension is prepared, ensuring a cell viability of over **80%** and a cell diameter not exceeding **40 μm**. Next, the prepared single-cell suspension, barcode-labeled magnetic beads (used to capture single-cell mRNA), and droplet generation oil are loaded into a portable microfluidic device.

This device generates a massive quantity of **water-in-oil droplets**. Under ideal conditions, each droplet contains one bead and one cell. Inside the droplet, the cell membrane lyses to release mRNA, which is then captured by the magnetic bead. Through **reverse transcription**, the mRNA from the same cell is tagged with an identical **barcode** (a unique molecular identifier for the cell), enabling single-cell labeling. Quality control is then performed on the cDNA (complementary DNA) after reverse transcription and amplification. cDNA that meets the required concentration and fragment size distribution standards is used for subsequent fragmentation and library construction to form a unique **circular library**. This library undergoes linear amplification to produce **DNA Nanoballs (DNBs)**—structures containing thousands of rolling-circle replicates—which are then used for downstream sequencing.

---

## 2. Sequencing Data Quality Control (QC)

### Data Structure

The raw data output consists of paired-end cDNA and Oligo FASTQ files.

* **DNA Data Structure**:
  * **Read 1**: Contains a 10bp barcode + 10bp barcode + 10bp **UMI (Unique Molecular Identifier)**. A UMI is a random molecular tag used to identify and quantify individual mRNA transcripts, helping to distinguish between original molecules and PCR duplicates.
  * **Read 2**: Contains 100bp cDNA.
* **Oligo Data Structure**:
  * **Read 1**: Contains 10bp barcode + 10bp barcode.
  * **Read 2**: Contains 10bp UMI + barcode + 10bp barcode.

The filtered FASTQ files are processed based on these structures (using the cDNA FASTQ as a primary example).

---

## 3. Reference Genome Alignment, Annotation, and UMI Correction

**Steps**:

* **Alignment**: STAR (version 2.7.1a) is used to align reads to the reference genome.
* **Annotation**: Genomic regions for the aligned reads are annotated based on a **GTF (Gene Transfer Format)** file.
* **UMI Correction**: The diversity of UMIs for each gene within a cell is used to evaluate gene expression levels. However, sequencing or PCR errors can lead to UMI inaccuracies, introducing bias. UMIs are corrected based on **Hamming distance** (a metric for measuring the difference between two strings of equal length). By default, if two sets of UMIs for the same gene in a single cell have a Hamming distance equal to 1, they are considered to have originated from the same transcript.

---

## 4. Beads Calling and Beads Merging

The **EmptyDrops** method is employed to select valid beads based on the statistical analysis of the raw **beads x genes** matrix.

Furthermore, the **Cosine Similarity** between beads is calculated based on the types and quantities of oligos present. The similarity values between each bead and all others are ranked from high to low. Each bead uses its similarity to the highest-ranking "empty bead" as a threshold for filtering. For the multi-bead merging algorithm:

* Connected components are calculated for values below the threshold.
* Strongly connected components are calculated for values above the threshold.

---

## 5. Cell Gene Expression

The **PISA count** module is used to perform statistical analysis on cells and genes, outputting three standard matrix files for downstream analysis. This format stores only non-zero count values, effectively reducing file size. The three files are defined as follows:

* **'barcodes.tsv.gz'**: The cell ID file.
* **'features.tsv.gz'**: The gene name file.
* **'matrix.mtx.gz'**: The cell/gene UMI count file. The first two lines are header/comment lines; the third line indicates the total number of genes, total cells, and total non-zero counts. From the fourth line onwards, the file contains gene index information, cell index information, and the corresponding non-zero UMI count values.

---

## 6. Basic Analysis Based on Expression Matrix

A **Seurat Object** serves as a container for storing single-cell datasets (such as count matrices) and the results of various analyses (such as **PCA** [Principal Component Analysis] or clustering results).
