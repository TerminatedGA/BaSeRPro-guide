<img src="https://github.com/user-attachments/assets/f7f9a00f-c9c0-4983-9b8b-e835b6ff5cb8" width="100" align="left" />

# BaSeRPro - Batch Sequence Retrieval and Processing for GenBank

### Application Preview

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/3c7ddb78-e904-4dd0-b6a2-c1e641e70578" width="750" />
      <br />
      <b>Retrieve + Align Sequences</b>
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/b57c1ae5-4d56-467f-864f-b67a339e90f5" width="750" />
      <br />
      <b>Align Only (Own File)</b>
    </td>
  </tr>
</table>

### Features

**Portable desktop application** (Windows) with two operating modes:

#### Retrieve + Align Sequences

Batch retrieval and processing of GenBank sequences via the NCBI Entrez API.

* **Search parameters** (comma-separated entries or multiple fields):
  * Organisms / species (at least one required)
  * Gene / Product / Locus tag / Amino acid sequence (at least one required for single-gene extraction)
  * Strain / Isolate / Title / Wildcard (Any)
  * Sequence length range
  * Country
  * Publication date range
  * Feature types: CDS, rRNA, tRNA, mRNA, tmRNA, ncRNA, precursor_RNA, prim_transcript, miscRNA

* **Three RefSeq modes:**
  * All sequences: no RefSeq filtering
  * RefSeq only: retrieve only RefSeq sequences
  * RefSeq priority: preferentially retrieve RefSeq sequences first; supplement with non-RefSeq sequences if the target count is not reached

* **Four extraction modes:**
  * Single gene
  * Entire sequence
  * Gene range: extracts a segment between two gene boundaries (requires two gene/product/locus tag arguments and an extract length limit)
  * Gene order: extracts a segment when genes appear in a specific order (requires two or more gene/product/locus tag arguments); supports both forward and inverse matching

* Automatic retry on duplicate accession numbers (e.g. NZ_ vs non-NZ_ duplicates)
* Automatic retry with email-only fallback if API key is invalid
* Progress bar for record fetching
* Optional target gene length for selecting the closest-length sequences

#### Align Only (Own File)

Align, trim, and cluster user-provided FASTA files without performing GenBank retrieval.

* Accepts one or more input FASTA files
* Multiple files can be aligned as a single merged file or individually

#### Post-retrieval Processing Pipeline

The following steps are shared by both modes:

* **Multiple sequence alignment** using FAMSA (via [pyfamsa](https://github.com/althonos/pyfamsa))
  * Codon-aware alignment: translates nucleotide sequences to protein, aligns at the protein level, and back-translates to preserve codon structure
  * Direct nucleotide alignment mode (fallback)

* **Alignment trimming** using a per-species gap threshold (0 -- 1)
  * A position is trimmed only if the gap frequency exceeds the threshold in **each** species
  * Default: 0.95

* **Consensus sequence generation**
  * Configurable consensus threshold (0 -- 1); default: 0.1
  * Full support for IUPAC ambiguous bases (both input and output)
  * Automatic detection of DNA, RNA, and amino acid sequences
  * Inserted as the first entry in the output alignment

* **Sequence clustering** using [MMseqs2](https://github.com/soedinglab/MMseqs2) easy-cluster
  * Configurable minimum sequence identity and minimum coverage
  * Generates per-cluster FASTA files and a cluster summary TSV

* **Cluster sorting** using [IQ-TREE](http://www.iqtree.org/) (v3.1.0) NONREV amino acid model
  * Sorts cluster representative sequences by phylogenetic tree order
  * Re-orders cluster member files and TSV to match

#### Output Files

* FASTA files of extracted/aligned/trimmed sequences
* `.xlsx` workbook containing:
  * Record annotations (accession, organism, gene, coordinates, description, country, RefSeq status)
  * Consensus degeneracy analysis (base frequencies at degenerate positions)
* When country filtering is enabled, sequences are split into per-country output files (plus an excluded-country file)

#### Advanced Options

* Expand extracted sequence range by a specified number of base pairs upstream and/or downstream (for checking gene boundaries)
* Save and load default configurations as JSON

### Requirements

* Python 3.10+
* Dependencies listed in [`src/requirements.txt`](src/requirements.txt):
  * BioPython, dacite, numpy, polars, pyfamsa, python-dateutil, requests, PySide6, tqdm, xlsxwriter
* Bundled executables (included in `src/binaries/mmseqs/` and `src/binaries/iqtree/`):
  * MMseqs2
  * IQ-TREE 3

### Running from Source

```bash
git clone https://github.com/TerminatedGB/BaSeRPro.git
cd BaSeRPro
pip install -r src/requirements.txt
python src/main.py
```

### Building a Standalone Executable

Requires [PyInstaller](https://pyinstaller.org/) and the dependencies above installed on Windows:

```bash
cd src
python -m PyInstaller --onefile --windowed ^
  --hidden-import=scoring_matrices ^
  --hidden-import=PySide6.QtCore ^
  --hidden-import=PySide6.QtGui ^
  --hidden-import=PySide6.QtWidgets ^
  --add-data="binaries/mmseqs;binaries/mmseqs" ^
  --add-data="binaries/iqtree;binaries/iqtree" ^
  --add-data="media/BaSeRPro.ico;media" ^
  --name="BaSeRPro" ^
  --icon=media/BaSeRPro.ico ^
  ./main.py
```

The generated executable is found in `src/dist/`.
