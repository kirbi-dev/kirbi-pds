# `org.kirbi.catalog.book` Lexicon

An [AT Protocol Lexicon](https://atproto.com/specs/lexicon) for publishing
records that describe books, based on the 15 elements of the
[Dublin Core Metadata Element Set, Version 1.1](https://www.dublincore.org/specifications/dublin-core/dces/).

Schema file: [`lexicons/org/kirbi/catalog/book.json`](lexicons/org/kirbi/catalog/book.json)

| Property   | Value                                   |
|------------|-----------------------------------------|
| NSID       | `org.kirbi.catalog.book`                        |
| Type       | `record`                                |
| Record key | `tid` (timestamp identifier)            |
| Required   | `title`, `createdAt`                    |

> **Note on the NSID:** an NSID's authority (`org.kirbi.catalog`) must
> correspond to a domain name you control (here, `catalog.kirbi.org`, a
> subdomain of `kirbi.org`) so the Lexicon can be resolved and published.
> Resolution looks up a DNS TXT record at `_lexicon.catalog.kirbi.org`.
> Change the `id` if you use a different domain.

## Design decisions

1. **Field names are Dublin Core element names.** Each property has the same
   name as its `dc:` element, so mapping to and from Dublin Core
   (e.g. OAI-PMH `oai_dc`, RDF, HTML `<meta name="DC.title">`) is trivial.
2. **Repeatable elements are arrays.** In Dublin Core every element is
   optional and repeatable. Here, elements that commonly have several values
   (`creator`, `subject`, `language`, `identifier`...) are arrays, even though
   their names are singular. Elements that normally have one value for a
   book (`title`, `description`, `date`, `type`, `rights`) are single strings.
3. **Only `title` is required** among the Dublin Core elements: a book record
   without a title is not useful. `createdAt` is also required, following
   AT Protocol conventions; it is the only property that is not a Dublin Core
   element.
4. **Values are plain strings ("Simple Dublin Core").** Encoding schemes such
   as W3CDTF dates, MIME types, and URIs are recommended in the property
   descriptions, but not enforced, because Lexicon has no pattern validation
   and Dublin Core allows free text. The exceptions are `language`, which uses
   the Lexicon `language` format (BCP 47 tags), and `type`, which lists the
   DCMI Type Vocabulary as `knownValues`.
5. **Length limits** are set on all strings and arrays, as recommended for
   Lexicons. `maxGraphemes` limits what users see; `maxLength` (in UTF-8
   bytes) is roughly 10× larger to accommodate any script.

## Properties

| Property      | DC element       | Type               | Limits                     | Recommended encoding |
|---------------|------------------|--------------------|----------------------------|----------------------|
| `title`       | `dc:title`       | string, required   | 300 graphemes              | — |
| `creator`     | `dc:creator`     | array of string    | 50 items × 100 graphemes   | `Family, Given` |
| `subject`     | `dc:subject`     | array of string    | 50 items × 100 graphemes   | Controlled vocabulary (LCSH, DDC...) or keywords |
| `description` | `dc:description` | string             | 3000 graphemes             | — |
| `publisher`   | `dc:publisher`   | array of string    | 10 items × 100 graphemes   | — |
| `contributor` | `dc:contributor` | array of string    | 50 items × 100 graphemes   | `Family, Given` |
| `date`        | `dc:date`        | string             | 64 bytes                   | W3CDTF: `YYYY`, `YYYY-MM`, `YYYY-MM-DD` |
| `type`        | `dc:type`        | string             | 128 bytes; default `Text`  | DCMI Type Vocabulary |
| `format`      | `dc:format`      | array of string    | 10 items × 100 graphemes   | MIME type, or physical medium |
| `identifier`  | `dc:identifier`  | array of string    | 20 items × 1000 bytes      | URI (`urn:isbn:...`, DOI URL...) |
| `source`      | `dc:source`      | array of string    | 20 items × 1000 bytes      | URI |
| `language`    | `dc:language`    | array of language  | 10 items                   | BCP 47 tag (enforced) |
| `relation`    | `dc:relation`    | array of string    | 50 items × 1000 bytes      | URI, including `at://` URIs |
| `coverage`    | `dc:coverage`    | array of string    | 20 items × 100 graphemes   | Place names, periods |
| `rights`      | `dc:rights`      | string             | 1000 graphemes             | Statement or license URI |
| `createdAt`   | —                | datetime, required | —                          | RFC 3339 (enforced) |

### `title` — *dc:title*

A name given to the resource. Include the subtitle after a colon if desired:
`"Fluent Python: Clear, Concise, and Effective Programming"`.

### `creator` — *dc:creator*

Entities primarily responsible for making the resource, usually the authors,
in the order they appear on the title page.

### `subject` — *dc:subject*

The topics of the book: keywords, key phrases, or classification codes.
Using a controlled vocabulary such as the Library of Congress Subject
Headings (LCSH) or Dewey Decimal Classification (DDC) improves discovery.

### `description` — *dc:description*

An account of the resource: an abstract, a table of contents, or a
free-text summary such as back-cover copy.

### `publisher` — *dc:publisher*

Entities responsible for making the resource available, e.g.
`"O'Reilly Media"`.

### `contributor` — *dc:contributor*

Entities that contributed to the resource but are not primary creators:
editors, translators, illustrators, authors of forewords, etc.

### `date` — *dc:date*

A point or period of time associated with an event in the lifecycle of the
resource, normally the publication date of the edition described.
Use [W3CDTF](https://www.w3.org/TR/NOTE-datetime) with only as much
precision as is known: `"1851"`, `"2022-04"`, or `"2022-04-12"`.
This is a plain string, not a Lexicon `datetime`, because `datetime` requires
a full timestamp, and many book dates are known only to the year.

### `type` — *dc:type*

The nature or genre of the resource. Recommended values come from the
[DCMI Type Vocabulary](https://www.dublincore.org/specifications/dublin-core/dcmi-type-vocabulary/):
`Collection`, `Dataset`, `Event`, `Image`, `InteractiveResource`,
`MovingImage`, `PhysicalObject`, `Service`, `Software`, `Sound`,
`StillImage`, `Text`. Books are `Text`, which is the default.
`knownValues` is an open set, so other values are allowed.

### `format` — *dc:format*

File format, physical medium, or dimensions. Use MIME types for digital
editions (`"application/epub+zip"`, `"application/pdf"`) and free text for
physical ones (`"hardcover"`, `"paperback"`, `"23 cm"`).

### `identifier` — *dc:identifier*

Unambiguous references to the resource. Prefer URIs so the kind of
identifier is explicit:

- ISBN: `urn:isbn:9781492056355`
- DOI: `https://doi.org/10.xxxx/yyyy`
- Wikidata: `https://www.wikidata.org/entity/Q...`
- Open Library: `https://openlibrary.org/books/OL...M`

### `source` — *dc:source*

Related resources from which this one is derived, e.g. the original work of
a translation, or the print edition an e-book was scanned from.

### `language` — *dc:language*

Languages of the resource's content as
[BCP 47](https://www.rfc-editor.org/info/bcp47) tags: `"en"`, `"pt-BR"`.
The PDS rejects malformed tags.

### `relation` — *dc:relation*

Related resources: other editions, other volumes in a series, companion
websites. Other `org.kirbi.catalog.book` records can be referenced by their
`at://` URI, e.g. `at://did:plc:abc123/org.kirbi.catalog.book/3l2k...`.

### `coverage` — *dc:coverage*

The spatial or temporal topic of the resource: place names
(`"Brazil"`), periods (`"20th century"`), or jurisdictions.

### `rights` — *dc:rights*

Information about rights held in and over the resource: a copyright
statement (`"© 2022 Luciano Ramalho"`) or a license URI
(`"https://creativecommons.org/licenses/by/4.0/"`).

### `createdAt`

Client-declared timestamp of when this record was created, in RFC 3339
format. This is not a Dublin Core element; it is the AT Protocol convention
for records, used for sorting in feeds and indexes. It is **not** the
publication date of the book — use `date` for that.

## Example record

```json
{
  "$type": "org.kirbi.catalog.book",
  "title": "Fluent Python: Clear, Concise, and Effective Programming",
  "creator": ["Ramalho, Luciano"],
  "subject": ["Python (Computer program language)", "Computer programming"],
  "publisher": ["O'Reilly Media"],
  "date": "2022-04",
  "type": "Text",
  "format": ["paperback"],
  "identifier": ["urn:isbn:9781492056355"],
  "language": ["en"],
  "createdAt": "2026-09-28T17:30:00.000Z"
}
```

A minimal valid record needs only `title` and `createdAt`:

```json
{
  "$type": "org.kirbi.catalog.book",
  "title": "Moby-Dick; or, The Whale",
  "createdAt": "2026-09-28T17:30:00.000Z"
}
```

## Publishing a record

Records are written to a user's repository with
`com.atproto.repo.createRecord`:

```sh
curl -X POST "https://$PDS/xrpc/com.atproto.repo.createRecord" \
  -H "Authorization: Bearer $ACCESS_JWT" \
  -H "Content-Type: application/json" \
  -d '{
    "repo": "'"$DID"'",
    "collection": "org.kirbi.catalog.book",
    "record": {
      "$type": "org.kirbi.catalog.book",
      "title": "Moby-Dick; or, The Whale",
      "creator": ["Melville, Herman"],
      "date": "1851",
      "createdAt": "2026-09-28T17:30:00.000Z"
    }
  }'
```

The response contains the record's `at://` URI and CID. The PDS assigns a
TID as the record key.

## Validating

The schema and sample records can be checked with the reference
implementation, [`@atproto/lexicon`](https://www.npmjs.com/package/@atproto/lexicon):

```js
import { Lexicons } from '@atproto/lexicon'
import fs from 'node:fs'

const lex = new Lexicons()
lex.add(JSON.parse(fs.readFileSync('lexicons/org/kirbi/catalog/book.json', 'utf8')))
lex.assertValidRecord('org.kirbi.catalog.book', record) // throws if invalid
```

## References

- [Dublin Core Metadata Element Set, Version 1.1](https://www.dublincore.org/specifications/dublin-core/dces/)
- [DCMI Type Vocabulary](https://www.dublincore.org/specifications/dublin-core/dcmi-type-vocabulary/)
- [AT Protocol Lexicon specification](https://atproto.com/specs/lexicon)
- [AT Protocol Record Keys](https://atproto.com/specs/record-key)
- [Namespaced Identifiers (NSIDs)](https://atproto.com/specs/nsid)
