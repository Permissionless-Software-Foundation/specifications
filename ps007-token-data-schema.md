# Token Data Schema

## Permissionless Software Foundation Specification 007 (PS007)

### Specification version: 1.0.0

### Date originally published: September 21, 2022

### Date last updated: September 21, 2022

## Authors

Chris Troutner

## Introduction
As adoption of the [PS002 specification for token mutable data](../ps002-slp-mutable-data.md) grows, there is a need to define a schema for this token data, to ensure that tokens will be compatible with software across the industry. This specification document describes basic data structures that are recommended for use in token mutable data. It attempts to 'future proof' the schema by providing data fields where future growth can take place.

## Immutable Data
Because the immutable data of the token can not be modified after token creation, the recommended data for this section is limited. This data should be used as the creator sees fit, to capture any metadata about the token at the time of its creation. Here is the recommended *minimum data* for the immutable data field:

```json
{
  "schema": "ps007-v1.0.0",
  "issuer": "Permissionless Software Foundation",
  "website": "https://psfoundation.info",
  "dateCreate": "2022-09-21T13:46:19.651Z",
  "userData": {},
  "jsonLd": {}
}
```
### Schema (required)
Soem stuff

### Issuer (optional)
This property allows the software creating the token to declare the entity that has issued the token. This is different than the 'author', which should be captured in the `userData` property. This field is intended to reference the software (or organization that created the software) which created the token.

### Website
The website property

### Notes
Mutable data fields already used:
- tokenIcon
- about
- nsfw
- userData

I want to add a 'category' and 'tags' section.
Add a jsonLd field

Immutable data fields already used:
- userData
