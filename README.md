## zoo

A portable datastructure for rapid prototyping in (viral) bioinformatics (under development). Instead of designing a database, we try to [grow](https://www.edge.org/conversation/brian_eno-composers-as-gardeners) one.

![](https://github.com/viehwegerlib/zoo/blob/master/zoo/img/cell.png)

### Install

Prerequisite: sourmash.

```
pip install git+https://github.com/dib-lab/sourmash.git@master
```

zoo itself:

```
# to use
pip install git+https://github.com/viehwegerlib/zoo.git@master
pip uninstall zoo

# to develop
git clone https://github.com/viehwegerlib/zoo
pip install -e zoo
```

Diffpatch of JSON objects is implemented in node. To install the required dependencies (assuming you have `node` and `npm` installed, e.g. with `brew` on the Mac.)

```
cd zoo/zoo/scripts
mkdir node_modules
npm install jsondiffpatch
npm install minimist
npm install mongodb
```

### Documentation

zoo provides a command-line tool as well as a Python library. For detailed information about zoo's intention, implementation, use cases and tutorials refer to the [wiki](https://github.com/viehwegerlib/zoo/wiki/Whitepaper). Documentation about zoo's functions and API is available at [readthedocs](https://readthedocs.org/).

### zoo CLI

```shell
# Create a JSON schema from zoo's templates to validate any data cell insertions.
zoo schema (core(metadata(influenza),annotation)) > schema.json
# Or use your own.
zoo schema --fp path/to/file (a(b,c)) > schema.json

# Stream GenBank records to data cell, and validate schema.
zoo load --source ncbi --fmt json \
--ids accessions.txt --stdout - | \
zoo init --db mockA --cell foo --validate schema.json -
# ... Initializing data cell.
# ... 42 entries inserted into cell "original".
# ... Primary key assigned to field "_id".
# ... inspect cell and commit

zoo status --db mockA --cell foo --example

# Make and commit changes (like you would with Git).
zoo commit --db mockA --cell foo original
# ... Dumping data cell.
# ... Minhash signature computed for molecule type: DNA
# ... | 42 Elapsed Time: 0:00:00
# ... Done.

# share
mkdir send
cp original.json send/
dat share send/
# ... Syncing Dat Archive: .../send
# ... Link: dat://73401e1b931164763eccsomelonglinkcefc718ebf49f6b4fe4dbad7

# In a faraway place, our collaborator (B) clones a copy of our cell and adds
# it to her "zoo" of other data cells.
mkdir receive
dat clone <link> receive/
zoo add --db mockB --cell foo --primkey genbank.accession receive/original.json
# ... Loading data cell.
# ... Index created on field "genbank.accession".
# ... 39 documents inserted in cell "foo".
# ... 3 duplicates skipped.

# Meanwhile, original.json was modified. B want his zoo to reflect the changes:
dat pull receive/

# diff it
zoo diff --db mockA --cell foo bar.json > diff.json
# ... Searching for changes (delta).
# ... Done.
# We can pipe this, too.
zoo diff --db mockA --cell foo bar.json | head -n2
# Apply changes to data cell.
zoo diff --patch --db mockA --cell foo diff.json
# ... Loading and applying delta.
# ... Done.

# pull
zoo pull --db mockB --cell foo receive/modified.json
# ... Updating cell's md5 hashes.
# ... / 0 Elapsed Time: 0:00:00
# ...
# ... 38 entries unchanged.
# ... 4 entries replaced.

# Now put data cells into your favourite analysis workflow, then use zoo's API
# to import/ export the results, like multiple sequence or reference-based
# alignments, phylogenetic trees, secondary structure ... happy exploratory
# data analysis. Also, set global vars to reduce typing.
ZOODB=mockB
ZOOCELL=foo
zoo digest --encode tree.nexus
zoo digest --decode msa.mafft.fa

# Not yet implemented: Send metadata about cell to a registry, so others can
# discover it.
zoo push ...

# Create a sequence Bloom tree (SBT) from the minhash signatures of a given
# cell.
zoo sbt_index --db ref --cell virus --ksize 16 --nsketch 1000 virusref
# ... Initialize SBT.
# ... Compute minhash signatures for selected documents.
# ... k-mer size: 16, sketch size: 1000
# ... \ 9158 Elapsed Time: 0:01:45
# ... Save SBT.
# ... Done.

# Use sourmash_lib to query other signatures about the cell's SBT.
sourmash sbt_search --ksize 16 virusref query.fa.sig
# ... running sourmash subcommand: sbt_search
# ... loaded query: survey.fa... (k=16, DNA)
# ... 0.11 0ef85591-d464-4953-915f-f673907b7e8e (here Zika reference genome)

# Export, e.g. to fasta, JSON or stdout.
zoo dump --query q.json --selection _id,meta.date,meta.geo.cou,seq \
--delim "|" --fmt fasta dump.fa
zoo dump --query q.json --selection _id,seq --fmt fasta - | grep ">" | head

# Pipe into sourmash.
zoo dump --query q.json --selection _id,seq --fmt fasta - | \
sourmash compute -k 16 -n 100 --singleton --out q.sig -

# Done, lets get some coffee.
zoo drop --db mockB --cell foo --force
zoo destroy --db mockB --force
```

### zoo API

zoo includes a Python library with an intuitive API and many functions to move data in and out of a data cell as well as for formatting the data for downstream analysis. The API covers amongst other things:

- [linked data](https://github.com/viehwegerlib/zoo/blob/master/examples/link.py), in the form of internal links to other documents or links to external files
- export to fasta, ...
- one-hot-encode the selected sequences for machine learning

### Sharing data

As illustrated above, [data cells](https://github.com/viehwegerlib/zoo/wiki/Whitepaper) are shared with the [dat protocol](https://github.com/datproject/dat). It couldn't be easier. Let's say you had a file `zika.json` with some experimental Zika data.

```shell
dat share .../zika_survey/  # contains zika.json
# Syncing Dat Archive: .../zika_survey
# Link: dat://ff92ce30e1ff6ebd75edeb42f04239367243a58b7838f50706bd995e5dbc5d4c
```

We can send this link to a colleague or put it in the zoo registry for others to find.

```shell
# meanwhile in a faraway place
dat clone ff92ce30e1ff6ebd75edeb42f04239367243a58b7838f50706bd995e5dbc5d4c
ls
# zika.json
```

### Scaling

zoo by design considers data to be an infinite stream, so scaling will become an issue, especially so if moving to larger organisms such as bacteria. Either the local storage capacity is exceeded or the analysis has too little resources. zoo offers convenient exports you can ship to larger compute environment.

```shell
# Not yet implemented: Port a zoo cell (batteries included).
zoo scale --container docker  # or rkt
```

### Tests

zoo adheres to pytest's package integration [guidance](http://doc.pytest.org/en/latest/goodpractices.html).

```shell
# cd into package directory and virtualenv (Python 3)
python setup.py test

# test Python 2.7 and 3.5
tox
```

### License

BSD-3-Clause

Copyright (c) 2017 Adrian Viehweger

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.801560.svg)](https://doi.org/10.5281/zenodo.801560)
