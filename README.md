# BtaSBML2964

Scripts for the reconstruction and analysis of **BtaSBML2964**, the first genome-scale metabolic model (GEM) of *Bos taurus*, comprising 2,964 genes, 13,278 reactions, and 8,652 metabolites. The model was reconstructed using [Human1](https://github.com/SysBioChalmers/Human-GEM) as a template and integrated with multi-omics data from Hanwoo (*B. taurus coreanae*) muscle satellite cells (MuSCs) for cultured meat applications.

This repository accompanies the manuscript *"Genome-scale metabolic reconstruction of Bos taurus and multi-omics analysis of Hanwoo muscle satellite cells to advance cultured meat production."*

## Installation guide

```bash
# 1. Clone the repository
git clone https://github.com/kaist-sbml/BtaSBML2986.git
cd BtaSBML2964

# 2. Create the environment and install dependencies
conda create -n cobra python=3.8
conda activate cobra
pip install cobra biopython pandas numpy requests scipy==1.10.1 memote==0.17.0 umap-learn pydeseq2
```

## Reconstruction workflow

## Extraction of *Bos taurus* metabolic information from KEGG

The notebook `cow_info_extraction.ipynb` retrieves the full set of *B. taurus* genes from the [KEGG REST API](https://rest.kegg.jp/) (organism code `bta`). For each gene, the corresponding KEGG Orthology (KO, "K number") is obtained, and the metabolic reactions associated with each KO are collected. The resulting gene–reaction and gene–KO mappings are saved as `gene_reaction.pickle` and `gene_k_number.pickle`.

## Building the *Bos taurus* protein sequence set

The notebook `cow_genes_fasta_make.ipynb` parses the *B. taurus* GenBank file (`genomic.gbk`) with [Biopython](https://biopython.org/), extracts the protein translations of all CDS features, and writes them to a FASTA file (`cow_gene_seq.fasta`). This file serves as the query/target for the homology search.

## Ortholog mapping by bidirectional BLASTP

The notebook `find_biblastp_match.ipynb` identifies one-to-one orthologs between *B. taurus* and human. It reads the BLASTP results (`cow_query.tsv`, `human_query.tsv`; e-value cutoff 1e-30), keeps the best hit (lowest e-value) for each query in both directions, and retains only **bidirectional best hits** (reciprocal best matches). The resulting human–cow ortholog dictionary is saved as `biblastp_match.pickle`.

## Converting the human template into a draft *Bos taurus* model

The notebook `change_gene_name.ipynb` loads the Human-GEM template and performs single-gene deletion analysis to identify essential human genes (those whose removal reduces the objective value below a threshold). Human genes are then renamed to their *B. taurus* orthologs using the bidirectional best-hit mapping; non-essential human genes without a cow ortholog are removed from the model. The resulting draft model is saved as `change_gene_name_cow.xml`.

## Identifying species-specific metabolites and reactions to add

The notebook `addition_list_identify.ipynb` determines which *B. taurus*–specific metabolites and reactions should be added to the draft model. It produces the input lists used in the next step — `metabolite_add`, `metabolite_formula`, `metabolite_overlap`, `reaction_add`, and `reaction_overlap` — which specify the metabolites to be added (with their formulas), the mapping of metabolites that overlap with those already present in the model, and the reactions to be added (with their overlaps).

## Adding species-specific metabolites, reactions, and GPR associations

The notebook `add_gene_reaction_metabolite.ipynb` loads the draft model (`change_gene_name_cow.xml`) and incorporates *B. taurus*–specific metabolic content derived from KEGG:
- adds cytosolic copies of metabolites that are required by new reactions but missing the cytosolic compartment, along with the necessary transport/demand reactions;
- adds new metabolites not present in the human template (with cytosolic and extracellular forms) together with their transport and exchange reactions, retrieving names and formulas from the KEGG;
- adds new KEGG reactions (with stoichiometry, names, and subsystems) to the model; and
- updates gene–protein–reaction (GPR) rules by attaching the corresponding *B. taurus* genes to each added reaction.

The output is `cow_add_information.xml`.

## Mass/charge balancing

The notebooks `addition_hydrogen_for_mass_balance.ipynb` perform mass-balance correction (e.g. adding protons/hydrogen to balance reactions).

## Workflow order

1. `cow_info_extraction.ipynb`
2. `cow_genes_fasta_make.ipynb`
3. `find_biblastp_match.ipynb`
4. `change_gene_name.ipynb`
5. `addition_list_identify.ipynb`
6. `add_gene_reaction_metabolite.ipynb`
7. `addition_hydrogen_for_mass_balance.ipynb`

