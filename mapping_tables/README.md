# MiHIN Mapping Tables

### Background

The HL7 data supplied in the MiHIN "ADT" use-case contains data in the native
coding scheme of the sending facility.  This can vary from one facility to
another and the same code may mean two different things between different facilities.

MiHIN has worked toward gathering the coding information from each facility
so that these native codes can be translated into standardized and/or custom
labels.  Historically this data has resided in a series of Excel spreadsheets
hosted on a [box.com](https://mihin.box.com/s/2r247jiarfyg524y7oe285eph4tddep3)
file share.  This delivery mechanism makes it challenging to keep the mapping
data current and accurate in an operation database for several reasons:

1) Data are stored in Excel spreadsheets, requiring an translation layer
2) As most of the files are 8Mb in size, the entire archive is currently 14Gb, which could require > 1hr to download
3) The spreadsheets are arranged in a way that is convenient for entry, but awkward for automated processing
4) Many of the files have duplicate and/or overlapping information
5) Data is often recorded with multiple values in a single cell, requiring a splitting to get unique values
6) Some sources don't specify a value at all if it already uses the MiHIN-suggested values

The purpose of this github repository is to provide the same information that is
contained in the spreadsheets, but in a form easily consumable by operational
systems.

### Process

The box.com is polled twice daily (at 6:00am and 6:00pm), and any new files are
automatically processed, and any affected files are updated and posted to this
github repository.  As a result, if a system is configured to poll the endpoint
URLs (see [target directory](target)), that system will **never have mapping table
data that is more than a few hours old**.

### Contents

* **`err`**
  * YAML files containing data errors and warnings found while processing the correspondng file(s)
* **`src`**
  * Tab-separated text files representing the contents of the spreadsheet.
The source code history in github can be used to conveniently determine what has
changed from one version to another
* **`target`**
  * This is the delivery endpoint for the mapping data.  To integrate the results into
an operational system see the [README.md](target/README.md) in the [target](target/) directory.
* **`filelist.txt`**
  * A sorted list of all the the original filenames represented in the target output
* **`README.md`**
  * This file
