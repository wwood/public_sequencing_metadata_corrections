This repository contains corrections to metadata in public sequence data
repositories such as NCBI SRA.

It is still in early development, so if you find a mistake in the metadata and
want to contribute, then the easiest way is to create a [new GitHub issue](https://github.com/wwood/public_sequencing_metadata_corrections/issues/new).

These corrections will hopefully be applied at their source e.g. at NCBI, but
this is future work.

# Parsing NCBI SRA metadata

There is a parsing script in the `bin` directory, which can be used to clean up
some of the metadata, by correcting some common issues observed in the deposited
metadata fields.

The input to the script is a file with one line per SRA run, where each line is
JSON formatted and contains the `acc` and `jattr` columns. As an example, we ran this query at https://console.cloud.google.com/bigquery

Note that you need to change the name of the bucket from `my-bucket-name` to something you have access to.

```sql
EXPORT DATA
  OPTIONS( uri='gs://my-bucket-name/sra_metadata_test/*.json',
    format='JSON') AS
SELECT
  acc,
  jattr
FROM
  `nih-sra-datastore.sra.metadata`
WHERE
  librarysource = 'METAGENOMIC'
  AND platform = 'ILLUMINA'
    
limit 100;
```

This generated a single file which was downloaded to our local machine, in the `example_data` directory (after changing `my-bucket-name`):

```sh
gsutil -m cp -r gs://my-bucket-name/sra_metadata_test/*.json .
```

At this point, the files can be deleted from the bucket (e.g. from
https://console.cloud.google.com/storage), to avoid incurring storage costs.

Then setting up the parsing script's conda environment:
```sh
mamba env create -n public_sequencing_metadata_corrections -f env.yml
conda activate public_sequencing_metadata_corrections
```

and running the script:
```sh
bin/parse_biosample_extras.py --json-input example_data/000000000000.json > example_data/000000000000.parsed.tsv
```

This outputs a TSV file with corrected metadata. Extra biosample metadata
columns can be output - see the help message of the script for more details.

For instance these fields are parsed into standardised form:
```sh
bin/parse_biosample_extras.py --json-input example_data/000000000000.json --extra-sample-keys collection_date_sam depth_sam temperature_sam
```

Note that the corrections to these data are not currently applied, the parsing
and corrections are separate (at the moment - integration coming).
