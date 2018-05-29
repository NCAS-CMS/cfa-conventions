.. _CFA-netCDF-conventions:

The CFA-netCDF conventions
==========================

The CFA-netCDF conventions describe the efficient netCDF file storage
of CF fields with master data arrays which have been aggregated within
the :ref:`CFA framework <framework>`.

Storing the master data array parameters in a string
----------------------------------------------------

Since the :ref:`parameters <CFA-framework-parameters>` which
completely describe a master data array may each be cast as one of a
set of particular basic types, these parameters may be easily encoded
in a `JSON (JavaScript Object Notation) string <http://www.json.org>`_
for simple inclusion in a netCDF file.

JSON is a lightweight data-interchange format which is easy for humans
to read and write and easy for machines to parse and generate. There
are JSON encoders and decoders for every reasonable language. See the
`JSON Wikipedia article <http://en.wikipedia.org/wiki/JSON>`_ for some
good examples of JSON strings.

The basic types recognised by JSON are (with JSON encoded examples):

* string (double-quoted Unicode, with backslash escaping: ``"foo"``,
  ``""``)

* number (``3.14159``, ``0``)

* boolean (true or false: ``true``, ``false``)

* null (empty: ``null``)

* Array (an ordered, comma-separated sequence of values enclosed in
  square brackets; the values do not need to be of the same type:
  ``[1, "a", [2, null, {"x": false}]]``)

* Object (an unordered, comma-separated collection of key:value pairs
  enclosed in curly braces, with the ':' character separating the key
  and the value; the keys must be strings and should be distinct from
  each other: ``{"x": [4, [true, null]], "y": 2.71828}``)

It clear that, with the exception of the partition's :ref:`subarray
<frame-subarray>` parameter, each of the values of the parameters
which describe a master data array is one or a combination of these
basic types. For example, the :ref:`pmshape <frame-pmshape>` parameter
is a list of numbers.

The same is actually true for the partition's :ref:`subarray
<frame-subarray>` parameter, although it is not so immediately
obvious. The issue is that it may be a reference to section of a file
or to a multidimensional array in memory. However, in the latter case
the sub-array must be written to a file in order to be stored, so this
reduces to the former case. A file reference is easily encapsulated by
a collection of the basic types. For example, if the reference is to a
netCDF file variable then all that is required are the file name (a
string), the name of the netCDF variable containing the sub-array (a
string) and its shape (a list of numbers) [#f1]_.

Thus, for storage purposes, all of the master data array parameters
may be encoded in a single JSON string.

See the complete description of :ref:`CFA-netCDF attributes <CFA-ref>`
for details on how each master data array parameter is encoded.

.. _scalar_variable:

Creating a netCDF variable for the master data array
----------------------------------------------------

A multidimensional master data array may be stored in a **scalar
netCDF variable** with the same **data type** as its master array,
whose attributes include the JSON encoded strings of the master data
array parameters. When read, this scalar array variable may then be
converted to a multidimensional array variable after the parameters
have been decoded. Note that the datum of this scalar netCDF variable
is ignored and need not be set.

.. _CFA-variable:

Such a variable is called a **CFA variable** and a file storing CFA
variables is called a **CFA-netCDF file** and should include both 'CF'
and 'CFA' in its global ``conventions`` attribute. A CFA-netCDF file
may contain a mixture of CFA variables and normal netCDF variables, or
even no CFA variables at all. In the latter case, the CFA-netCDF file
is also a CF-netCDF file.

.. _CFA-private-variable:

For partitions whose :ref:`subarray <frame-subarray>` parameter refers
to a sub-array in memory, that sub-array is written to the CFA-netCDF file
itself as a **CFA private variable**. This is a normal netCDF
variable marked as containing the sub-array for a partition of one or
more CFA variables and should not be interpreted otherwise according
to the CF conventions. As this sub-array now exists in a netCDF file,
the relevant CFA variables may refer to this sub-array as for any
netCDF variable, i.e. by specifying the netCDF file name [#f2]_, the
netCDF variable name and its shape. See :ref:`example 4 <example4>`.

.. _Advantages-of-netCDF:

Advantages of netCDF
--------------------

File storage of an aggregated field stored does not need to use netCDF
format (for example, plain text `XML
<http://en.wikipedia.org/wiki/XML>`_ or `CDL
<http://www.unidata.ucar.edu/software/netcdf/docs/netcdf/CDL-Syntax.html>`_
format files would be sufficient) but there some reasons why it is
desirable:

