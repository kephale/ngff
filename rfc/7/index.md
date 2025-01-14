# RFC-7: Mesh Support in NGFF

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

Add a new `mesh` property to the image-label metadata schema to support multi-resolution mesh representations.

The mesh metadata MUST be stored in a zarr.json file with the following structure:

```json
{
  "zarr_format": 3,
  "node_type": "mesh",
  "attributes": {
    "ome": {
      "version": "0.5",
      "mesh": {
        "version": "0.1",
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
  }
}
```

For each segmented object:
- A binary manifest file specifying:
  - Octree decomposition parameters
  - LOD scale information
  - Fragment locations and sizes
- Draco-encoded mesh fragments for each octree node

### Storage Layout

```
[data].zarr/
  ├── labels/
  │   └── [segmentation]/
  │       ├── zarr.json  # Contains mesh metadata
  │       ├── fragments/ # Binary mesh data
  │       └── manifest/  # Binary manifest files
```

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

This proposal adds complexity but provides necessary functionality for 3D visualization. The use of Draco compression creates an external dependency but provides significant storage benefits.

## Compatibility

This proposal adds new capabilities without affecting existing functionality. Reading mesh data is optional - implementations that don't support meshes can ignore the mesh metadata and data.

## Changelog

| Date | Description | Link |
|------------|-------------------------------|------------------------------------------|
| 2024-01-13 | Initial RFC draft | TBD |