# Encoding and Evolution

## Introduction

Backwards compatibility: Ensures that newer code can read data written by older code

Forwards compatibility: Ensures that older code can read data written by newer code.


If you want an older client to be able to successfully call a newer service, you need backward compatibility on the request and forward compatibility on the response. 
If you want a newer client to be able to call an older service, you need forward compatibility on the request and backwards compatibility on the response. 

Backwards compatibility is usually easier to achieve. You know the format of the existing data, so you can explicitly handle it. Forwards compatibility can be trickier - you require old code to ignore additions made by newer code.

## Formats for Encoding data

Translation from in-memory representation to a byte sequence is called encoding/serialisation/marshalling. The reverse - translation from a byte-sequence into an in-memory representation is called decoding/parsing/deserialisation/unmarshalling. 

Encoding/serialising: translate from an in-memory representation to a byte-sequence.
Decoding/deserialising: translate from a byte-sequence to an in-memory representation. 

Generally a bad idea to use your language's built-in encoding for anything other than very transient purposes.

JSON and XML are good for public-facing stuff, and are solid data-transferring formats since many people use them: 'The difficulty of getting different organisations to agree on anything outweighs most other concerns' (p166).

Binary schemas are introduced to solve the ineffiency problems of JSON/XML schemas. 

### Protobuf

Protocol Buffers (protobuf) is a binary encoding library developed at Google. 

DB nullable for new properties is the db equivalent of Kleppmann's advice on field tags.

### Avro

#### Writer's schema and Reader's schema

There is a difference between the writer's and reader's schema. To decode some data, an application uses 2 schemas:

writer's schema: identical to the one used for encoding
reader's schema: may be different. 

## Merits of schemas



