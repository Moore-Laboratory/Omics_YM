# Omics_YM

R code for metabolomic/proteomic data analysis from mass spectrometry and RNA-sequencing studies.

## Overview

This repository contains a comprehensive collection of R scripts and RMarkdown workflows for analyzing multi-omics datasets, including:
- **Metabolomic data** from mass spectrometry adapted from https://github.com/vari-bbc/KOBOLD_Scripts
- **Proteomic data** from mass spectrometry  adapted from https://github.com/vari-bbc/KOBOLD_Scripts
- **Bulk RNA-sequencing data** under different experimental conditions

The project uses analysis workflows standardized for the Moore laboratory. These workflows include processing, quality control, statistical analysis, and pathway enrichment of omics data.

## Quick Start

New to this repository? Get started in 3 steps:

1. **Clone the repository**:
   ```bash
   git clone https://github.com/Moore-Laboratory/Omics_YM.git
   cd Omics_YM
   ```

2. **Open the R Project** in RStudio:
   ```
   Omics_YM.Rproj
   ```

3. **Choose your analysis** and follow the appropriate workflow section below:
   - Mass spectrometry data → See [Mass-spec/](#mass-spec)
   - RNA-seq data → See [bulk_RNA-seq_HFD/](#bulk-rna-seq-hfd) or [bulk_RNA-seq_KI_vs_WT/](#bulk-rna-seq-ki-vs-wt)

## Project Structure

```
Omics_YM/
├── README.md                          # This file
├── LICENSE                            # GPL-3.0 License
├── Omics_YM.Rproj                     # R Project file
├── Mass-spec/                         # Mass spectrometry analyses
│   ├── Metabolomics_Analysis.Rmd      # Metabolomic data processing and analysis
│   ├── Proteomics_Analysis.Rmd        # Proteomic data processing and analysis
│   └── renv.lock                      # R environment dependencies for Mass-spec
├── bulk_RNA-seq_HFD/                  # Bulk RNA-seq analysis (High Fat Diet study)
│   ├── enrichment_4_comps_2025.Rmd    # Pathway enrichment analysis
│   └── four_more_comps_2025.Rmd       # Gene expression comparisons
└── bulk_RNA-seq_KI_vs_WT/             # Bulk RNA-seq analysis (KI vs WT comparison)
    ├── deseq_1348_with_sire.Rmd       # DESeq2 differential expression analysis
    └── pathways_1348_with_sire.Rmd    # Pathway analysis and visualization
```

## Directory Descriptions

### Mass-spec/
Contains workflows for analyzing mass spectrometry-based metabolomic and proteomic data.

- **Metabolomics_Analysis.Rmd**: End-to-end metabolomic analysis including:
  - Data quality control and normalization
  - Statistical analysis of metabolite abundances
  - Metabolite annotation and identification
  - Visualization of results

- **Proteomics_Analysis.Rmd**: Complete proteomic analysis pipeline featuring:
  - Protein quality metrics
  - Statistical differential abundance analysis
  - Visualization of results
  - Protein functional analysis and integration pathway databases

### bulk_RNA-seq_HFD/
Analysis of bulk RNA-sequencing data from the High Fat Diet (HFD) experimental study.

- **four_more_comps_2025.Rmd**: Comparative gene expression analysis across multiple conditions with visualization
- **enrichment_4_comps_2025.Rmd**: Gene ontology and pathway enrichment analysis of differentially expressed genes

### bulk_RNA-seq_KI_vs_WT/
Analysis comparing Knock-In (KI) genetic model versus Wild-Type (WT) control.

- **deseq_1348_with_sire.Rmd**: DESeq2-based differential expression analysis with statistical modeling
- **pathways_1348_with_sire.Rmd**: Pathway enrichment and biological interpretation of results

## Requirements

### Software
- **R** (>= 4.0)
- **RStudio** (recommended for working with RMarkdown files)

### R Packages
Key dependencies include:
- Data analysis: `DESeq2`, `limma`, `tidyverse`, `data.table`
- Statistical: `igraph`, `scales`
- Visualization: `ggplot2`, `pheatmap`, `ComplexHeatmap`
- Pathway analysis: `clusterProfiler`, `pathview`, `msigdbr`
- Proteomics/Metabolomics: `xcms`, `metaboAnalyst`

For reproducibility, each analysis directory includes an `renv.lock` file specifying exact package versions. Use `renv::restore()` to recreate the analysis environment.

## Usage

### Prerequisites
1. Clone this repository:
   ```bash
   git clone https://github.com/Moore-Laboratory/Omics_YM.git
   cd Omics_YM
   ```

2. Open the R Project:
   ```r
   # In RStudio, open Omics_YM.Rproj
   ```

### Running Analyses

Each RMarkdown file (`.Rmd`) is a self-contained analysis workflow. To run an analysis:

1. Open the desired `.Rmd` file in RStudio
2. Install/update dependencies if needed:
   ```r
   # For Mass-spec analyses
   renv::restore(project = "Mass-spec")
   
   # Or generally
   renv::restore()
   ```
3. Change the `file =` path to the dataset you want to analyze

4. Click "Knit" or run:
   ```r
   rmarkdown::render("path/to/analysis.Rmd")
   ```

This will generate an HTML report with analysis results and visualizations.

## Analysis Workflows

### Metabolomics Workflow

The pipeline expects a CSV metabolomics data file.

#### Input Data Format

Required input structure:
- First column contains sample names
- Remaining columns contain metabolite abundance values
- Sample names encode genotype and diet information via hyphens, e.g., `ID_genotype-diet_replicate`
- QC and blank samples are identified by sample names containing "qc" or "blank"

**Example input structure**:

| Sample | Metabolite_001 | Metabolite_002 | Metabolite_003 |
|--------|----------------|----------------|----------------|
| S1_WT-normal_rep1 | 100.5 | 205.3 | 89.2 |
| S2_KO-HFD_rep1 | 95.2 | 210.1 | 92.5 |
| QC_pool_01 | 102.1 | 207.8 | 88.9 |
| BLANK_01 | 2.3 | 1.5 | 0.8 |

The script automatically renames the first column to `Sample`.

#### Metadata Input

The workflow also requires an Excel metadata file containing sex information, but can be easily modified if sex is present in the sample IDs.

**Before running**: Update the working directory path to match your data location:
```r
setwd("~/path/to/your/data/")
```

#### Required R Packages

**CRAN packages**:
- ggplot2
- ggforce
- dplyr
- ggrepel
- ggsci
- factoextra
- emmeans
- patchwork
- pheatmap
- readxl

**Bioconductor / specialized packages**:
- limma
- imputeLCMD
- brglm2

#### Workflow Summary

1. Data Import
2. Blank Filtering
3. QC and Blank Removal
4. Metadata Parsing
   - Extracts sample-level experimental annotations from sample names
5. Sex Metadata Merge
6. Missing Data Assessment
7. Missingness Specificity Testing (brglm2)
8. Missing Data Filtering (>30% missingness)
9. Missing Value Imputation (QRLIC)
10. Exploratory Analysis (PCA and heatmaps)
11. Differential Abundance Analysis (limma)
    - Model design: `~ 0 + Geno_Diet + Sex`
12. Volcano Plot Generation
13. Top 20 Metabolite Boxplots

The pooled analysis saves:
- `Pooled_top_boxplot.png`

#### Key Functions

**`plot_table_save(data)`**
- Purpose: Generates volcano plots, prints the plot, writes contrast-level results
- Outputs:
  - `Contrast_volcano.png`
  - `Contrast_results.csv`

**`plot_differences(df, level = "Pooled")`**
- Purpose: Selects top 20 metabolites and creates boxplots
- Outputs:
  - `Pooled_top_boxplot.png`

#### Generated Output Files

For each differential abundance contrast:
- Volcano plot: `*_volcano.png`
- Differential abundance results: `*_results.csv`

Other outputs:
- Top metabolite boxplots: `Pooled_top_boxplot.png`

Visual outputs rendered in HTML report:
- Missingness heatmap
- Missingness specificity result tables
- PCA plots
- Metabolite abundance heatmap
- Volcano plots
- Top metabolite boxplots

#### Notes and Assumptions

- The script is designed for metabolomics data from mouse HFD experiments
- The first column of the CSV must contain sample names
- Sample names must be consistently formatted so genotype, diet, and ID can be parsed
- Missing value imputation uses QRLIC from imputeLCMD
- Differential abundance uses limma with proper contrast specification

#### Reproducibility & Citation

Suggested references to cite in methods:
- **limma**: Ritchie et al. (2015) for differential abundance modeling
- **imputeLCMD**: Straube et al. for QRILC missing value imputation
- **brglm2**: Kosmidis & Firth for bias-reduced generalized linear models
- **emmeans**: Lenth (2024) for estimated marginal means and pairwise contrasts

---

### Proteomics Workflow

#### Input Data

The pipeline expects an Excel proteomics report, such as output from a protein quantification workflow or Spectronaut-style report.

Required input fields include:
- `PG.ProteinNames`
- `PG.Genes`
- `PG.ProteinDescriptions`
- Quantitative protein abundance columns

The current script assumes:
- Quantitative sample columns are located in columns 12–27 (change as needed in the script)
- Sample names contain encoded genotype and diet information via hyphens and underscores, e.g., `ID_genotype-diet_replicate`
- Genotype and diet can be parsed from the sample name structure

**Before running the analysis**, update the file path in the R script:
```r
file = "~/path/to/your/data.xlsx"
```

#### Required R Packages

**CRAN packages**:
- ggplot2
- ggforce
- dplyr
- ggrepel
- patchwork
- pheatmap
- readxl
- factoextra
- emmeans
- ggsci

**Bioconductor packages**:
- limma
- clusterProfiler
- org.Mm.eg.db
- imputeLCMD
- msigdbr
- brglm2

#### Workflow Summary

1. Data Import and Formatting
2. Missing Data Assessment
3. Missing Data Filtering (>30% missingness)
4. Missing Value Imputation (QRLIC)
5. Exploratory Analysis (PCA and heatmaps)
6. Differential Abundance Analysis (limma)
   - Model design: `~ 0 + Geno_Diet`
7. Volcano Plot Generation
8. Functional Enrichment Analysis (ORA and GSEA)
9. Top Protein Boxplots

#### Key Functions

**`plot_table_save(df)`**
- Purpose: Adds protein annotation, classifies significance/direction, generates volcano plots, saves results
- Outputs:
  - `*_volcano.png`
  - `*_results.csv`

**`plot_differences(df)`**
- Purpose: Generates boxplots for top proteins grouped by genotype and diet
- Outputs:
  - `*_top_boxplot.png`

**`run_overrep(df)`**
- Purpose: Performs GO over-representation analysis on significant proteins
- Outputs:
  - `*_overrep_plots.png`
  - `*_overrep_results.csv`

**`run_gsea(df)`**
- Purpose: Performs GSEA using ranked logFC values with GO Biological Process terms
- Outputs:
  - `*_gsea_plots.png`
  - `*_gsea_results.csv`

#### Generated Output Files

For each contrast:
- Volcano plot: `*_volcano.png`
- Differential abundance table: `*_results.csv`
- Over-representation plots: `*_overrep_plots.png`
- Over-representation results: `*_overrep_results.csv`
- GSEA plots: `*_gsea_plots.png`
- GSEA results: `*_gsea_results.csv`

Other outputs:
- PCA plot: `pc.tif`
- Top protein boxplots: `*_top_boxplot.png`

#### Notes and Assumptions

- This pipeline is designed for mouse proteomics data
- Gene annotation uses `org.Mm.eg.db`
- Protein-to-gene mapping depends on `PG.Genes` being present and formatted as mouse gene symbols
- Sample naming must be consistent for genotype and diet parsing to work correctly
- The input file column structure must match the script assumptions
- The script currently assumes quantitative columns are columns 12–27 (update as needed)
- Missingness specificity testing code is present but commented out to reduce run time
- The workflow uses FDR-adjusted p-values for significance calling
- Enrichment results depend strongly on gene symbol mapping quality

#### Reproducibility

The script sets:
```r
set.seed(777)
```

#### Reproducibility & Citation

Suggested references to cite in methods:
- **limma**: Ritchie et al. (2015) for differential abundance modeling
- **clusterProfiler**: Yu et al. for GO enrichment and GSEA
- **imputeLCMD**: Straube et al. for QRILC missing value imputation
- **org.Mm.eg.db**: Carlson M (2024) for mouse gene annotation

---

### RNA-seq Workflows

Bulk RNA-seq analyses use:
1. **Alignment & Counting**: Pre-processed read counts
2. **DESeq2 Analysis**: Differential expression with proper statistical modeling
3. **Visualization**: MA plots, volcano plots, heatmaps
4. **Enrichment Analysis**: GO terms, KEGG pathways, MSigDB signatures
5. **Interpretation**: Functional classification and biological insights

Each workflow follows the standardized Moore laboratory pipeline and generates publication-ready figures and statistical tables.

---

## Key Features

- ✅ **Reproducible**: Fully documented RMarkdown workflows with version-controlled environments
- ✅ **Multi-omics**: Includes analysis of metabolomics, proteomics, and transcriptomics
- ✅ **Statistical Rigor**: Proper statistical testing with multiple testing correction
- ✅ **Publication-ready**: High-quality visualizations suitable for figures
- ✅ **Flexible**: Modular design allows adaptation for new datasets
- ✅ **Documented**: Detailed comments and explanations throughout code

## Output

Each analysis generates:
- **HTML Report**: Interactive summary with embedded figures
- **Tables**: CSV files with statistical results
- **Plots**: Publication-quality visualizations (PNG, PDF, TIF)
- **Session Info**: R version and package information for reproducibility

## Troubleshooting

### Common Issues

**Package Installation Fails**
- Ensure R >= 4.0 is installed
- Update packages: `update.packages()`
- Try installing from Bioconductor: `BiocManager::install("package_name")`

**renv::restore() Takes Too Long**
- First restore is slower; subsequent restores are cached
- Use `renv::clean()` to remove old packages if disk space is limited

**Column Numbers Don't Match My Data**
- Update column indices in the script: `data <- read_xlsx(file, sheet = 1, range = cell_cols(12:27))`
- Check your Excel file structure with `readxl::excel_sheets()` and `readxl::read_xlsx(file, n_max = 1)`

**Sample Parsing Fails**
- Verify sample names follow the expected format: `ID_genotype-diet_replicate`
- Check for typos or inconsistent naming schemes
- Review the parsing logic in the "Metadata Parsing" section

## License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE](LICENSE) file for details.

## Citation

If you use these analysis workflows in your research, please cite:

```
Moore Laboratory Omics Analysis Workflows
https://github.com/Moore-Laboratory/Omics_YM
```

And include relevant method citations as noted in each workflow section.

## Contact

For questions or issues related to these analyses, please contact the Moore Laboratory or open an issue on the GitHub repository.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your improvements
4. Submit a pull request with a clear description of changes

## Version History

- **v1.0** (2026-05-11): Initial public release with metabolomics, proteomics, and RNA-seq pipelines

---

**Last Updated**: May 11, 2026
