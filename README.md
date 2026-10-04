# Gut bacterial communities in invasive and native birds of the Iberian Peninsula

Analysis code accompanying the manuscript. Everything reported in the paper is produced by
`TFM_MartinezVergara.ipynb`, from the raw per-sample genus count tables through to the figures
and statistical tables.

## What the study asks

Cloacal bacterial communities of two invasive parrot species (*Myiopsitta monachus*, *Psittacula
krameri*) and three native passerines (*Passer domesticus*, *Sturnus unicolor*, *Turdus merula*),
sampled in urban parks in Seville, Spain, and profiled by 16S rRNA amplicon sequencing on the Oxford
Nanopore platform. Three questions, in order:

1. **Between species** — does the host species predict the diversity and composition of its gut
   bacterial community, once sampling locality is accounted for?
2. **Within species** — within a host species, is diversity related to locality, sex or body
   condition?
3. **Composition** — which bacterial genera drive any differences, and how consistent are they
   between individuals of the same host species?

Sampling effort was not equal across the five species. *Myiopsitta monachus* yields too few birds
that clear every quality and completeness requirement to estimate its own mean beside a locality
effect, so it is reported descriptively throughout and left out of the models, under the rule fixed
in section 02.2. The inferential comparison is between *Psittacula krameri* and the three native
passerines.

## Reading the results without running anything

The notebook is written to be consulted as a web page. Every code cell prints a verification block,
every result table then follows in a cell of its own as a searchable, sortable table, and each main
figure is followed by an interactive version whose points can be hovered to identify the individual
bird or genus behind them. Exporting the executed notebook gives one self-contained file in which all of that still
works with no kernel behind it:

```bash
jupyter nbconvert --to html --embed-images TFM_MartinezVergara.ipynb
```

Both libraries are embedded in the notebook rather than linked over the network, so the page stays
interactive offline. One caveat: the notebook preview on github.com strips JavaScript, so the
interactivity shows in the exported HTML — served from GitHub Pages, or downloaded and opened —
and not in the `.ipynb` preview.

## Running the analysis

```bash
pip install -r requirements.txt
jupyter notebook TFM_MartinezVergara.ipynb     # run all cells
```

It runs top to bottom with no manual intervention and takes a few minutes on an ordinary laptop.
Principal coordinates analysis, PERMANOVA, PERMDISP, the centered log-ratio transform and
multiplicative zero replacement are implemented directly from their published definitions, with
self-tests, rather than pulled from a package that does not install on every platform, so the
notebook runs unchanged on Windows, macOS and Linux.

Randomness — rarefaction draws, permutation tests, randomized quantile residuals — is seeded from a
single `RANDOM_SEED`, so a re-run reproduces the reported numbers exactly.

## Inputs

```
data/raw/Base_de_datos.csv             bird metadata: species, locality, date, sex, morphometrics
data/raw/data_sample/<SPECIES>/        one Genus_counts.tsv per bird
data/raw/gtdb_taxonomic_reference.tsv  the reference used to resolve genus names
```

## Outputs

```
data/processed/       intermediate matrices, including the rarefied matrix and the per-bird
                      diversity table every model is fitted to
results/statistics/   effect tests, marginal means, post-hoc comparisons, sensitivity analyses
results/taxonomy/     compositional results, including the CLR matrix and genus loadings
results/diversity/    distance matrices and goodness-of-fit tables
results/qc_reports/   every exclusion, with the reason attached
results/output_index.csv   an index of all of the above, written by section 11.1
images_graphs/        figures by topic; images_graphs/diagnostics/ holds the residual and
                      goodness-of-fit plots
```

## How the analysis is organized

| Section | Content | Input matrix |
| --- | --- | --- |
| 01.0 | Locate and load input datasets | raw `Genus_counts.tsv` files + metadata |
| 02.0 | Taxonomic curation and the analytical matrix | `community_matrix_pre_curation` |
| 03.0 | Sequencing depth diagnostics | `community_matrix_analysis` |
| 04.0 | Rarefaction depth selection | `community_matrix_analysis` |
| 05.0 | Alpha diversity | `rarefied_matrix` (50 replicates) |
| 06.0 | Between-species models | `alpha_data` |
| 07.0 | Within-species models | `intra_specific_datasets` |
| 08.0 | Beta diversity | `rarefied_matrix` → Bray–Curtis |
| 09.0 | Taxonomic composition | `composition_counts` → CLR |

Every curation and modeling decision is made against an explicit, stated criterion rather than a
convenient constant, and each criterion is fixed before the data it acts on are inspected. Nothing
is filtered silently: every exclusion is written to `results/qc_reports/` with its reason. Each code
cell ends with a printed summary and, where an invariant has to hold for the next cell to mean
anything, an `assert` that checks it — so a notebook that runs to the end has verified its own
assumptions along the way.

Distribution families follow from the support of each response rather than from a search over
candidates, and every reported model is checked with randomized quantile residuals (Dunn & Smyth
1996), which are exactly standard normal under a correctly specified model whatever the family.
PERMANOVA is always reported with a PERMDISP on the identical samples, because a significant
PERMANOVA with a significant PERMDISP means the groups differ in community *structure*, not in mean
composition.

## Regenerating the notebook

The `.ipynb` is generated, not hand-edited. The cell sources live in `build_notebook.py`, and
`emit.py` writes the notebook from them. Edit the builder, not the notebook.

```bash
python emit.py        # writes TFM_MartinezVergara.ipynb
```

`exec_cells.py` runs every cell in one namespace as a fast check while editing; `run_nb.py` executes
the notebook in a real Jupyter kernel and reports the error and figure counts.
