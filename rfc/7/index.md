# RFC-7 Mesh Support in NGFF

```{toctree}
:hidden:
:maxdepth: 1
reviews/index
comments/index
responses/index
versions/index
```

Add support for multiscale mesh representations in OME-NGFF by adopting Neuroglancer's multiscale mesh format.

## Status

This proposal is in early stages. Status: `D1`

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

This RFC adds support for multi-resolution triangle meshes in OME-NGFF by adopting Neuroglancer's [multi-resolution mesh format](https://github.com/google/neuroglancer/blob/master/src/datasource/precomputed/meshes.md), enabling efficient storage and retrieval of surface meshes representing segmented objects at multiple levels of detail.

## Background

Surface mesh representations are essential for visualization and analysis of 3D structures in bio-imaging. The Neuroglancer project has developed a robust multi-resolution mesh format that provides efficient storage and progressive loading through octree-based spatial decomposition and Draco compression.

## Proposal

The mesh data for a segmentation must be organized as:

1. An `.ngattrs` JSON file with the following required fields:
   - `"@type"`: Must be `"neuroglancer_multilod_draco"`
   - `"vertex_quantization_bits"`: Must be `10` or `16`
   - `"transform"`: 4x3 homogeneous coordinate transform matrix
   - `"lod_scale_multiplier"`: Scale factor for LOD scales

2. For each segmented object:
   - A binary manifest file specifying octree decomposition and LOD information
   - Draco-encoded mesh fragment data for each octree node

### Manifest Format

Binary manifest files must specify:
- Octree parameters (chunk shape, grid origin)
- LOD information (number of levels, scales, vertex offsets)
- Fragment information per LOD (counts, positions, offsets)

### Mesh Fragment Format

Each mesh fragment must be:
- Draco-encoded triangular mesh
- Grid-aligned for LOD > 0
- Position-quantized based on vertex_quantization_bits

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [IETF RFC 2119].

## Stakeholders

Primary stakeholders include developers and scientists working with 3D segmentation visualization and analysis.

### Socialization

* [Github issue](https://github.com/ome/ngff/issues/33)
* [Initial meeting with stakeholders](https://github.com/ome/ngff/issues/33#issuecomment-2555637903)
* [not done yet] Discussion in OME-NGFF community calls

## Implementation

Implementation requires:
- JSON and binary manifest parsing
- Draco mesh encoding/decoding
- Octree-based spatial decomposition
- Coordinate transform handling

## Drawbacks, risks, alternatives, and unknowns

Drawbacks:
- Dependency on Draco compression
- Implementation complexity
- Learning curve for developers

Alternatives considered:
- Creating a new mesh format
- Using simpler single-resolution formats (glTF, PLY)

## Compatibility

This proposal adds new capabilities without affecting existing functionality. Implementation is optional for NGFF readers/writers.

## Changelog

| Date       | Description                   | Link                                    |
|------------|-------------------------------|------------------------------------------|
| 2024-01-13 | Initial RFC draft            | TBD                                     |