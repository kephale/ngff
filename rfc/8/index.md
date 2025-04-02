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

This RFC proposes adding support for storing multi-resolution triangle mesh representations of segmented objects in OME-NGFF by adapting the [Neuroglancer mesh format specification](https://github.com/google/neuroglancer/blob/master/src/datasource/precomputed/meshes.md).

## Background

Surface mesh representations are essential for visualization and analysis of 3D structures in bio-imaging. The current NGFF specification lacks native support for mesh representations. The Neuroglancer project provides a well-tested multi-resolution mesh format that would fill this gap.

## Proposal

### Integration with NGFF

Following the pattern established in the OME-NGFF specification for image data and label data, mesh data will be stored as a new group within the Zarr hierarchy.

### Storage Layout

When storing mesh data alongside images and labels in an NGFF container:

```
[data].zarr/
  ├── zarr.json          # Root metadata with OME version
  ├── 0/                 # Image data (highest resolution)
  │   └── 0/             # First channel/timepoint
  ├── 1/                 # Image data (next resolution level)
  │   └── 0/             # First channel/timepoint
  ├── labels/            # Optional segmentation data
  │   └── segmentation/  # Segmentation data
  │       ├── 0/         # Highest resolution segmentation
  │       ├── 1/         # Next resolution level
  │       └── ...
  └── meshes/            # Mesh data (name is flexible, identified by zarr.json)
      ├── zarr.json      # Mesh metadata identifying type and version
      ├── info           # Neuroglancer mesh format metadata
      ├── 1              # Mesh data for object ID 1
      ├── 1.index        # Mesh manifest for object ID 1
      ├── 2              # Mesh data for object ID 2
      ├── 2.index        # Mesh manifest for object ID 2
      └── ...
```

### Mesh Group Metadata

The mesh group contains a `zarr.json` that identifies it as a mesh collection:

```json
{
  "zarr_format": 3,
  "node_type": "external",
  "attributes": {
    "ome": {
      "version": "0.5",
      "mesh": {
        "version": "0.1",
        "type": "neuroglancer_multilod_draco",
        "source": {
          "image": "../",
          "labels": "../labels/segmentation"
        }
      }
    }
  }
}
```

The mesh metadata includes:
- `version`: String specifying the version of the mesh specification
- `type`: String identifying the mesh format (currently only "neuroglancer_multilod_draco")
- `source`: Object specifying the relative paths to the associated image and label data

### Neuroglancer Mesh Format

The format follows the Neuroglancer Precomputed mesh specification with specific additions for OME-NGFF compatibility:

#### The `info` File Format

```json
{
  "@type": "neuroglancer_multilod_draco",
  "vertex_quantization_bits": 10,
  "transform": [1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0],
  "lod_scale_multiplier": 2.0,
  "data_type": "uint64",
  "num_channels": 1,
  "type": "segmentation"
}
```

Parameters:
- `vertex_quantization_bits`: Precision for vertex coordinates (10 or 16)
- `transform`: 4x3 homogeneous transformation matrix that SHOULD align with the coordinate transformations specified in the associated multiscale image
- `lod_scale_multiplier`: Scaling factor between LOD levels

#### Binary Format

The binary manifest (.index) and mesh data files follow the Neuroglancer Precomputed format specifications unchanged.

### Relationship to Label Data

Each mesh object ID corresponds to a label ID in the segmentation volume. The transformation matrix specified in the `info` file should align with the coordinate transformations specified in the original image and label data.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [IETF RFC 2119].

- The mesh group MUST contain a `zarr.json` file with the mesh metadata as specified
- The mesh metadata MUST contain the `version`, `type` and `source` fields
- The mesh group MUST contain an `info` file following the Neuroglancer format
- The source field SHOULD reference the associated image and/or label data
- The transformation matrices SHOULD align with those in the associated multiscale image
- Implementations MAY support all levels of detail or only the highest resolution meshes
- Readers that do not support meshes MUST ignore the mesh group

## Stakeholders

The main stakeholders are bio-image visualization tool developers and scientists working with 3D segmentation data.

### Socialization
* [Github issue](https://github.com/ome/ngff/issues/33)
* [Initial meeting with stakeholders](https://github.com/ome/ngff/issues/33#issuecomment-2555637903)

## Implementation

Implementation requires:
1. Metadata handling following NGFF conventions
2. Binary manifest and mesh fragment handling
3. Draco compression for mesh data
4. Proper placement within the NGFF Zarr hierarchy

A reference implementation is provided that:
1. Generates meshes from segmented volumes
2. Creates multiple levels of detail
3. Divides meshes into spatial fragments
4. Compresses fragments using Draco
5. Writes binary manifests, mesh data, and metadata

## Drawbacks, risks, alternatives, and unknowns

Drawbacks:
- Additional dependency on Draco compression
- Increased complexity in the NGFF specification

The Neuroglancer format was chosen because:
1. It supports multi-resolution visualization (essential for large datasets)
2. It has an established ecosystem of tools
3. It provides efficient spatial organization and compression of mesh data

## Compatibility

This proposal adds new capabilities without affecting existing functionality. Reading mesh data is optional - implementations that don't support meshes can ignore the mesh data and continue to work with image and label data.

## Changelog

| Date | Description | Link |
|------------|-------------------------------|------------------------------------------|
| 2024-01-13 | Initial RFC draft | TBD |
| 2024-03-20 | Updated to standalone format (not dependent on Collections) | TBD |