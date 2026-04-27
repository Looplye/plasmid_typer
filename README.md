# PlasmidTyper

Command-line tool for automated plasmid family classification using protein marker search and a probabilistic model.

---

## Documentation

### Description

PlasmidTyper takes one or more plasmid protein sequences (`.faa`) and classifies them into plasmid families. It does so by searching a curated database of family-specific protein markers using MMseqs2, then scoring each plasmid against all families using a probabilistic model. The top-3 most likely families are reported for each plasmid, along with confidence scores.

### Why?

Plasmid typing is a key step in understanding the epidemiology of antimicrobial resistance (AMR). Existing tools often require exact sequence identity or are limited to specific incompatibility groups. PlasmidTyper uses protein-level homology search, making it more robust to sequence divergence, and supports both classical and machine-learning-based classification strategies.

---

## Installation & Usage

### Setup 

For plasmid_typer to work, you will need an environment with mmseqs2. You will also need prodigal if your files aren't already translated in a fasta protein format.

#### With conda 

To create a new environment, open a terminal and write :
```bash
conda create -n name python
```
with name the name you want your environment to be named.
Then, activate your new environment : 
```bash
conda activate name
```
And finally install mmseqs2 : 
```bash
conda install mmseqs2
```
If you need, you can also install prodigal the same way : 
```bash
conda install prodigal
```

#### With pip 
```bash
pip env smth
```


### Installation 

#### With conda 

dans le dossier setup/conda/grayskull et SANS ETRE DANS un environment : 
```bash
conda create -n test-plasmid -c ./conda-packages plasmid-typer
```

#### With pip 

dans un terminal :
```bash 
python3 -m pip install --index-url https://test.pypi.org/simple/ --no-deps plasmid-typer
```

if dependencies problems : 
```bash 
pip install --upgrade plasmid_typer
```

lien du projet :
https://test.pypi.org/project/plasmid-typer/0.0.10/ 



### Usage

```bash
./plasmid_typer.py query_path [options]
```

#### Pipeline steps

1. **Translation** *(optional)* — Prodigal translates nucleotide FASTA to protein FASTA (results written in `Prodigal_res`)
2. **MMseqs2 search & filtering** — each plasmid is searched against the marker database then hits below per-marker thresholds (qcov, tcov, pident) are removed (results written to `MMseqs2_res.tsv`)
3. **Typing** — hits are scored using the the AI model (results written to `typing_wide.tsv` and `typing_long.tsv`)

#### Requirements 

- Requires plasmid sequences as input (protein FASTA directly or nucleotide FASTA using the `-n` flag to be translated first via Prodigal)
- Plasmids with few or highly divergent markers may be classified as `UNKNOWN`

#### Positional arguments

`query_path` : Path to a `.faa` file or a folder containing multiple `.faa` files 

#### Optional arguments

| Flag | Default | Description |
|---|---|---|
| `-o`, `--output_path` | `./PlasmidTyper_<timestamp>` | Output folder |
| `-d`, `--database_path` | `database/` | database |
| `--model` | `database/model_presence_only_enrichedcore_*.joblib` | Pre-trained classification model |
| `-t`, `--n_threads` | `2` | Number of MMseqs2 threads |
| `-s`, `--sensitivity` | `7.5` | MMseqs2 sensitivity (4–7.5) |
| `-f`, `--files` | `False` | Disaggregate all MMseqs2 results in many files |
| `-n`, `--nucleotide` | `False` | Translate nucleotide input with Prodigal before search |
| `-v`, `--mmseqs_verbosity` | `2` | MMseqs2 verbosity (0=quiet, 3=verbose) |
| `-q`, `--prodigal_quiet` | `False` | Suppress Prodigal output |

#### Example

```bash
# Single plasmid (protein sequences)
./plasmid_typer.py my_plasmid.faa -o results

# Folder of plasmids, will use 8 CPU to run
./plasmid_typer.py plasmids_faa -o results -t 8

# Nucleotide input (will run Prodigal first)
./plasmid_typer.py my_plasmid.fasta -n -o results
```

---

#### Output

| File | Description |
|---|---|
| `typing_wide.tsv` | One row per plasmid — predicted family, top-3 families and probabilities |
| `typing_long.tsv` | One row per plasmid × rank — easier to filter and plot |
| `MMseqs2_res/` or `MMseqs2_res.tsv` | Raw MMseqs2 results (filtered) |

Plasmids with a top-1 probability below the confidence threshold are reported as `UNKNOWN`.

---

## Architecture

```
plasmid_typer/
├── plasmid_typer.py          # Entry point — argument parsing, call main module
└── scriptes/
    ├── main.py               # Main pipeline: path management, call all functions
    ├── config.py             # Parsing & parameter validation
    ├── mmseqs2.py            # MMseqs2: DB index creation, run MMseqs2 & result filtering
    ├── translate.py          # Prodigal for nucleotide → protein translation
    └── AI/
        ├── type_plasmid.py   # High-level typing functions
        └── utils.py          # CSR matrix construction, model inference, scoring

database/
    ├── markers_cluster_specific_plasmids_atleast3.faa   # Protein marker database
    ├── markers_thresholds_database.tsv                  # Per-marker qcov/tcov/pident thresholds
    └── model_presence_only_enrichedcore_*.joblib        # Pre-trained model
```

---

## Dependencies

| Dependency | Version |
|---|---|
| Python | 3.14.3|
| joblib | 1.5.3 |
| matplotlib | 3.10.8 |
| pandas | 3.0.1 |
| scikit-learn | 1.8.0|
| scipy | 1.17.1 |
| threadpoolctl | 3.6.0 |
| tqdm | 4.67.3 |

| mmseqs2 | 18.8cc5c|
| Prodigal | 2.6.3 *(only with `-n`)* |

---

## References

- Steinegger M, Söding J. *MMseqs2 enables sensitive protein sequence searching for the analysis of massive data sets.* Nature Biotechnology, 35(11), 1026–1028 (2017)
- Mirdita M, Steinegger M, Söding J. *MMseqs2 desktop and local web server app for fast, interactive sequence searches.* Bioinformatics, 35(16), 2856–2858 (2019)

---

## Authors

- Pierre Bouteyre - scripte creation (June 2025)
- Lou Planterose - optimization and documentation (2026)