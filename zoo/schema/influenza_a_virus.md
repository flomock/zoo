## Comments on schema for Influenza A virus

```
// Note: Each virus segment receives one document, because zoo is sequence 
// centric.

{
    "_id": null,
    "log": {
        // "first": null,
        // "mod": null,
        // "append": null // just append log stuff here
    },
    // logging stuff about who changed what when
    "metadata": {
        "ids": [], 
        // a list of alternative IDs, e.g. NCBI, GISAID, ...
        // e.g. {"source": "GenBank", "id": null}
        "description": "",
        "date": {
            "y": null, 
            "m": null, 
            "d": null
        },
        "geo": {
            "long": null, 
            "lat": null
        },
        "location": "",
        "host": "",
        // "processing": "", // any description concerning sample prep
        // "docs": [], // any articles, repos etc. the sequence is described in

    },
    "sequence": "",
    "derivative": {
        "minhash": [], // output from sourmash goes here, multiple k possible
        "msa": [
            {
                "id": null, 
                "gaps": {}
            }
        ]
        // "deopt": [] // list of SNPs that have been "deoptimized"
    },
    "relative": {
        "taxonomy": {
            // "order": "",
            // "family": "",
            // "subfamily": "", // e.g. coronaviruses
            // "genus": "",
            // "subtype": "", // not needed because nomenclature contains info
            "nomenclature": "", // A/duck/Ukraine/1/63(H3N8)
        },
        // a database collection represents a species, i.e. Influenza A virus
        "phylogeny": {
            "id": "" // the identifier of the tree node
        }
    },
    "annotation": [
        {
            "id": null, 
            "source": "", // GISAID, hand-crafted by John Doe, algo name, ...
            "name": "", // e.g. NA, PA, ...
            "fuzzy": null, // Boundaries fuzzy? See GenBank file docs.
            "start": null,
            "end": null
        } 
        // spliced entried receive two "lines", just like in BED format
    ],
    "links": {
        "int": [
            {
                "link": "",
                "type": "" // e.g. other segments, GISAID ID for complete set
            }
        ], 
        "ext": [
            {
                "link": "",
                "type": "" // "dat" link, "path" to tree/ MSA/ raw reads, ...
                // "description": "" // might be good, not sure
            }
        ]
    }
}
```