# Token Data Schema

## Permissionless Software Foundation Specification 007 (PS007)

### Specification version: 1.0.1

### Date originally published: September 21, 2022

### Date last updated: September 29, 2022

## Authors

Chris Troutner

## Introduction
As adoption of the [PS002 specification for token mutable data](../ps002-slp-mutable-data.md) grows, there is a need to define a schema for this token data, to ensure that tokens will be inter-compatible with software across the industry. This specification document describes basic data structures that are recommended for use in token data. It attempts to 'future proof' the schema by providing data fields where future growth can take place.

## Immutable Data
Because the immutable data of the token can not be modified after token creation, the recommended data for this section is limited. Here is the recommended *minimum data* for the immutable data field:

```json
{
  "schema": "ps007-v1.0.1",
  "dateCreated": "2022-09-21T13:46:19.651Z",
  "userData": {},
  "jsonLd": {},
  "issuer": "Permissionless Software Foundation",
  "issuerUrl": "https://psfoundation.info",
  "license": ""
}
```

### schema (required)
The `schema` property indicates the version of this specification that the token data follows. This allows software to know how to process the rest of the data that follows. For that reason, this field is required.

### dateCreated (required)
The `dateCreated` property should be a date string in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format. This can be easily generated in JavaScript with the [Data.prototype.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString) method.

*Note:* This field is somewhat redundant, since the block height of when the token was created will also capture a timestamp. The `dateCreated` property provides a more convenient way to capture this timestamp. The `dateCreated` property should not diverge very far from the implicit timestamp of the block in which the transaction is mined.

### userData (required)
The `userData` property is intended to 'future proof' this specification by allowing a field for users to place arbitrary data. If no data is provided, this property should resolve into an empty object (`{}`).

### jsonLd (required)
The `jsonLd` property is another 'future proof' property. It's intended to capture [JSON-LD](https://json-ld.org/) structured data following [Schema.org](https://schema.org/) schema. The recommended schema model is the [CreativeWork](https://schema.org/CreativeWork) model. If no JSON-LD data is expressed, this property should resolve into an empty object (`{}`).

### issuer (optional)
This property allows the software creating the token to declare the entity that has issued the token. This is different than the 'author', which should be captured in the `userData` or `jsonLd` property. This field is intended to reference the software (or organization that created the software) which created the token.

### issuerUrl (optional)
The `issuerUrl` allows the Issuer to add a URL for additional information about them.

### license (optional)
The `license` property can be used to express any licensing information applied to the token. It can be a string or an object. As a string, it can express the license completely, or provide a URL to the license data. As an object, it can express the license as structured data.

## Mutable Data
Because the mutable data field of a token can change over time, the risks are much lower in specifying schema, and the data can be much more expressive. The properties listed below are an example of the *minimal* amount of data that is recommended for creating a token.

```json
{
  "schema": "ps007-v1.0.1",
  "tokenIcon": "https://bafybeicvlcwv3flrwa4egmroyicvghevi6uzbd56drmoerjeguu4ikpnhe.ipfs.dweb.link/psf-logo.png",
  "fullSizedUrl": "",
  "nsfw": false,
  "userData": {},
  "jsonLd": {},
  "about": "This is a short sentence about the token",
  "category": "art",
  "tags": ["logo", "psf", "software"],
  "mediaType": "image",
  "currentOwner": {},
}
```

### schema (required)
The `schema` property indicates the version of this specification that the token data follows. This allows software to know how to process the rest of the data. For that reason, this field is required. Note that it is OK for the mutable data to specify a different version than the immutable data. Since mutable data can change over time, it's anticipated that both the data and schema version will be updated over time to reflect changes to this specification.

### tokenIcon (required)
Tokens are generally expected to have a visual icon associated with them. For that reason, this field is required. However, if the issuer does not want to associate an icon with the token, this field can resolve into an empty string (`""`).

### fullSizedUrl (required)
Tokens are often used to represent rich media such as videos, 3D-objects, or high-resolution images. It is not appropriate to use this media as the token icon, because the large size of the data will create a poor user experience when viewing the token in a wallet. The `fullSizedUrl` field is used to link to this large-data resource. If not used, it should resolve into an empty string (`""`).

This property can also be an object, so that it can communicate links to various platforms where the content can be found. If expressed as an object, the object should contain a `default` field at a minimum, which provides views a 'default' URL to express. Here is an example:

```json
{
  ...
  "fullSizedUrl": {
    "default": "https://youtu.be/Xbpa3t58nzA",
    "youtube": "https://youtu.be/Xbpa3t58nzA",
    "rumble": "https://rumble.com/v14yior-introduction-to-nfts-on-bch.html",
    "odysee": "https://odysee.com/@trout:5/introduction-to-nfts-on-bch:1",
    "lbry": "lbry://@trout#5/introduction-to-nfts-on-bch#1",
    "ipfs": "bafybeif3emr4immcvvco2y5tlav7jtfdvoescj5vaelqwbz6scdfzj4oka",
    "filecoin": "https://bafybeif3emr4immcvvco2y5tlav7jtfdvoescj5vaelqwbz6scdfzj4oka.ipfs.dweb.link/video1352867031.mp4"
  }
}
```

### nsfw (required)
NSFW is an acronym for 'Not Safe For Work'. If the token represents content that would generally not be considered 'safe' or socially acceptable in a corporate work environment, this property should be set to `true`. This allows the displaying software to categorize the token appropriately and only display it when the user has opted-in to seeing NSFW content.

Most display software will also incorporate collective moderation, allowing users to flag NSFW content that was inappropriately marked by the token creator. While the `nsfw` property can only be indicated voluntarily by the token creator, display software may take punitive measures (such as blacklisting the token from their platform) when users flag it. Thus, it is in the best interest of the token creator to set this field honestly.

### userData (required)
The `userData` property is intended to 'future proof' this specification by allowing a field for users to place arbitrary data. If no data is provided, this property should resolve into an empty object (`{}`).

### jsonLd (required)
The `jsonLd` property is another 'future proof' property. It is intended to capture [JSON-LD](https://json-ld.org/) structured data following [Schema.org](https://schema.org/) schema. The recommended schema model is the [CreativeWork](https://schema.org/CreativeWork) model. If no JSON-LD data is desired, this property should resolve into an empty object (`{}`).

### about (optional)
The `about` property should contain a string of human-readable information, to help people quickly grasp the context of the token. This is optional data. If not used, this property should resolve to an empty string (`""`).

### category (optional)
The `category` property should contain a string of data. This is optional data, and when provided, can be used by the display software to easily categorize the token. If not used, this property should resolves to an empty string (`""`).

### tags (optional)
The `tags` property should contain an array of strings. This is optional data, and when provided, can be used by the display software to associate it with other tokens. If not used, this property should resolves to an empty string (`""`).

### medaiType (optional)
This is a string describing the type of media the token represents. Here are common formats:
- `image`
- `audio`
- `video`
- `text`
- `3d`

### currentOwner (optional)
The `currentOwner` property is an object that contains information about the current owner of the token. This field is provided to allow the token creator to update the ownership information as the token changes hands. This provides leverage for token creators to enforce their license terms.

If this field is not used, it should be expressed as an empty object (`{}`). Here is an example of how current ownership could be expressed:

```json
{
  ...
  "currentOwner": {
    "bchAddr": "bitcoincash:qzzrdesj9el7tusyt0aujnqeja3e8zs05ga2km6wlc",
    "name": "Chris Troutner",
    "contactEmail": "chris.troutner@gmail.com"
  }
}
```
