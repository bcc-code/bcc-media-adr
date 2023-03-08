# Elasticsearch

This is a record of our decisions when researching Elasticsearch to consider replacing existing search engines with this
solution, to prepare for document and article searches.

## Indexing

### Options

This is an example of indexing options that will support different language stemmers, etc.

```json5
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default-no": {
          "tokenizer": "standard",
          "char_filter": [
            "segment_code"
          ],
          "filter": [
            "norwegian_stemmer",
            "asciifolding"
          ]
        },
        "default-en": {
          "tokenizer": "standard",
          "char_filter": [
            "segment_code"
          ],
          "filter": [
            "english_stemmer",
            "asciifolding"
          ]
        }
      },
      "filter": {
        // stemmers remove endings in words, so that for example "gjerrighet" turns into "gjerrig" 
        // in both indexes and queries
        "norwegian_stemmer": {
          "type": "stemmer",
          "language": "norwegian"
        },
        "english_stemmer": {
          "type": "stemmer",
          "language": "english"
        }
      },
      "char_filter": {
        // filters segment codes away from the string
        "segment_code": {
          "type": "pattern_replace",
          "pattern": "(\\{\\d*\\})",
          "replacement": "",
          "all": true
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "text": {
        "properties": {
          "no": {
            "type": "text",
            "analyzer": "default-no",
            "term_vector": "with_positions_offsets"
          },
          "en": {
            "type": "text",
            "analyzer": "default-en",
            "term_vector": "with_positions_offsets"
          }
        }
      }
    }
  }
}

```

### Document preparation

We start with an export from Whisper AI which looks like this:

```json5
{
  "text": "{{complete_text}}",
  "segments": [
    {
      "id": 0,
      "seek": 0,
      "start": 0.0,
      "end": 17.0,
      "text": "{{segment_text}}",
      "tokens": [
        0,
        1
      ],
      "temperature": 0.0,
      "avg_logprob": -0.25,
      "compression_ratio": 1.2,
      "no_speech_prob": 0.62
    },
    //...
  ],
  "language": "{{language_code}}"
}
```

We ignore the complete text field and use the segments to create a highlight-friendly indexable document:

```json5
{
  "text": {
    //... {*segment_code*} {{segment_text}} ...
    "no": " {0} segment text {1} segment text, for id 2. {3} segment text 3",
    "en": "... {125} more text {126} ..."
  }
}
```

We can then parse the highlight response:

```json5
// result from a query with "gjerrighet", query documented below
{
  "_index": "{{index_id}}",
  "_id": "{{document_id}}",
  "_score": 22.52314,
  "highlight": {
    "text.no": [
      "for eksempel <em>gjerrig</em>? {227} Ja, det er jo onde ånder som er beseiret deg. {228} Alle <em>gjerrige</em> mennesker ",
      " best, han hadde sin egen sønn. {231} Så er du <em>gjerrig</em> og tenker på deg selv. {232} Det er onde ånder ",
      "grinete, {401} mener det og mener det. {402} Det er <em>gjerret</em>. {403} Så kommer livet deres fram. {404} Ja. {405",
      "få en ekstra herlig lønn av Herren. {19} Jeg vil <em>gjerne</em> lese litt i... {20} Skal vi se i Johannes. {21}",
      "dest mer du får for deg selv, {36} dest mer vil du <em>gjerne</em> ha, {37} og dest mindre føler du kjærlighet og "
    ]
  }
}
```

And retrieve the segment ids from the responses, and use the start times stored on the segment to decide where the user
should be sent when clicking it.

## Search Query

### Cross-language search

```json5
{
  "_source": false,
  // avoid using more network bandwidth than necessary
  "query": {
    "bool": {
      "should": [
        {
          "multi_match": {
            "query": "{{query}}",
            "boost": 3,
            // boost this with a higher factor to prioritize exact matches
            "fields": [
              "text.no",
              "text.en"
            ]
          }
        },
        {
          "multi_match": {
            "query": "{{query}}",
            "fuzziness": "AUTO",
            "boost": 2,
            // boosts this with a lower factor to prioritize matches
            "fields": [
              "text.no",
              "text.en"
            ]
          }
        }
      ]
    }
  },
  "size": 10,
  "from": 0,
  "highlight": {
    "order": "score",
    "phrase_limit": "1024",
    "type": "fvh",
    "boundary_scanner": "word",
    "fields": {
      // we can specify additional options here, but probably not needed
      "text.no": {},
      "text.en": {}
    }
  }
}
```

This query doesn't factor in primary or secondary languages, but can probably be done with a filter query, and boosted
based on some property. Performance impact from making a more complex query is yet to be determined, but likely not a
blocker.

## Data integrity

We won't need to consider the indexes important, as we should be able to just rebuild the datasets and clusters with
minimal effort. We can consider the data integrity in our search engine as non-important, and rather just rebuild the
indexes whenever something happens.

### Scale

Based on some minor tests, we likely won't be needing a beast of a cluster to serve our current dataset. A cluster
with 3 nodes with 1 GB of RAM will most likely be sufficient, at least for now. Scaling up shouldn't be an issue either.
As stated above, data integrity is not a priority, so if something fails, we shouldn't need to hesitate to just rebuild
the cluster from scratch.

