Introduction
============

The proposed `CF aggregation rules
<https://cf-pcmdi.llnl.gov/trac/ticket/78>`_ allow for the aggregation
of `CF <http://cf-pcmdi.llnl.gov>`_ fields across multiple
dimensions. These rules are based solely on the fields' metadata,
therefore allowing aggregation to be reliably automated. This raises
the possibility of large amounts of aggregations being created and
therefore storing these collections is clearly desirable.

This document describes how the data arrays of such aggregations could
be stored in the memory of an application and proposes a convention --
the NCA (`netCDF <http://www.unidata.ucar.edu/software/netcdf/>`_
aggregate) convention -- for their efficient file storage.

These ideas have been implemented in `cf-python
<http://cfpython.bitbucket.org>`_. In particular, its `Large Amounts
of Massive Arrays (LAMA)
<http://cfpython.bitbucket.org/docs/0.9.6/build/lama.html>`_
functionality stores arrays in this fashion.

The key element of the NCA files is that they are CF-compliant netCDF
files, albeit ones which require extra processing to realise their
aggregated data arrays.

This style of aggregation is distinct from `NetCDF Markup Language
(NcML) aggregation
<http://www.unidata.ucar.edu/software/netcdf/ncml/v2.2/Aggregation.html>`_,
which has long-standing use in the community, in that it allows
simultaneous aggregation across arbitrarily positioned dimensions, can
aggregate arbitrary parts of arrays and can aggregate arrays stored in
arbitrary formats (in-memory, netCDF, PP, *etc.*).