* Nearly all of the discovery metadata in a CFA-netCDF file is easily
  retrievable with standard, unmodified netCDF libraries. The only
  exception is the ordered list netCDF dimension names of a CFA
  variable, and to ameliorate any difficulties associated with this,
  these are stored in their own netCDF property for easy access (see
  the :ref:`cfa_dimensions` property).

* A plain text CDL representation of a CFA-netCDF file is trivial to
  produce with the ncdump utility.

* An important part of the CFA conventions is the ability to mix,
  within the same CFA-netCDF file, normal CF-netCDF variables
  (possibly with very large data arrays) with CFA variables (which may
  contain references to CFA private variables, possibly with very
  large data arrays, in the same file), so a binary format is
  desirable to keep the file size to a minimum.

* A CFA variable may contain any CF property such as cell methods,
  flags, ancillary variables, coordinates, formula_terms, *etc.* By
  using a netCDF file, there is no need to invent new encodings for
  complex field properties which have already been standardized for
  netCDF variables by the CF conventions.

Recommended usage
-----------------

CFA-netCDF files should have the file name extension ".nca". 

It is recommended, though not necessary to write the following types
of variable as normal netCDF variables:

* One dimensional coordinates and their bounds (to facilitate
  discovery).

* Master data arrays with only one partition whose data array would
  otherwise be written to a CFA-netCDF file as a CFA private variable (to
  avoid unnecessary obfuscation).

Examples
--------

The following example files are described by their CDL representation.

Example 3
~~~~~~~~~

A simple CFA-netCDF file::

   netcdf temperature.nca {
   dimensions:
     time = 48 ;
     lat = 64 ;
     lon = 128 ;
   variables: 
     double time(time) ;
       time:long_name = "time" ;
       time:units = "days since 0000-1-1" ;
     double lat(lat) ;
       lat:units = "degrees_north" ;
       lat:standard_name = "latitude" ;
     double lon(lon) ;
       lon:units = "degrees_east" ;
       lon:standard_name = "longitude" ;
     float tas ; 
       tas:standard_name = "air_temperature" ;
       tas:units = "K" ;
       tas:cf_role = "cfa_variable" ;
       tas:cfa_dimensions = "time lat lon" ;
       tas:cfa_array = '{"pmshape": [2],
                         "pmdimensions": ["time"],
                         "Partitions": [{"index": [0],
                                         "data": {"file": "/home/david/test1.nc",
                                                  "shape": [12, 64, 128],
                                                  "ncvar": "tas"  
                                                 },
                                         "location": [[0, 12], [0, 64], [0, 128]],
                                         "format": "netCDF"
                                        },
                                        {"index": [1],
                                         "data": {"file": "/home/david/test2.nc",
                                                  "shape": [36, 64, 128],
                                                  "ncvar": "tas2"
                                                 },
                                         "location": [[12, 48], [0, 64], [0, 128]],
                                         "format": "netCDF"
                                        }
                                       ]
                        }' ;

   // global attributes:
       :Conventions = "CF-1.5 CFA" ;

   data:
   
    time = 164569, 164599.5, 164630.5, 164660, 164689.5, 164720, 164750.5, 
          // etcetera.
   
    lat = -87.8638000488281, -85.0965270996094, -82.3129119873047,
          // etcetera.
    
    lon = 0, 2.8125, 5.625, 8.4375, 11.25, 14.0625, 16.875, 19.6875, 22.5, 
          // etcetera.

Points to note:

* The file specifies two conventions.
* The file contains one CFA variable (``tas``) and three normal
  variables (``time``, ``lat`` and ``lon``).
