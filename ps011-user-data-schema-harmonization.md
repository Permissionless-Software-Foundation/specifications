# User Data Schema & Harmonization

## Permissionless Software Foundation Specification 011 (PS011)

### Specification version: 1.0.0

### Date originally published: July 29, 2025

### Date last updated: July 29, 2025

## Authors

Chris Troutner

## Acknowledgments

- This document builds on top of [PS002 SLP Mutable Data](./ps002-slp-mutable-data.md) and [PS007 Token Data Schema](./ps007-token-data-schema.md) specifications.

## 1. Introduction

Members of the PSF are rapidly building out new apps and features that leverage SLP token mutable data. This document exists to [harmonize](https://www.integrate.io/glossary/what-is-data-harmonization/) the use of the `userData` field in a tokens [mutable data](./ps007-token-data-schema.md) across several apps. Developers and businesses are invited to adopt the schema laid out in this document to encourage cross-capability between apps. This specification has already been implemented by the following web apps:

- [TokenTiger.com](https://tokentiger.com) is a user-friendly app for creating NFTs and fungible tokens. It provides a user-interface for editing the mutable data for a token.
- [SLP DEX](https://dex.psfoundation.info) is a DEX and social media platform for buying, selling, and curating tokens.
- [wallet.psfoundation.info](https://wallet.psfoundation.info) is an open source, non-custodial web wallet that supports SLP tokens.

## 2. Background

The [PS007 specification](./ps007-token-data-schema.md) includes a schema for mutable data attached to a SLP tokens. Within that schema is a `userData` field, which is a defined as an empty object (`{}`) by default.

[Token Tiger](https://tokentiger.com) is building out multiple user interfaces for editing token mutable data. In the simplest editor, the give the user the ability to edit the `userData` field of the mutable data. Multiple PSF stakeholders have agreed that this is the most obvious and convenient place to allow users to add structured, arbitrary data to their tokens.

The [SLP DEX](https://dex.psfoundation.info) is committed to supporting Token Tiger and future SLP token developers. This `userData` field can be leveraged by users of the DEX to attach rich media to their token. It can also be used to help categorize and tag the data in the token, allowing token creators to create a better user experience for consumers of the token.

## 3. Schema

Below is example JSON of the type of object that could fill the `userData` field of token mutable data:

```json
{
  "currentUrl": "https://slp-tokens.com",
  "media": [
    {
      "filename": "pfp-screenshot.png",
      "url": "https://bit.ly/4lOaCzD",
      "description": "A screenshot"
    }, 
    {
      "filename": "thumbnail-cat.png",
      "url": "https://bit.ly/4faTr8W",
      "description": "cat pic"
    }
  ],
  "markdown": "This is **markdown** formatted text.",
  "categories": ["art", "cats"],
  "tags": ["example", "cats", "screenshot"]
}
```

Each field should have the following types and default values:
- `Variable Name` - <type> - default value
- `currentUrl` - <string> - ''
- `media` - <array> - []
  - `filename` - <string> - ''
  - `url` - <string> - ''
  - `description` - <string> - ''
- `markdown` - <string> - ''
- `categories` - <array> - []
- `tags` - <array> - []