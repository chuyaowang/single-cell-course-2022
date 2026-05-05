# scATAC Sequencing and Pre-processing

> source: course slides

---

## 1. Biological Background

**ATAC-seq** stands for **Assay for Transposase-Accessible Chromatin using sequencing** ATAC-seq utilizes the tranposon-transposase mechanism found in nature. To understand ATAC-seq, it helps to think of them as a **biological "copy-paste" machine**. In nature, they are part of a system used by "jumping genes" to move around a genome.

A **transposon** is a specific segment of DNA that can move from one location to another within a genome. It usually consists of a "payload" (like an antibiotic resistance gene) flanked by specific "recognition sequences" at both ends.

A **transposase (TnP)** is the enzyme (a protein) that carries out the movement. It acts like a pair of molecular scissors and a glue stick combined. It recognizes the specific ends of the transposon, grabs them, cuts the target DNA, and "pastes" the transposon into the new site. The specific enzyme used in ATAC-seq is called **Tn5**. It is highly efficient and has been engineered in labs to be "hyperactive," meaning it works much faster than the version found in nature.

### 1.1 How They Work Together in ATAC-seq

In a normal cell, most DNA is tightly packed around proteins (histones) like thread on a spool. Only a small amount is "open" or "accessible" for the cell to use.

1. **The Probe:** We add the **Tn5 Transposase** loaded with **artificial transposons** (adapters) to the cell nuclei.
2. **The Constraint:** The enzyme is too bulky to get into the tightly packed DNA. It can only reach the **open chromatin**.
3. **The Action (Tagmentation):** The enzyme cuts the open DNA and simultaneously pastes the sequencing adapters into it. This dual action is called **Tagmentation** (Tagging + Fragmentation).
4. **The Result:** By sequencing only the pieces that the transposase managed to "tag," we create a map of exactly which parts of the genome were open and active in that specific cell.

---

## 2. The Library Construction Process

### 2.1 Principles of Tn5 Transposase in ATAC-seq Library Construction

The **Tn5 transposon** consists of a core sequence encoding resistance genes and two **Inverted Repeats (IR)** called **IS50** sequences. IS50 features 19bp inverted ends—the **Outside End (OE)** and **Inside End (IE)**—which serve as the binding sites for the Transposase.

During a transposition event, two TnP molecules bind to the OE ends to form two **TnP-OE complexes**. These complexes dimerize via the C-terminus of the TnP to form a active **Tn5 Transposome**. Once activated, the complex cleaves the target DNA and inserts the transposon into the sequence. The resulting "sticky ends" are filled by DNA polymerase and ligase, creating 9bp **Direct Repeats (DR)** at both ends.

Researchers discovered that the entire transposon sequence is not required; the transposase only needs the core end sequences to insert and link DNA into the genome. By attaching **sequencing adapters** (synthetic DNA segments required for the sequencing machine to recognize the sample) to these core end sequences, adapters can be introduced directly during fragmentation. While traditional library prep requires time-consuming steps like physical fragmentation, end repair, adapter ligation, and multiple purifications, using Tn5 combines these into a single step, vastly improving efficiency.

### 2.2 Single-Cell ATAC-seq (scATAC-seq) Library Construction

The following describes the **DNBelab C-Series** high-throughput single-cell ATAC library construction workflow. This system utilizes a negative pressure-driven **droplet microfluidic system** combined with proprietary high-density microbeads to achieve high-throughput cell barcoding.

1. **Preparation:** High-quality **single-cell nuclear suspensions** (isolated nuclei) are prepared, ensuring a viability rate above **80%**.
2. **Encapsulation:** The nuclear suspension, barcode-labeled magnetic beads (for capturing *DNA* fragments), and droplet generation oil are loaded into a portable microfluidic device.
3. **Transposition & Labeling:** The device generates thousands of **water-in-oil droplets**. Ideally, each droplet contains one bead and one nucleus. Inside the droplet, the Tn5 enzyme enters the nucleus to fragment **open chromatin** (accessible DNA regions not tightly wrapped around proteins). The released DNA fragments are captured by the bead and tagged with a unique **barcode** (a molecular identifier). This ensures all DNA fragments from the same individual cell share an identical label.

---

## 3. scATAC Automated Analysis V3 System: Process and Results

### 3.1 Data Structure

The raw output consists of paired-end reads:

* **Read 1 (70bp):** Contains a 1-20bp **barcode** sequence followed by 21-70bp of genomic **DNA** sequence.
* **Read 2 (50bp):** Consists entirely of 1-50bp genomic **DNA** sequence.

### 3.2 Alignment

Alignment is performed using **Chromap**, a fast and accurate tool for chromatin profiling data.

**Advantages of Chromap:**

1. **Speed:** Approximately 10 times faster than **BWA-MEM2** (Burrows-Wheeler Aligner - Maximal Exact Matches), a standard industry alignment tool.
2. **Accuracy:** Maintains a level of accuracy comparable to BWA-MEM2.

**The Chromap Alignment Workflow:**

> See [Chromap paper](https://www.nature.com/articles/s41467-021-26865-w)

1. **Barcode Identification:** Identifies barcodes and matches them against a **whitelist** (a known list of valid barcode sequences). Reads that do not match are discarded (currently, no mismatches are permitted).
2. **Adapter Trimming:** Removes the synthetic adapter sequences used during library prep.
3. **Genomic Mapping:** Aligns the remaining DNA sequences to the reference genome.
4. **Output:** Generates an alignment file (typically in **BED** format, which stands for **Browser Extensible Data**, a text file used to store genomic coordinates).

## 3.3 Background Removal (De-noising)

There is a significant difference in the number of **fragments** captured by beads associated with intact cells compared to beads that either captured no cell or a ruptured cell. Therefore, we rank every bead based on its total fragment count. If the fragment count drops suddenly (forming a "knee" or "elbow" in the plot), the beads below this **inflection point** (sudden drop) are considered **empty beads**.

## 3.4 Beads Merging

During the experiment, a high concentration of beads is often used to maximize cell capture, which can result in multiple beads being encapsulated within a single droplet. To restore the true chromatin accessibility profile of a single cell, the **d2c** (droplet-to-cell) algorithm is used to merge beads based on the similarity of their captured fragments. See [reference paper](https://www.nature.com/articles/s41467-020-14667-5).

* **Scenario:** A single droplet captures multiple beads but only one cell.
* **Method:** By calculating the **Jaccard Index** (a statistical measure of similarity) between two beads, the algorithm can infer whether those beads originated from the same droplet. If the similarity is high, they are merged into a single cell profile.

## 3.5 Peak Calling

**Peak calling** is a computational method used to identify regions in the genome that are significantly enriched with aligned sequencing reads. In the context of **ATAC-seq**, a "peak" represents a region of **open (accessible) chromatin**.

The current pipeline utilizes **MACS2** (Model-based Analysis of ChIP-Seq), a tool based on read count statistics, to perform peak calling.
