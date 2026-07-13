# Data

Raw data is not committed to this repository.

## Source

Download the freMTPL2freq dataset from [OpenML](https://www.openml.org/d/41214).

Save the downloaded `.arff` file to this folder (`data/raw/ACQsci.arff`), then run:

    python src/load_data.py

This will parse the ARFF file and save a cleaned parquet file to `data/processed/freMTPL2freq.parquet`.

## Dataset details

- Name: freMTPL2freq
- Source: French motor third-party liability insurance portfolio
- Rows: 678,013 policies
- Format: ARFF (36MB)
- Reference: Noll, A., Salzmann, R. and Wuthrich, M.V. (2020) Case study: French motor third-party liability claims. SSRN Working Paper 3164764.
