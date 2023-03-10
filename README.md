# Prediction of alternative conformations using AlphaFold 2

This repository if a fork of our previous work ["Sampling alternative conformational states of transporters and receptors with AlphaFold2"](https://elifesciences.org/articles/75751) by Diego del Alamo, Davide Sala, Hassane S. Mchaourab, and Jens Meiler. To have a general overview, please read the README.md at https://github.com/delalamo/af2_conformations. Here, our previous workflow is extended with the aim of 1. predict the intended conformational state of proteins by combining genetic information and template structures in the most effective way. In case of GPCRs, users can simply state which ativation state they want to predict and the script will combine the best templates found in GPCRdb with the MSA. 2. Refine a custom PDB to identify the closest energetic minima.

### How to use the code in this repository

Before importing the code contained in the `scripts/` folder, the user needs to install the AlphaFold source code and download the parameters to a directory named `params/`. Additional Python modules that must be installed include [Numpy](https://numpy.org/), [Requests](https://docs.python-requests.org/en/latest/), and [Logging](https://abseil.io/docs/python/guides/logging).

The scripts can be imported and used out-of-the-box to fetch multiple sequence alignments and/or templates of interest:

```python
from af2_conformations.scripts import mmseqs2

# Jobname for reference
jobname = 'T4_lysozyme'

# Amino acid sequence. Whitespace and inappropriate characters are automatically removed
sequence = ("MNIFEMLRIDEGLRLKIYKDTEGYYTIGIGHLLTKSPSLNAAKSELDKAIGRNCNGVIT"
            "KDEAEKLFNQDVDAAVRGILRNAKLKPVYDSLDAVRRCALINMVFQMGETGVAGFTNSL"
            "RMLQQKRWDEAAVNLAKSRWYNQTPNRAKRVITTFRTGTWDAYKNL" )
            
# PDB IDs, written uppercase with chain ID specified
pdbs = ["6LB8_A",
        "6LB8_C",
        "4PK0_A",
        "6FW2_A"]

# Initializes the Runner object that queries the MMSeqs2 server
mmseqs2_runner = mmseqs2.MMSeqs2Runner( jobname, sequence )

# Fetches the data and saves to the appropriate directory
a3m_lines, template_path = mmseqs2_runner.run_job( templates = pdbs )
```
To predict a specific activation state of a GPCR target, the pdbs list must contain one of the following string in the first position ("Inactive", "Active", "Intermediate"). The script will use the best four templates in the specified activation state retrieved from GPCRdb.org . PDB ids can be excluded simply adding those to the list without chain ID specified like ["Inactive", "7FII"] to predict the activation state of your target without using 7FII. Which PDB ids are used to bias the prediction can be retrieved from the loggin.info level. Here an example on outputting info into example.log and run a prediction of LSHR.

```
import logging
logging.basicConfig(filename='example.log', level=logging.DEBUG)


```

The following code then runs a prediction without templates. Note that the `max_msa_clusters` and `max_extra_msa` options can be provided to reduce the size of the multiple sequence alignment. If these are not provided, the networks default values will be used. Additional options allow the number of recycles, as well as the number of loops through the recurrent Structure Module, to be specified. In addition, ptm can be enabled to print pTM-score of model within file name. 

```python
from af2_conformations.scripts import predict

predict.predict_structure_no_templates( sequence, "out.pdb",
         a3m_lines, model_id = 1, max_msa_clusters = 16,
         max_extra_msa = 32, max_recycles = 1, n_struct_module_repeats = 8, ptm = True )
```

To run a prediction with templates:

```python
predict.predict_structure_from_templates( sequence, "out.pdb",
        a3m_lines, template_path = template_path,
        model_id = 1, max_msa_clusters = 16, max_extra_msa = 32,
        max_recycles = 1, n_struct_module_repeats = 8 , ptm = True )
```
To run a prediction with a custom PDB template:

```python
predict.predict_structure_from_custom_template( sequence, "out.pdb",
        a3m_lines, template_pdb = template_pdb,
        model_id = 1, max_msa_clusters = 16, max_extra_msa = 32,
        max_recycles = 1, n_struct_module_repeats = 8 , ptm = True )
```

There is also functionality to introduce mutations (e.g. alanines) across the entire MSA to remove the evolutionary evidence for specific interactions (see [here](https://www.biorxiv.org/content/10.1101/2021.11.29.470469v1) and [here](https://twitter.com/sokrypton/status/1464748132852547591) on why you would want to do this). This can be achieved as follows:

```python
# Define the mutations and introduce into the sequence and MSA
residues = [ 41,42,45,46,56,59,60,63,281,282,285,286,403,407 ]
muts = { r: "A" for r in residues }
mutated_msa = util.mutate_msa( a3m_lines, muts )
```

### Known issues

Here is a shortlist of known problems that we are currently working on:
* The MMSeqs2 server queries the PDB70, rather than the full PDB. This can cause some structures to be missed if their sequences are nearly identical to those of other PDB files.
* Multimer prediction is not currently supported.
* Custom MSAs are not currently supported.

If you find any other issues please let us know in the "issues" tab above.

### Citation

If the code in this repository has helped your scientific project, please consider citing our preprint:

```bibtex
@article {10.7554/eLife.75751,
article_type = {journal},
title = {Sampling alternative conformational states of transporters and receptors with AlphaFold2},
author = {del Alamo, Diego and Sala, Davide and Mchaourab, Hassane S and Meiler, Jens},
editor = {Robertson, Janice L and Swartz, Kenton J and Robertson, Janice L},
volume = 11,
year = 2022,
month = {mar},
pub_date = {2022-03-03},
pages = {e75751},
citation = {eLife 2022;11:e75751},
doi = {10.7554/eLife.75751},
url = {https://doi.org/10.7554/eLife.75751},
journal = {eLife},
issn = {2050-084X},
publisher = {eLife Sciences Publications, Ltd},
}
```
