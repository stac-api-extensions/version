# Items and Collections API Version Extension

- **OpenAPI specification:** [openapi.yaml](openapi.yaml)
- **Conformance URIs:** <https://api.stacspec.org/v0.1.0/ogcapi-features/extensions/version>
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-api-spec/blob/main/README.md#maturity-classification):** Proposal
- **Dependencies**: [STAC API - Features](https://github.com/radiantearth/stac-api-spec/blob/main/ogcapi-features/README.md)

The STAC API - Features conformance class doesn't support semantics to creating and accessing different
versions of an Item or Collection.
The Version Extension defines the API resources and semantics for creating and accessing versioned records.

## Methods

| Path                                                                     | Content-Type Header | Description                                                                  |
| ------------------------------------------------------------------------ | ------------------- | ---------------------------------------------------------------------------- |
| `GET /collections/{collectionID}/versions`                               | `application/json`  | Returns a catalog response with links to all versions of a given collection. |
| `GET /collections/{collectionID}/versions/{versionId}`                   | `application/json`  | Returns a collection record.                                                 |
| `GET /collections/{collectionID}/items/{featureId}/versions`             | `application/json`  | Returns a catalog response with links to all versions of a given item.       |
| `GET /collections/{collectionID}/items/{featureId}/versions/{versionId}` | `application/json`  | Returns an item record.                                                      |

## How It Works

When this extension is implemented, the API supports `versions` resource that list all versions
of an item or collection and provides permanent links to each version.

The latest version of an item is accessible at `/collections/{collectionID}/items/{itemsId}`.
The record has a link with `"rel": "permalink"` that links to the permanent location of the record
which is `/collections/{collectionID}/items/{itemsId}/versions/{versionId}`.

If the record has a previous version it also provides a link to the permanent location of that version
using `"rel": "predecessor-version"`.

Each updated records provides the link to the previous version.

The path `/collections/{collectionID}/items/{itemsId}/versions/` provides a catalog response with
list of links to all versions available for an item.

## Version ID

Version ID is a unique identifier for a version of an Item or Collection.
This extension remains agnostic about what the identifier should be.
There are many options for a versioning schema including:
- md5 hash of the record
- datetime epoch
- incrementing number
- semantic versioning

## Link "rel" Types

The extension uses [RFC5829](https://tools.ietf.org/html/rfc5829) rel types to link to different versions:

| Type                | Description                                                                                                                                                                           |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| latest-version      | Points to the latest version of the record.                                                                                                                                           |
| version-history     | Points to the list of versions.                                                                                                                                                       |
| predecessor-version | Points to the previous version of the document.                                                                                                                                       |
| successor-version   | Points to the successor version in the version history.                                                                                                                               |
| permalink           | Points to the permanent location of the record. This location points to a specific version, remains accessible and will not change even when there are future versions of the record. |

## Example

For an item record with the id `this_is_my_id` and version of `02`, this is how the versioning works

Request to `GET /collections/my_collection/items/this_is_my_id`:
```json
{
    "id": "this_is_my_id",
    "type": "Feature",
    "bbox": [],
    "geometry": {},
    "properties": {},
    "links": [
        {
            "rel": "self",
            "href": "/collections/my_collection/items/this_is_my_id"
        },
        {
            "rel" : "permalink",
            "href": "/collections/my_collection/items/this_is_my_id/versions/02"
        },
        {
            "rel": "predecessor-version",
            "href": "/collections/my_collection/items/this_is_my_id/versions/01"
        },
        {
            "rel": "latest-version",
            "href": "/collections/my_collection/items/this_is_my_id"
        },
        {
            "rel": "version-history",
            "href": "/collections/my_collection/items/this_is_my_id/versions"
        }
    ]
}
```

Request to `GET /collections/my_collection/items/this_is_my_id/versions/02`:
```json
{
    "id": "this_is_my_id",
    "type": "Feature",
    "bbox": [],
    "geometry": {},
    "properties": {},
    "links": [
        {
            "rel": "self",
            "href": "/collections/my_collection/items/this_is_my_id/versions/02"
        },
        {
            "rel" : "permalink",
            "href": "/collections/my_collection/items/this_is_my_id/versions/02"
        },
        {
            "rel": "predecessor-version",
            "href": "/collections/my_collection/items/this_is_my_id/versions/01"
        },
        {
            "rel": "latest-version",
            "href": "/collections/my_collection/items/this_is_my_id"
        },
        {
            "rel": "version-history",
            "href": "/collections/my_collection/items/this_is_my_id/versions"
        }
    ]
}
```

Request to `GET /collections/my_collection/items/this_is_my_id/versions/01`:
```json
{
    "id": "this_is_my_id",
    "type": "Feature",
    "bbox": [],
    "geometry": {},
    "properties": {},
    "links": [
        {
            "rel": "self",
            "href": "/collections/my_collection/items/this_is_my_id/versions/01"
        },
        {
            "rel" : "permalink",
            "href": "/collections/my_collection/items/this_is_my_id/versions/01"
        },
        {
            "rel": "predecessor-version",
            "href": "/collections/my_collection/items/this_is_my_id/versions/01"
        },
        {
            "rel": "latest-version",
            "href": "/collections/my_collection/items/this_is_my_id"
        },
        {
            "rel": "version-history",
            "href": "/collections/my_collection/items/this_is_my_id/versions"
        }
    ]
}
```

## FAQ

- **How do I find the latest version of an item?**
  - By going to `/collections/{collectionID}/items/{itemsId}` *or*
  - by looking at `"rel": "latest-version"`.

- **How do I find the permalink of an item I'm looking at?**
  - By looking at the `href` value of a link with `"rel": "permalink"`

- **How do I find the previous versions of a record?**
  - By going to `/collections/{collectionID}/items/{itemsId}/versions`

- **How do I find the order of versions?**
  - By following the `"rel": "predecessor-version"` links.