* The CFA variable stores the master data array's dimensions in a
  separate attribute (``cfa_dimensions``) to facilitate reconstruction
  of a multidimensional variable without having to decode the
  ``cfa_array`` string.
* The ``cfa_array`` string has been split over many lines for enhanced
  readability. New lines are permitted in JSON strings.
* The CFA variable defines its data type and units in the normal
  manner, so that these parameters of the master array may be omitted
  from the ``cfa_array`` attribute.
* The CFA variable may have any CF-netCDF attributes, with no
  restrictions.
* Partition parameters which are the same as their master array may be
  omitted.

.. _example4:

Example 4
~~~~~~~~~

Storing a master data array with an in-memory partition data array::

   netcdf temperature2.nca {
   dimensions:
     time = 48 ;
     lat = 64 ;
     lon = 128 ;
     cfa12 = 12 ;
     cfa64 = 64 ;
     cfa128 = 128 ;
   variables: 
     double time(time) ;
       time:long_name = "time" ;
       time:units = "days since 0000-1-1" ;
     double lat(lat) ;
       lat:units = "degrees_north" ;
       lat:standard_name = "latitude" ;
     double lon(lon) ;
       lon:units = "degrees_east" ;
       lon:standard_name = "longitude" ;
     float tas ; 
       tas:standard_name = "air_temperature" ;
       tas:units = "K" ;
       tas:cf_role = "cfa_variable" ;
       tas:cfa_dimensions = "time lat lon" ;
       tas:cfa_array = '{"pmshape": [2],
                         "pmdimensions": ["time"],
                         "Partitions": [{"index": [0],
                                         "punits" : "K @ 273.15",
                                         "pdimensions": ["lon", "time", lat"],
                                         "flip": ["time"],
                                         "data": {"shape": [128, 12, 64],
                                                  "ncvar": "cfa_45sdf83745"  
                                                 },
                                         "location": [[0, 12], [0, 64], [0, 128]],
                                         "format": "netCDF"
                                        },
                                        {"index": [1],
                                         "data": {"file": "/home/david/test2.nc",
                                                  "shape": [36, 64, 128],
                                                  "ncvar": "tas2"
                                                 },
                                         "location": [[12, 48], [0, 64], [0, 128]],
                                         "format": "netCDF"
                                        }
                                       ]
                        }' ;
     float cfa_45sdf83745(cfa128, cfa12, cfa64) ; 
       cfa_45sdf83745:cf_role = "cfa_private" ;
   
               
   // global attributes:
       :Conventions = "CF-1.5 CFA" ;
       
   data:
   
    time = 164569, 164599.5, 164630.5, 164660, 164689.5, 164720, 164750.5, 
          // etcetera.
   
    lat = -87.8638000488281, -85.0965270996094, -82.3129119873047,
          // etcetera.
   
    lon = 0, 2.8125, 5.625, 8.4375, 11.25, 14.0625, 16.875, 19.6875, 22.5, 
          // etcetera.
   
    cfa_45sdf83745 = -4.5, 3.5, 23.6, -4.45, 13.5, 13.6,
          // etcetera.

Points to note:

* The in-memory partition data array has been written to the file with
  an automatically generated variable name (``cfa_45sdf83745``), which
  has an attribute ``cfa_private`` to mark it as a private variable
  according to the CFA convention.

* The in-memory array had different units and dimension order relative
  to the master array.

* The time dimension of the in-memory array runs in the opposite
  direction to the time dimension of the master data array, but the
  other dimensions run in the same sense as the master array.

* The CFA private variable has dimensions which are only used by it
  and other CFA private variables.

----

.. rubric:: Footnotes

.. [#f1] The shape is required since the sub-array may omit (contain)
         size one dimensions which are (are not) present in the master
         data array. Also, a multi-character string array in memory
         may be different to the shape of the array stored in a netCDF
         file, which may be stored as a character array with an extra
         trailing dimension.

.. [#f2] In this case, though, the file name may be omitted, in which
         case the name of the CFA-netCDF file is assumed. See the
         :ref:`file <file>` attribute.
