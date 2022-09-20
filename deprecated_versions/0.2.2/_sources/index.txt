.. cf-python documentation master file, created by
   sphinx-quickstart on Wed Aug  3 16:28:25 2011.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

On the storage of arbitrarily aggregated data arrays (|version|)
################################################################

David Hassell, `NCAS <http://www.ncas.ac.uk/>`_

Abstract
========

This document proposes a `netCDF
<http://www.unidata.ucar.edu/software/netcdf/>`_ file format
convention for the efficient storage of a `CF
<http://cf-pcmdi.llnl.gov>`_ field created by the `CF aggregation
rules <https://cf-pcmdi.llnl.gov/trac/ticket/78>`_, whose data array
is distributed over one or more files, one of which may be the storage
file itself. The storage is efficient because, wherever possible, data
are replaced with references to the files containing them. To fully
accommodate such fields, this convention allows for:

* Simultaneous aggregation across more than one dimension.

* Storage of changes to arbitrary parts of the aggregated array.

* Aggregation accounting for different but equivalent:

  - order of dimensions
  - number of size 1 dimensions
  - senses in which dimensions run
  - units of the data values
  - missing data values

The convention is derived from a framework for storing such aggregated
arrays in memory, which is also described.

Although not required to store such aggregated fields, the framework
and convention also allow for:

* Storage of (not necessarily contiguous) subspaces of aggregated
  arrays.

* Aggregation of arrays stored in a mixture of formats (in-memory,
  netCDF, PP, *etc.*).

* Manipulation and subsequent storage of larger-than-memory arrays.

Contents
========

.. toctree::
   :maxdepth: 1
   :numbered:

   introduction
   framework
   nca
   nca_attributes

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

