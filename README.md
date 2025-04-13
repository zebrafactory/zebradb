# Zebra DB

This project aims to build:

* A strongly typed, strictly validated document-oriented database

* With a super space efficient split-schema binary encoding

* That preserves history because it's built on a GIT-like object store

* Plus has quantum-secure transaction signing using [ZebraChain](https://github.com/zebrafactory/zebrachain)

* Uses WASM for secure, high-performance map/reduce under the hood (which means you can write map/reduce fuctions in any language that can compile to WASM)

* And should also have a proper query language, if only someone could design it (hint, hint)


## Split-schema binary encoding

Self describing data is great. Well defined schemas are also great.

The downside of document oriented databases is size (because the key names and types are stored
over and over). But as long as you are using the same well defined schema for a gazillion
documents, why not just store this schema once (in its own special document), and have all
instances of this type reference the schema document by its hash.

So a Zebra DB document looks like this:

```
SCHEMA_HASH || DOCUMENT_VALUES
```

There are only two kinds of "self description" left in `DOCUMENT_VALUES`: variable length values
must include a length prefix, and optional values must include a key. Everything else is in the
`SCHEMA_DOCUMENT`.

Required values come first, followed by optional values:

```
SCHEMA_HASH || REQUIRED_DOCUMENT_VALUES || OPTIONAL_DOCUMENT_VALUES
```

`REQUIRED_DOCUMENT_VALUES` can be variable or fixed length depending on whether it contains
any variable length values. `OPTIONAL_DOCUMENT_VALUES` is, by nature, always variable length.

Optional values start with a fixed-length key:

```
OPTION_KEY || OPTION_VALUE
```

`OPTION_KEY` is 1 byte if the schema has up to 256 optional values, 2 bytes for up to 2^16 optional
values, and so on.

`OPTIONAL_DOCUMENT_VALUES` are sorted by `OPTION_KEY`.