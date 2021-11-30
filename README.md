# Prediction of alternative conformations using AlphaFold 2

This repository accompanies the manuscript ["Sampling the conformational landscapes of transporters and receptors with AlphaFold2"](https://www.biorxiv.org/content/10.1101/2021.11.22.469536v2) by Diego del Alamo, Davide Sala, Hassane S. Mchaourab, and Jens Meiler. The code used to generate these models can be found in `scripts/` and was derived from the closely related repository [ColabFold](https://github.com/sokrypton/ColabFold/). This repository also includes the scripts used to plot the data, which can be found in `figures/`, as well as scripts to analyze data `analyses_scripts`. Finally, a Google Colab notebook is available for use in `notebooks/`.

The model generation code does not change the underlying AlphaFold v2.0.1 prediction pipeline (multimer prediction is not currently supported - we are working on that!). Therefore, please follow the installation instructions provided by DeepMind and review the AlphaFold2 [license](https://github.com/deepmind/alphafold/blob/main/LICENSE) and [disclaimer](https://github.com/deepmind/alphafold#license-and-disclaimer) before use (additionally, please refer to the [AlphaFold FAQ](https://alphafold.ebi.ac.uk/faq) and [ColabFold FAQ](https://github.com/sokrypton/ColabFold/blob/main/README.md)). The objective of the code contained here is to provide access to otherwise hard-to-reach settings that facilitate the generation of conformationally heterogeneous models of protein structures. Genetic and/or structural databases *do not* need to be downloaded - everything is accessible through the cloud via the [MMseqs2 API](https://github.com/soedinglab/MMseqs2).

<p align="center"><img src="https://github.com/delalamo/af2_conformations/blob/9e247c321ebd82ed3716070094721077c5656d9d/figures/header/fig1_header.png" height="350"/></p>

*De novo* prediction of protein structures in multiple alternative conformations can usually be achieved for proteins that are absent from the AlphaFold2 training set, i.e. their structures were not determined prior to 30 April 2018. Predicting multiple conformations of proteins in the training set is, in our experience, sometimes but not usually possible.

We recommend sampling across several MSA depths. When MSAs are too shallow, the proteins are totally misfolded, whereas when they are too deep the models are conformationally uniform. The "Goldilocks range" of MSA depths that achieve the maximum number of correctly folded, but structurally diverse models seems to differ from protein to protein; in our experience, they appear to correlate with the number of amino acids. In any case, initial guesses for MSA depths can range 32-128 sequences for proteins absent from the training set and 8-64 sequences for proteins in the training set (note that this is much less than the 1000-5000 sequences that are used by AlphaFold2 by default). Once generated, these models can be analyzed using any dimensionality reduction and/or clustering algorithm; in our study we use PCA and focus mainly on the models at either extreme.

### How to use the code in this repository

Before importing the code contained in the `scripts/` folder, the user needs to install the AlphaFold source code and download the parameters to a directory named `params/`. Additional Python modules that must be installed include [Numpy](https://numpy.org/) and [Logging](https://abseil.io/docs/python/guides/logging).

The scripts can be imported and used out-of-the-box to fetch multiple sequence alignments and/or templates of interest:

```
from af2_conformations.scripts import predict
from af2_conformations.scripts import util
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

The following code then runs a prediction without templates. Note that the `max_msa_clusters` and `max_extra_msa` options can be provided to reduce the size of the multiple sequence alignment. If these are not provided, the networks default values will be used. Additional options allow the number of recycles, as well as the number of loops through the recurrent Structure Module, to be specified.

```
predict.predict_structure_no_templates( sequence, "out.pdb",
         a3m_lines, model_id = 1, max_msa_clusters = 16,
         max_extra_msa = 32, max_recycles = 1, n_struct_module_repeats = 8 )
```

To run a prediction with templates:

```
predict.predict_structure_from_templates( sequence, "out.pdb",
        a3m_lines, templates = pdbs, template_path = template_path,
        model_id = 1, max_msa_clusters = 16, max_extra_msa = 32,
        max_recycles = 1, n_struct_module_repeats = 8 )
```


