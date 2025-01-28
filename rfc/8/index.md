# RFC-8: Mesh Support in NGFF

Add support for mesh representations of segmented objects in OME-NGFF by adapting the Neuroglancer mesh format.

## Status

This proposal is very early. Status: D1

```{list-table} Record
:widths: 8, 20, 20, 20, 15, 10
:header-rows: 1
:stub-columns: 1

*   - Role
    - Name
    - GitHub Handle
    - Institution
    - Date
    - Status
*   - Author
    - Kyle Harrington
    - @kephale
    - CZI
    - 2024-01-13
    - 
```

## Overview

This RFC proposes adding support for storing multi-resolution triangle mesh representations of segmented objects in OME-NGFF by adapting the Neuroglancer mesh format specification.

## Background

Surface mesh representations are essential for visualization and analysis of 3D structures in bio-imaging. The current NGFF specification lacks native support for mesh representations. The Neuroglancer project provides a well-tested multi-resolution mesh format that would fill this gap.

## Proposal

Add mesh support to NGFF by integrating with the upcoming collections proposal for specifying image-segmentation relationships. The mesh data will be stored as an external artifact within the Zarr hierarchy.

### Integration with Collections

Meshes are integrated into the OME-NGFF specification as members of collections. A collection can reference both images and their associated mesh representations:


```json
{
    "ome": {
        "version": "0.x",
        "collection": {
            "name": "em_reconstruction",
            "members": [
                {
                    "type": "image",
                    "path": "./raw",
                    "attributes": {
                        // image-specific attributes
                    }
                },
                {
                    "type": "mesh",
                    "path": "./meshes",
                    "attributes": {
                        "type": "neuroglancer_multilod_draco",
                        "vertexQuantizationBits": 10,
                        "lodScaleMultiplier": 2.0,
                        "coordinateTransformations": [
                            {
                                "type": "scale",
                                "scale": [1.0, 1.0, 1.0]
                            }
                        ]
                    }
                }
            ]
        }
    }
}
```


### Storage Layout

When storing mesh data within a collection:

```
[data].zarr/
  ├── zarr.json          # Collection metadata (shown above)
  ├── raw/               # Image data
  │   ├── zarr.json      # Image metadata
  │   └── ...
  └── meshes/            # Mesh data
      ├── zarr.json      # External node type metadata
      ├── info           # Mesh format metadata
      ├── manifest/      # Binary manifest files (unsharded)
      └── fragments/     # Binary mesh fragment data (unsharded)
```

### Mesh Directory Metadata

The mesh directory contains a `zarr.json` that identifies it as an external node:

```
{
  "zarr_format": 3,
  "node_type": "external",
  "attributes": {
    "ome": {
      "version": "0.5"
    }
  }
}
```

The mesh-specific metadata lives in the collection's member attributes (as shown in the Integration with Collections section above) rather than in the mesh directory's zarr.json. This allows the same mesh data to be referenced by multiple collections with different transformations or rendering settings.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [IETF RFC 2119].

## Stakeholders

The main stakeholders are bio-image visualization tool developers and scientists working with 3D segmentation data.

### Socialization
* [Github issue](https://github.com/ome/ngff/issues/33)
* [Initial meeting with stakeholders](https://github.com/ome/ngff/issues/33#issuecomment-2555637903)

## Implementation

Implementation requires:
1. Metadata handling following NGFF conventions
2. Binary manifest and mesh fragment handling
3. Integration with existing coordinate transform system

## Drawbacks, risks, alternatives, and unknowns

This proposal depends on Collections RFC-7.

This proposal adds complexity but provides necessary functionality for 3D visualization. The use of Draco compression creates an external dependency but provides significant storage benefits.

## Compatibility

This proposal adds new capabilities without affecting existing functionality. Reading mesh data is optional - implementations that don't support meshes can ignore the mesh metadata and data.

## Changelog

| Date | Description | Link |
|------------|-------------------------------|------------------------------------------|
| 2024-01-13 | Initial RFC draft | TBD |