# ESM Collection Specification

- [ESM Collection Specification](#esm-collection-specification)
  - [Overview](#overview)
    - [Collection Specification](#collection-specification)
    - [Catalog](#catalog)
    - [Assets (Data Files)](#assets-data-files)
  - [Catalog fields](#catalog-fields)
    - [Attribute Object](#attribute-object)
    - [Assets Object](#assets-object)
    - [Aggregation Control Object](#aggregation-control-object)
    - [Aggregation Object](#aggregation-object)
  - [Examples](#examples)
  - [Python Validator](#python-validator)
    - [Installation](#installation)
    - [Usage](#usage)

## Overview

This document explains the structure and content of an ESM Collection.
A collection provides metadata about the catalog, telling us what we expect to find inside and how to open it.
The collection is described is a single json file, inspired by the STAC spec.

The ESM Collection specification consists of three parts:

### Collection Specification

The _collection_ specification provides metadata about the catalog, telling us what we expect to find inside and how to open it.
The descriptor is a single json file, inspired by the [STAC spec](https://github.com/radiantearth/stac-spec).

```json
{
  "esmcat_version": "0.1.0",
  "id": "sample",
  "description": "This is a very basic sample ESM collection.",
  "catalog_file": "sample_catalog.csv",
  "attributes": [
    {
      "column_name": "activity_id",
      "vocabulary": "https://raw.githubusercontent.com/WCRP-CMIP/CMIP6_CVs/master/CMIP6_activity_id.json"
    },
    {
      "column_name": "source_id",
      "vocabulary": "https://raw.githubusercontent.com/WCRP-CMIP/CMIP6_CVs/master/CMIP6_source_id.json"
    }
  ],
  "assets": {
    "column_name": "path",
    "format": "zarr"
  }
}
```

### Catalog

The collection points to a single catalog.
A catalog is a CSV file.
The meaning of the columns in the csv file is defined by the parent collection.

```csv
activity_id,source_id,path
CMIP,ACCESS-CM2,gs://pangeo-data/store1.zarr
CMIP,GISS-E2-1-G,gs://pangeo-data/store1.zarr
```

### Assets (Data Files)

The data assets can be either netCDF or Zarr.
They should be either [URIs](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier) or full filesystem paths.

## Catalog fields

| Element             | Type                                                      | Description                                                                                                                                                               |
| ------------------- | --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| esmcat_version      | string                                                    | **REQUIRED.** The ESM Catalog version the collection implements.                                                                                                          |
| id                  | string                                                    | **REQUIRED.** Identifier for the collection.                                                                                                                              |
| title               | string                                                    | A short descriptive one-line title for the collection.                                                                                                                    |
| description         | string                                                    | **REQUIRED.** Detailed multi-line description to fully explain the collection. [CommonMark 0.28](http://commonmark.org/) syntax MAY be used for rich text representation. |
| catalog_file        | string                                                    | **REQUIRED.** Path to a the CSV file with the catalog contents.                                                                                                           |
| catalog_dict        | array                                                     | If specified, it is mutually exclusive with `catalog_file`. An array of dictionaries that represents the data that would otherwise be in the csv.                         |
| attributes          | [[Attribute Object](#attribute-object)]                   | **REQUIRED.** A list of attribute columns in the data set.                                                                                                                |
| assets              | [Assets Object](#assets-object)                           | **REQUIRED.** Description of how the assets (data files) are referenced in the CSV catalog file.                                                                          |
| aggregation_control | [Aggregation Control Object](#aggregation-control-object) | **OPTIONAL.** Description of how to support aggregation of multiple assets into a single xarray data set.                                                                 |

### Attribute Object

An attribute object describes a column in the catalog CSV file.
The column names can optionally be associated with a controlled vocabulary, such as the [CMIP6 CVs](https://github.com/WCRP-CMIP/CMIP6_CVs), which explain how to interpret the attribute values.

| Element     | Type   | Description                                                                            |
| ----------- | ------ | -------------------------------------------------------------------------------------- |
| column_name | string | **REQUIRED.** The name of the attribute column. Must be in the header of the CSV file. |
| vocabulary  | string | Link to the controlled vocabulary for the attribute in the format of a URL.            |

### Assets Object

An assets object describes the columns in the CSV file relevant for opening the actual data files.

| Element            | Type   | Description                                                                                                                        |
| ------------------ | ------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| column_name        | string | **REQUIRED.** The name of the column containing the path to the asset. Must be in the header of the CSV file.                      |
| format             | string | The data format. Valid values are `netcdf` and `zarr`. If specified, it means that all data in the catalog is the same type.       |
| format_column_name | string | The column name which contains the data format, allowing for variable data types in one catalog. Mutually exclusive with `format`. |

### Aggregation Control Object

An aggregation control object defines neccessary information to use when aggregating multiple assets into a single xarray data set.

| Element              | Type                                        | Description                                                                             |
| -------------------- | ------------------------------------------- | --------------------------------------------------------------------------------------- |
| variable_column_name | string                                      | **REQUIRED.** Name of the attribute column in csv file that contains the variable name. |
| groupby_attrs        | array                                       | Column names (attributes) that define data sets that can be aggegrated.                 |
| aggregations         | [[Aggregation Object](#aggregation-object)] | **OPTIONAL.** List of aggregations to apply to query results                            |

### Aggregation Object

An aggregation object describes types of operations done during the aggregation of multiple assets into a single xarray data set.

| Element        | Type   | Description                                                                                                                                                                                                                                                                                                                                                                                          |
| -------------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| type           | string | **REQUIRED.** Type of aggregation operation to apply. Valid values include: `join_new`, `join_existing`, `union`                                                                                                                                                                                                                                                                                     |
| attribute_name | string | Name of attribute (column) across which to aggregate.                                                                                                                                                                                                                                                                                                                                                |
| options        | object | **OPTIONAL.** Aggregration settings that are passed as keywords arguments to [`xarray.concat()`](https://xarray.pydata.org/en/stable/generated/xarray.concat.html) or [`xarray.merge()`](https://xarray.pydata.org/en/stable/generated/xarray.merge.html#xarray.merge). For `join_existing`, it must contain the name of the existing dimension to use (for e.g.: something like `{'dim': 'time'}`). |

## Examples

- **[examples/](examples/)** directory contains examples of esm collections.

- **[intake-esm-datastore](https://github.com/NCAR/intake-esm-datastore/tree/master/catalogs)** repository contains another set of real world esm collections.

## Python Validator

### Installation

The [Python validator for the esm-collection-spec](https://github.com/NCAR/esmcol-validator) can be installed in any of the following ways:

Using Pip via PyPI:

```bash
python -m pip install esmcol-validator
```

Using Conda:

```bash
conda install -c conda-forge esmcol-validator
```

Or from the source repository:

```bash
python -m pip install git+https://github.com/NCAR/esmcol-validator.git
```

### Usage

```bash
$ esmcol-validator --help
Usage: esmcol-validator [OPTIONS] ESMCOL_FILE

  A utility that allows users to validate esm-collection json files against
  the esm-collection-spec.

Options:
  --esmcol-spec-dirs TEXT
  --version TEXT           [default: master]
  --verbose                [default: False]
  --timer                  [default: False]
  --log-level TEXT         [default: CRITICAL]
  --help                   Show this message and exit.
```

Example:

```bash
$ esmcol-validator sample-pangeo-cmip6-collection.json
{'collections': {'valid': 1, 'invalid': 0}, 'catalog_files': {'valid': 1, 'invalid': 0}, 'unknown': 0}
{
    "collections": {
        "valid": 1,
        "invalid": 0
    },
    "catalog_files": {
        "valid": 1,
        "invalid": 0
    },
    "unknown": 0
}
```
