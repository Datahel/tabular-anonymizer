# Tabular Anonymizer

Anonymization and pseudonymization tools for tabular data. 


This library provides tools and methods for anonymization and privacy protection of data in Pandas DataFrame format.

## Installation

### Using pip-tools

To install this package using [pip-tools](https://pypi.org/project/pip-tools/1.8.0/):

Add `-e https://github.com/Datahel/tabular-anonymizer.git#egg=tabular_anonymizer` to your `requirements.in`

Run:

        $ pip-compile --generate-hashes --allow-unsafe -o requirements.txt requirements.in
        $ pip-sync requirements.txt

### Using pip

To install this package using [pip](https://pip.pypa.io/en/stable/):

Run:

        $ pip install git+https://github.com/Datahel/tabular-anonymizer.git
        
### Git clone + pip (if you want to inspect the examples)

You can alternatively clone this repository and install library from local folder with pip using -e flag:

        $ git clone https://github.com/Datahel/tabular-anonymizer.git
        $ pip install -e tabular-anonymizer

You can then try out the examples found under `examples/` folder. 

## Usage

### Anonymization

DataFrameAnonymizer anonymization functionality supports K-anonymity alone or together with L-diversity or T-closeness 
using Mondrian algorithm.

#### K-Anonymity in practice

In this simplified example there is mock dataset of 20 persons about their age, salary and education. 
We will anonymize this using mondrian algorithm with K=5.

![Dataframe before anonymization](documents/mondrian_data.png?raw=true "Dataframe")

After mondrian partitioning process (with K=5), data is divided to groups of at least 5 using age and salary as dimensions.

![Values after mondrian partitioning](documents/mondrian_plot.png?raw=true "Partitioned data")

In anonymization process a new dataframe is constructed and groups are divided to separate rows by sensitive attribute (education). 

![Anonymized dataset](documents/mondrian_anonymized.png?raw=true "Anonymized data with K=5")

You can test this in practice with: examples/plot_partitions.py 

#### Example: K-Anonymity using DataFrameAnonymizer

    import pandas as pd
    from tabular_anonymizer import DataFrameAnonymizer

    # Setup dataframe
    df = pd.read_csv("./adult.csv", sep=",")
    
    # Define sensitive attributes
    sensitive_columns = ['label']

    # Anonymize dataframe with k=10
    p = MondrianAnonymizer(sensitive_columns)
    df_anonymized = p.anonymize_k_anonymity(df, k=10)

#### Example: K-Anonymity with L-diversity using DataFrameAnonymizer

    import pandas as pd
    from tabular_anonymizer import DataFrameAnonymizer

    # Setup dataframe
    df = pd.read_csv("./adult.csv", sep=",")
    
    # Define sensitive attributes
    sensitive_columns = ['label']

    # Anonymize dataframe with k=10
    p = MondrianAnonymizer(sensitive_columns)
    df_anonymized = p.anonymize_l_diversity(df, k=10, l=2)


### Pseudonymization

Pseudonymization tool is intended for combining data from multiple sources. Both datasets share an identifier column. The function `combine_and_pseudonymize` replaces the identifier with a hash.

![Dataframe before pseudonymization](documents/pseudonymization_before.png?raw=true "Dataframe")

![Dataframe before pseudonymization](documents/pseudonymization_after.png?raw=true "Dataframe after pseudonymization of education column")

![Encryption process](documents/pseudonymization_encryption.png?raw=true "Pseudonymization and ecryption process")



#### Example: Pseudonymization of dataframe column with generated secret key

    from tabular_anonymizer import utils
    file1 = "exampples/adult.csv"
    df = pd.read_csv(file1, sep=",", index_col=0)
    # Simple way
    utils.pseudonymize(df, 'column_name', generate_nonce=True)

#### Example: Pseudonymization of multiple dataframes and columns shared secret key

    # Let's assume we have two dataframes df1 and df2. 
    # Both dataframes have common identifier data in columns column_name1 and column_name2, for example birth date
    # If you want to merge these datasets for example you can encrypt both columns using shared salt before that. 

    from tabular_anonymizer import utils

    # Generate nonces to be used as salt
    nonce1 = utils.create_nonce() # Generated random salt #1
    nonce2 = utils.create_nonce() # Generated random salt #2

    # Pseudonymize given columns using sha3_224 with two salts
    utils.pseudonymize(df1, 'column_name1', nonce1, nonce2)
    utils.pseudonymize(df2, 'column_name2', nonce1, nonce2)

#### Example: Combining two datasets with shared common column with pseudonymization

    # Let's assume that dataframes df1 and df2 have equal size and common column "id" which is direct identifier 
    # (such as phonenumber). We can combine (merge) these two datasets and pseudonymize values in ID-column
    # so it is no longer sensitive information.

    from tabular_anonymizer import utils

    # combine (merge) two datasets with common index column and pseudonymize
    df_c = utils.combine_and_pseudonymize(df1, df2, 'id')

#### Example: Post-processing & partial masking

    # Convert intervals to partially masked ['20220', '20210'] => '202**'
    generalize(df, 'zip', generalize_partial_masking)

    # Original table
    #                               id|    zip
    #                               1 | '20220'
    #                               2 | '20210'
    # Anonymized table (K=2)
    #                               zip
    #                               ['20220', '20210']
    # After partial masking
    #                               zip
    #                               '202**'

### Example jupyter notebooks

Besides example scripts, there are Jupyter notebooks can be found in examples-folder for testing purposes. 

    examples/sample_notebook.ipynb # Example how to use tabular anonymizer
    examples/check_anonymity.ipynb # Example for validating anonymizer results

#### Run notebooks in GitHub Codespaces

If you use GitHub codespaces, you can execute example scripts directly in VSCode browser interface. Required plugins are included in codespaces container configuration. 

#### Running Jupyter lab as server in codespaces

Codespaces allows you to run notebooks directly in we interface. However, if you need to run jupyter-lab server in codespaces, follow these instructions.

1. Start jupyter-lab server in codespaces terminal using following command :

        jupyter-lab --ip 0.0.0.0 --config .devcontainer/jupyter-server-config.py --no-browser

2. Observe jupyter-lab server log and click link pointing to 127.0.0.1, eg: http://127.0.0.1:8888/lab?token=... A small popup with link titled "Follow link using forwarded port" appears. Click the link and codespaces will redirect you to Jupyterlab user interface.  

![Jupyter_server_codespaces](documents/jupyterlab_codespaces.png?raw=true "JupyterLab in codespaces")

#### Run examples and jupyterlab in local docker environment

You can run Jupyterlab and do experiments with tabular anonymizer in docker container:

     docker build . -t tabular-anonymizer && docker run --rm -it -p 8888:8888 tabular-anonymizer  

Open http://127.0.0.1:8888 in your web browser and navigate to examples/sample_notebook.ipynb

Hit ctrl + c to quit container.


## Acknowledgements

Mondrian algorithm of this library is based on [glassonion1/AnonyPy](https://github.com/glassonion1/anonypy) mondrian implementation. 

Visualization example (plot_partitions.py) is based on Nuclearstar/K-Anonymity plot implementation. [Nuclearstar/K-Anonymity](https://github.com/Nuclearstar/K-Anonymity)
