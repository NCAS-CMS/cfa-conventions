Introduction
============

The `proposed CF aggregation rules
<https://cf-pcmdi.llnl.gov/trac/ticket/78>`_ offer a fully general
methodology for combining two or more CF fields which logically form
one, larger dataset.

To fully accommodate such fields, any in-memory or on-disk storage
must allow for:

* Simultaneous aggregation across more than one dimension.

* Aggregation accounting for different but equivalent:

  - orders of dimensions
  - numbers of size 1 dimensions
  - senses (directions) in which dimensions run
  - units of the data
  - missing data values

A framework for in-memory storage must also allow for the aggregation
of previously aggregated fields, since that is required for
multidimensional aggregations. The framework presented here satisfies
these requirements and is abstract in that it is "language neutral",
i.e. it does not rely on structures particular to any one computer
programming language, and should therefore be implementable in any
sufficiently rich programming environment.

Having created such an abstract framework, it will also serve as the
model for storing aggregated fields on disk. To be useful, a file
encoding must follow a documented and standardized convention, since
such files could form part of a maintained archive. The convention
proposed here, the CFA-netCDF convention, proposes encodings for
netCDF file storage. The key element of a CFA-netCDF file is that it
is a CF-like netCDF file, which only differs from a normal CF-netCDF
file in that it requires extra processing to realise any aggregated
data arrays. In all other aspects, the metadata in a CFA-netCDF file
is identical to CF metadata.

In addition to storing aggregated fields, the abstract framework and
file storage convention proposed here also allows for:

* Storage of (not necessarily contiguous) subspaces of aggregated
  fields *without duplicating* any part of the original datasets which
  span the subspace.

* Storage of changes to (not necessarily contiguous) parts of
  aggregated fields.

* Aggregation and storage of datasets originating in a mixture of
  formats (in-memory, netCDF, `PP
  <http://badc.nerc.ac.uk/help/formats/pp-format/>`_, *etc.*),
  provided that they may be cast as CF fields.

* In-memory manipulation and on-disk storage of larger-than-memory
  fields.

It is important that the ideas presented are demonstrably viable from
a software engineering point of view, so the `cf-python (v0.9.7)
<http://cfpython.bitbucket.org/>`_ software library has been modified
to implement the full CF aggregation algorithm, the ability to read
and write CFA-netCDF files and the additional benefits listed above.
