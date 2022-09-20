The NCA convention
==================

**A proposal for the efficient netCDF file storage of aggregated data
arrays**

Storing the aggregated data array parameters in a string
--------------------------------------------------------

Since the parameters which completely describe an aggregated data
array may each be cast as one a set of particular basic types, then
these parameters may be easily encoded in a `JSON (JavaScript Object
Notation) <http://www.json.org>`_ string for simple inclusion in a
netCDF file.

JSON is a lightweight data-interchange format which is easy for humans
to read and write and easy for machines to parse and generate. There
are JSON encoders and decoders for every reasonable language. See the
`JSON Wikipedia article <http://en.wikipedia.org/wiki/JSON>`_ for some
good examples of JSON strings.

These basic types recognised by JSON are (with JSON encoded examples):

* string (``'foo'``, ``''``)

* number (``3.14159``, ``0``)

* boolean (``true``, ``false``)

* Undefined (``null``)

* list of any of the basic types (``[1, 'a', ['2, null, {'x': false}]]``)

* associative array with strings for keys and any of the basic types
  for values (``{'x': [4, [true, null]], 'y': 2.71828}``)

With the exception of partition's :ref:`data <frame-data>` parameter,
it is clear that each of the values of the parameters which describe
an aggregated data array is one of these basic types. For example, the
:ref:`pmshape <frame-pmshape>` parameter is a list of numbers.

The partition's :ref:`data <frame-data>` parameter may also be
described by these basic types, but is more complicated. It will be
one of:

1. A reference to section of a file.
2. A reference to an actual array in memory.

In case 1 there is no problem, as it will always be possible to
encapsulate the reference with a collection of these basic types. For
example, if the reference is to a netCDF file variable then all that
is required are its filename (a string), the name of the netCDF
variable containing the sub-array (a string) and its shape (a list of
numbers) [#f1]_.

In case 2, the sub-array in memory gets stored in a netCDF file
variable, and so case 1 applies. See the notes on :ref:`private NCA
variables <private-NCA-variable>` for details.

.. _scalar_variable:

Creating a netCDF variable for the aggregated data array
--------------------------------------------------------

A multidimensional aggregated data array may be stored in a **scalar
netCDF variable** with the same **data type** as its master array, one
of whose attributes is the JSON encoded string of the aggregated data
array parameters. When read, this scalar array variable may then be
converted to a multidimensional array variable after the parameters
have been decoded. Note that the scalar netCDF variable normal netCDF
datum is ignored and need not be set.

.. _NCA-variable:

Such a variable is called an **NCA variable** (netCDF aggregate
variable) and a file storing NCA variables is called an **NCA file**
(netCDF aggregate file) and should include both 'CF' and 'NCA' in its
global ``conventions`` attribute.

.. _private-NCA-variable:

For partitions whose :ref:`data <frame-data>` parameter refers to a
sub-array in memory, that array must be written to the NCA file as a
**private NCA variable**. This is a normal netCDF variable marked as
containing the sub-array for a partition of one or more NCA
variables. As this sub-array now exists in a netCDF file (the NCA file
itself), the relevant NCA variables may refer to this sub-array as for
any netCDF variable, i.e. by specifying the netCDF file name [#f2]_,
the netCDF variable name and its shape. See :ref:`example 3
<example3>`.

Advantages
^^^^^^^^^^

* NCA files are CF compliant.

* Variables within an NCA file may be a mixture of normal,
  multidimensional array variables and NCA variables.

* Much, if not all, of the discovery metadata in an NCA is independent
  of the aggregated data array convention and so is accessible to all
  netCDF readers.

* Changes to the elements of the aggregated data array may be stored.

* An NCA variable may have any attributes attached to it.

* NCA encoded parameters may be succinct, as there are clear rules on
  taking default values for partitions.

Disadvantages
^^^^^^^^^^^^^

* Software which creates and reads NCA files in their entirety needs
  to be able to cope with aggregated data arrays. `cf-python
  <http://cfpython.bitbucket.org>`_ can do this.

.. _NcMl:

Comparison with NcML aggregation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`NetCDF Markup Language (NcML) aggregation
<http://www.unidata.ucar.edu/software/netcdf/ncml/v2.2/Aggregation.html>`_
doesn't allow:

* Simultaneous aggregation across arbitrarily positioned dimensions.

* Aggregation of arbitrary parts of arrays.

* Aggregation arrays stored in arbitrary formats (in-memory, netCDF,
  PP, *etc.*).

----

**Example 2: A simple NCA file**::

   netcdf temperature.nca {
   dimensions:
          time = 48 ;
          lat = 64 ;
          lon = 128 ;
   variable: 

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
               tas:nca_dimensions = "time lat lon" ;
               tas:nca_array = "{directions': {'lat': false,
                                               'time': true,
                                               'lon': true
                                              },
                                 'pshape': [2],
                                 'pdimensions': ['time'],
                                 'Partitions': [{'index': [0],
                                                 'data': {'file': '/home/david/test1.nc',
                                                          'shape': [12, 64, 128],
                                                          'ncvar': 'tas'  
                                                         },
                                                 'location': [[0, 12], [0, 64], [0, 128]],
                                                 'format': 'netCDF'
                                                },
                                                {'index': [1],
                                                 'data': {'file': '/home/david/test2.nc',
                                                          'shape': [36, 64, 128],
                                                          'ncvar': 'tas2'
                                                         },
                                                 'location': [[12, 48], [0, 64], [0, 128]],
                                                 'format': 'netCDF'
                                                }
                                               ]
                                }" ;

   // global attributes:
                  :Conventions = "CF-1.5 NCA" ;
   data:
   
    time = 164569, 164599.5, 164630.5, 164660, 164689.5, 164720, 164750.5, 
          // etcetera.
   
    lat = -87.8638000488281, -85.0965270996094, -82.3129119873047,
          // etcetera.
    
    lon = 0, 2.8125, 5.625, 8.4375, 11.25, 14.0625, 16.875, 19.6875, 22.5, 
          // etcetera.

Points to note:

* The file specifies two conventions.
* The file contains one NCA variable (``tas``) and three normal
  variables (``time``, ``lat`` and ``lon``).
* The NCA variable stores the aggregated data array's dimensions in a
  separate attribute (``nca_dimensions``) to facilitate
  reconstruction of a multidimensional variable without having to decode
  the ``nca_array`` string.
* The ``nca_array`` string has been split over many lines for
  enhanced readability. Arbitrary new lines are permitted in JSON strings.
* The NCA variable defines its data type and units in the normal manner,
  so that these parameters of the master array may be omitted from
  the ``nca_array`` attribute.
* The NCA variable may have any CF-netCDF attributes, with no
  restrictions.
* Parameters of the partitions which are the same as their master array
  may be omitted.

.. _example3:

----

**Example 3: storing an aggregated data array with an in-memory partition data array**::

   netcdf temperature2.nca {
   dimensions:
           time = 48 ;
           lat = 64 ;
           lon = 128 ;
           nca12 = 12 ;
           nca64 = 64 ;
           nca128 = 128 ;
   variable: 
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
                   tas:nca_dimensions = "time lat lon" ;
                   tas:nca_array = "{directions': {'lat': false,
                                                   'time': true,
                                                   'lon': true
                                                  },
                                     'pshape': [2],
                                     'pdimensions': ['time'],
                                     'Partitions': [{'index': [0],
                                                     'units' : 'K @ 273.15',
                                                     'dimensions': ['lon', 'time', lat'],
                                                     'directions': {'time': false},
                                                     'data': {'shape': [128, 12, 64],
                                                              'ncvar': 'nca_45sdf83745'  
                                                             },
                                                     'location': [[0, 12], [0, 64], [0, 128]],
                                                     'format': 'netCDF'
                                                    },
                                                    {'index': [1],
                                                     'data': {'file': '/home/david/test2.nc',
                                                              'shape': [36, 64, 128],
                                                              'ncvar': 'tas2'
                                                             },
                                                     'location': [[12, 48], [0, 64], [0, 128]],
                                                     'format': 'netCDF'
                                                    }
                                                   ]
                                    }" ;
           float nca_45sdf83745(nca128, nca12, nca64) ; 
                   nca_45sdf83745:nca_private = 1 ;
   
               
   // global attributes:
                   :Conventions = "CF-1.5 NCA" ;
   data:
   
    time = 164569, 164599.5, 164630.5, 164660, 164689.5, 164720, 164750.5, 
          // etcetera.
   
    lat = -87.8638000488281, -85.0965270996094, -82.3129119873047,
          // etcetera.
   
    lon = 0, 2.8125, 5.625, 8.4375, 11.25, 14.0625, 16.875, 19.6875, 22.5, 
          // etcetera.
   
    nca_45sdf83745 = -4.5, 3.5, 23.6, -4.45, 13.5, 13.6,
          // etcetera.

Points to note:

* The in-memory partition data array has been written to the file with an
  automatically generated variable name (``nca_45sdf83745``), which has an
  attribute ``nca_private`` to mark it as a private variable according to
  the NCA convention.
* The in-memory array had different units and dimension order relative to the
  master array.
* The time dimension of the in-memory array is decreasing, but the other
  dimensions run in the same sense as the master array.
* The private NCA variable has dimensions which are only used by private NCA
  variables.

----

Recommended usage
^^^^^^^^^^^^^^^^^

It is recommended, though not necessary to write the following types
of variable as normal (non-NCA) netCDF variables:

* 1-dimensional coordinates and their bounds (to facilitate
  discovery).
* Aggregated data arrays with only one partition whose data array
  would be written to NCA file as a private NCA variable (to avoid
  unnecessary obfuscation).

.. rubric:: Footnotes

.. [#f1] The shape is required since the shape of a multi-character
         string array in memory may be different to the shape of the
         array stored in a netCDF file, which may be stored as a
         character array with an extra trailing dimension.

.. [#f2] In this case, though, the file name may ommitted, in which
         case the name of the NCA file is assumed. See the :ref:`file
         <file>` attribute.
