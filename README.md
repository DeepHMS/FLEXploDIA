<div align="center">

# FLEXploDIA

**The pipeline for advanced proteomics Library Exploration, Visualization, PASEF Method Development and Validation**

[![Try it online](https://img.shields.io/badge/Try%20it-online-blue)](https://deepbiswas-flexplodia.hf.space/)
[![Desktop version](https://img.shields.io/badge/Desktop-download-brightgreen)](https://github.com/DeepHMS/FLEXploDIA)

Developed and maintained in the **Steen2 Lab**, Boston Children's Hospital

</div>

---

## Try it online

**▶ [deepbiswas-flexplodia.hf.space](https://deepbiswas-flexplodia.hf.space/)**

No installation, nothing to configure — open the link and upload a library.

> ### ⚠️ The online version is for small libraries only
>
> It runs on a **small server shared by everyone using it at the same time**. It is meant for
> exploring modest spectral libraries and trying the tool out — not for processing real
> experiment datasets.
>
> | | Online version | Desktop version |
> |---|---|---|
> | **Recommended library size** | up to **50 MB** | no limit |
> | **Hard maximum** | **100 MB** (upload is refused above this) | no limit |
> | **DIA-NN `report-lib.parquet`** | **not supported** | fully supported |
> | **Speed** | shared CPU, slower | full speed on your own machine |
> | **Your data** | uploaded to a shared public server | never leaves your machine |
>
> **If you work with real datasets, large libraries, or `.parquet` files, use the desktop
> version.** It installs on your own PC, has no size limit, is considerably faster, and keeps
> your unpublished data local. See [Installation](#installation-desktop-version) below.

### Why `.parquet` is blocked online

A DIA-NN `report-lib.parquet` is small on disk but enormous once opened — a 69 MB file holds
around **3.7 million fragment rows**. Converting it writes hundreds of megabytes of text and
holds a decoded copy in memory while doing so. That does not fit in a shared container, so the
Library Optimizer and Library Extractor refuse `.parquet` online and point you here instead.
The desktop version converts the same file in about a minute.

---

## What FLEXploDIA does

FLEXploDIA is a single application for working with DIA/dia-PASEF spectral libraries end to
end: understanding what is in a library, cleaning and standardising it, designing a PASEF
acquisition method against it, and checking afterwards whether the acquisition delivered what
the method predicted.

| Tab | What it does |
|---|---|
| 📊 **Library Explorer** | Eleven interactive views of a whole library or a single protein — precursor *m/z*, ion-mobility and retention-time distributions, intensity relationships, charge-state breakdown, a 3D ion cloud, and a b/y fragment-coverage plot. Exact precursor, peptide and protein counts are computed from the whole file, not a sample. |
| 🧰 **Library Editor** | Three sub-tabs that prepare a library: **Library Optimizer** (convert any supported format to one global format and map it to your FASTA), **Library Extractor** (pull a focused sub-library out by protein, isoform or modification), **Library Merger** (combine two libraries with column alignment and deduplication). |
| 🔬 **Prot./Pep. Cloud Viz.** | Maps one protein's peptidoforms in ion-mobility × *m/z* space, with fragment-ion fingerprints, modification and missed-cleavage colouring, FASTA sequence coverage, and optional overlay of a PASEF window scheme. |
| ⚙️ **Method Developer** | Designs and exports **Slice-PASEF** and **Diagonal-PASEF** isolation schemes from a real library's ion distribution, with coverage scoring and ready-to-use method files. |
| 🧪 **PASEF Validator** | Compares the DDA library a method was built from against the DIA-NN library the acquisition actually produced — protein by protein, peptidoform by peptidoform, with side-by-side fragment fingerprints. |

Two app-wide panels sit above the tabs: an editable **Modification Table** (UniMod names and
plot colours, used consistently everywhere) and an **Abbreviations** reference.

---

## Supported library formats

| Format | Read | Notes |
|---|---|---|
| **FragPipe / MSFragger** `.tsv` | ✅ | The reference layout. Works everywhere without conversion. |
| **DIA-NN** library `.tsv` | ✅ | Convert in the Library Optimizer for full plot support. |
| **DIA-NN** `report-lib.parquet` | ✅ desktop only | Converted to `.tsv` in the Optimizer. Requires `pyarrow`. |
| **Bruker ProteoScape** `.tsv` / `.csv` | ✅ | Convert in the Library Optimizer for full plot support. |

A library that has not been through the Library Optimizer may be missing columns some plots
need — most often a retention-time column. The Explorer draws what the library supports and
names the axis according to the scale the data is actually on.

---

## Installation (desktop version)

**Requirements:** Python **3.10 or newer** on Windows, macOS or Linux.

```bash
# 1. Get the code
git clone https://github.com/DeepHMS/FLEXploDIA.git
cd FLEXploDIA

# 2. Create an isolated environment
python -m venv venv

#    Windows
venv\Scripts\activate
#    macOS / Linux
source venv/bin/activate

# 3. Install the dependencies
pip install -r requirements.txt

# 4. Run it
python app.py
```

The app starts a local web server and prints a URL — by default
**http://127.0.0.1:7860**. Open it in your browser. Everything runs on your own machine;
nothing is uploaded anywhere.

To stop it, press `Ctrl+C` in the terminal.

### Dependencies

```
gradio
pandas
numpy
plotly
scipy
pyarrow      # only needed to read DIA-NN .parquet libraries
```

### Notes

- **`pyarrow` is required for `.parquet`.** Without it, every other format still works and
  the app says what to install if you open a `.parquet`.
- **No upload size limit locally.** The desktop build deliberately removes the ceiling the
  online version has, so multi-gigabyte libraries load.
- **First launch may take a few seconds** while Gradio starts.

---

## Typical workflows

### 1. Standardise a library and map it to your FASTA

**Library Editor → Library Optimizer**

Upload the library and the FASTA the search actually used, then press **Convert & Map**. In one
pass FLEXploDIA:

1. keeps only rows that name a protein;
2. splits every protein group (`P54819;P54819-2;P54819-3`) so each row carries **one UniProt ID
   and one gene**;
3. matches each accession against the FASTA **by identifier** — sequences are never compared;
4. writes what matched as a `.tsv`, plus a **status file** accounting for every row of the
   source.

You get two downloads. The status log prints in the box so the protein count can be read
alongside how many rows never had an accession in the first place.

> **Use the FASTA the search actually used.** Whether it contains isoform entries decides how
> isoform accessions are treated: with isoforms they stay separate proteins; without them they
> fold into their canonical parent. A different FASTA gives different protein counts — this is
> expected, and it is why the status file exists.

### 2. Explore what is in a library

**Library Explorer** — upload and pick `Overall`, or a single protein from the dropdown.
Headline counts are exact (computed by streaming the whole file), while the scatter plots draw
a representative sample so they stay responsive on large libraries.

### 3. Extract a focused sub-library

**Library Editor → Library Extractor** — load a library, optionally strip chosen
modifications, then keep or remove selected proteins (with related isoforms detected for you)
and export the result.

### 4. Design a PASEF method

**Method Developer** — load a library, choose Slice-PASEF or Diagonal-PASEF, tune the window
scheme against the real ion distribution, and export the method files as a `.zip` with a
coverage summary.

### 5. Validate an acquisition

**PASEF Validator** — upload the DDA library the method came from, the DIA library DIA-NN
produced, the shared FASTA and the PASEF method file. Pick a protein and compare peptidoform
by peptidoform, with fragment fingerprints side by side.

---

## Reference

### The global library format

The Library Optimizer writes a FragPipe-shaped layout so every other tab reads one accession
and one gene per row.

**Core columns** — `ProteinId`, `GeneName`, `PeptideSequence`, `ModifiedPeptideSequence`,
`PrecursorCharge`, `PrecursorMz`, `PrecursorIonMobility`, `ProductMz`, `LibraryIntensity`,
`Annotation`

**Added by the Optimizer** — `AllMappedProteins`, `AllMappedGenes`, `Proteotypic`,
`NormalizedRetentionTime`, `AverageExperimentalRetentionTime`

`AllMappedProteins` preserves the protein group exactly as the search engine wrote it, so the
original row is always recoverable after the group has been split apart.

### Retention time

The two retention-time columns are named explicitly because they are **not interchangeable**:

- `AverageExperimentalRetentionTime` — real elution time (stored in seconds)
- `NormalizedRetentionTime` — indexed retention time (iRT), which runs **negative** and is
  **not minutes**

A DIA-NN `report-lib` carries only indexed retention time, so the Optimizer writes it to the
normalised column and says so. Plots and tooltips label whichever scale the library is actually
on, and never present iRT as minutes.

### The status file

One row per row of the source library, with columns `SourceRow`,
`ModifiedPeptideSequence`, `PrecursorCharge`, `ProteinId`, `Status`, `MappedAccessions`,
`UnmappedAccessions`. Every row gets exactly one status:

| Status | Meaning |
|---|---|
| `Protein ID found and mapped to FASTA` | every accession in the group matched — row kept |
| `Partially mapped to FASTA` | some accessions matched, others are not in the FASTA — row kept, unmatched ones named |
| `No FASTA protein mapped` | the row names proteins, but none are in the FASTA |
| `No Protein ID found` | the row names no protein at all |
| `Low confidence (Q.Value above cut-off)` | filtered out by the q-value setting |
| `Decoy` | decoy entry |

The counts balance against the source row total, so nothing disappears without a reason you
can read.

---

## Performance

Measured on a DIA-NN `report-lib.parquet` of **3,721,046 fragment rows** with a
**40,000-entry FASTA**, on a fast SSD:

| Step | Time |
|---|---|
| Indexing the FASTA (40,000 entries) | ~0.2 s |
| Convert & Map, canonical-only FASTA | ~70 s |
| Convert & Map, FASTA including isoforms | ~95 s |

Writing the output is the dominant cost — the converted library is several hundred megabytes of
text. Expect longer on a mechanical disk, or if antivirus or a cloud-sync client (OneDrive,
Dropbox) inspects the file as it is written; excluding the working folder from both helps
considerably.

**Isoforms affect the output size.** A FASTA that lists isoforms keeps them as separate
proteins, so a group like `P54819;P54819-2;P54819-3;P54819-5` becomes four rows instead of one.
A canonical-only FASTA collapses them. Both are correct — they describe different search
spaces.

Long operations show a **live row count** over the FLEXploDIA logo, so you can tell processing
from a hang.

---

## Data and privacy

**Desktop version** — everything stays on your machine. No library, FASTA or result is sent
anywhere. The only outbound request FLEXploDIA makes is an optional UniProt lookup to fetch a
protein sequence for the coverage view when you have not supplied a FASTA.

**Online version** — uploads go to a public shared server. Files are held only in the
container's temporary workspace and are wiped on every restart or sleep, so nothing is stored
long-term — but the server is shared. **Do not upload unpublished or sensitive data to the
online version.** Use the desktop build for anything confidential.

---

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| `❌ This file is NNN MB. The online version accepts libraries up to 100 MB` | The online size ceiling. Use the desktop version. |
| `.parquet libraries are not processed on the online version` | Expected online. Use the desktop version. |
| `Reading .parquet needs the pyarrow package` | `pip install pyarrow` into the same environment. |
| `This parquet is incomplete: it begins correctly but the end of the file is missing` | The upload was interrupted or the app was stopped mid-copy. The original is usually fine — upload again and let it finish. |
| `This is not a parquet file` | A `.tsv`/`.csv` renamed `.parquet`. Switch the library type to the `.tsv` option. |
| Retention-time plots are empty | The library carries no retention-time column. Run it through the Library Optimizer. |
| Protein count differs from what you expected | It depends on the FASTA you map against — an isoform-bearing FASTA yields more proteins than a canonical-only one. The status file explains every row. |
| Port 7860 already in use | An earlier run is still alive. Close it, or set `GRADIO_SERVER_PORT` to another port. |

---

## Citation

<!-- TODO: replace with the paper or preprint once available. -->
If FLEXploDIA is useful in your work, please cite this repository and the accompanying
publication (details to follow).

---

## Contact

Developed and maintained by the **Steen2 Lab**, Boston Children's Hospital, USA.

- **Dr. Deeptarup Biswas** — [Deeptarup.Biswas@childrens.harvard.edu](mailto:Deeptarup.Biswas@childrens.harvard.edu)
- **Prof. Judith Steen** — [Judith.Steen@childrens.harvard.edu](mailto:Judith.Steen@childrens.harvard.edu)

Bug reports and feature requests are welcome via
[GitHub Issues](https://github.com/DeepHMS/FLEXploDIA/issues).

---

## License

<!-- TODO: add a LICENSE file and name it here (e.g. MIT, BSD-3-Clause, GPL-3.0). -->
See `LICENSE`.
