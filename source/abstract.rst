Abstract
========

The `proposed CF aggregation rules
<https://cf-pcmdi.llnl.gov/trac/ticket/78>`_ describe how multiple CF
fields may be combined into one, larger field. These rules are fully
general and are based purely on CF metadata. To make full use of such
an algorithm, it is necessary that such an aggregated field may be
stored within software memory (for field creation and manipulation)
and on disk (for long term storage).

This document proposes the abstract **CFA framework** for the storage
of CF-aggregated CF fields, and the **CFA-netCDF conventions** for
their efficient storage in netCDF files.

As with other similar file storage conventions, on-disk storage is
efficient because, wherever possible, the component fields from which
an aggregated field is composed are stored as references to the files
containing the original datasets.

Both the abstract framework and the ability to read and write NCA
files are implemented in the `cf-python
<http://cfpython.bitbucket.org/>`_ software library.
