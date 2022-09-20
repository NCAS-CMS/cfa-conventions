.. _NCA_convention:

The NCA convention
==================

**A proposal for the efficient netCDF file storage of master data
arrays**

Storing the master data array parameters in a string
----------------------------------------------------

Since the parameters which completely describe a master data array may
each be cast as one a set of particular basic types, then these
parameters may be easily encoded in a `JSON (JavaScript Object
Notation) <http://www.json.org>`_ string for simple inclusion in a
netCDF file.

JSON is a lightweight data-interchange format which is easy for humans
to read and write and easy for machines to parse and generate. There
are JSON encoders and decoders for every reasonable language. See the
`JSON Wikipedia article <http://en.wikipedia.org/wiki/JSON>`_ for some
good examples of JSON strings.

The basic types recognised by JSON are (with JSON encoded examples):

* string (``'foo'``, ``''``)

* number (``3.14159``, ``0``)

* boolean (``true``, ``false``)

* undefined (``null``)

* ordered list of any of the basic types (``[1, 'a', ['2, null, {'x':
  false}]]``)

* associative array with strings for keys and any of the basic types
  for values (``{'x': [4, [true, null]], 'y': 2.71828}``)

It clear that, with the exception of the partition's :ref:`sub_array
<frame-sub_array>` parameter, each of the values of the parameters
which describe a master data array is one or a combination of these
basic types. For example, the :ref:`pmshape <frame-pmshape>` parameter
is a list of numbers.

The same is actually true for the partition's :ref:`sub_array
<frame-sub_array>` parameter, although it is not so immediately
obvious. The issue is that it may be a reference to section of a file
or to a multidimensional array in memory. However, in the latter case
the sub-array must be written to a file in order to be stored, so this
reduces to the former case. A file reference is easily encapsulated by
a collection of the basic types. For example, if the reference is to a
netCDF file variable then all that is required are the filename (a
string), the name of the netCDF variable containing the sub-array (a
string) and its shape (a list of numbers) [#f1]_.

Thus, for storage purposes, all of the master data array parameters
may encoded in a single JSON string.

See the :ref:`complete description of netCDF NCA attributes <NCA_ref>`
for details on how each master data array parameter is encoded.

.. _scalar_variable:

Creating a netCDF variable for the master data array
----------------------------------------------------

A multidimensional master data array may be stored in a **scalar
netCDF variable** with the same **data type** as its master array, one
of whose attributes is the JSON encoded string of the master data
array parameters. When read, this scalar array variable may then be
converted to a multidimensional array variable after the parameters
have been decoded. Note that the datum of this scalar netCDF variable
is ignored and need not be set.

.. _NCA-variable:

Such a variable is called an **NCA variable** (netCDF aggregate
variable) and a file storing NCA variables is called an **NCA file**
(netCDF aggregate file) and should include both 'CF' and 'NCA' in its
global ``conventions`` attribute. An NCA file may contain a mixture of
NCA variables and normal netCDF variables (or even no NCA variables at
all).

.. _NCA-private-variable:

For partitions whose :ref:`sub_array <frame-sub_array>` parameter
refers to a sub-array in memory, that sub-array is written to the NCA
file itself as an **NCA private variable**. This is a normal netCDF
variable marked as containing the sub-array for a partition of one or
more NCA variables and should not be interpreted otherwise according
to the CF conventions. As this sub-array now exists in a netCDF file,
the relevant NCA variables may refer to this sub-array as for any
netCDF variable, i.e. by specifying the netCDF file name [#f2]_, the
netCDF variable name and its shape. See :ref:`example 4 <example4>`.

Advantages
~~~~~~~~~~

* Variables within an NCA file may be a mixture of normal,
  multidimensional array variables and NCA variables.

* Much, if not all, of the discovery metadata in an NCA file is
  independent of the master data array convention and so is accessible
  to all netCDF readers.

* Changes to the elements of the master data array may be easily
  stored.

* Arbitrary (not necessarily contiguous) subsapces of master data
  arrays may be stored (see the :ref:`part <part>` parameter).

* An NCA variable may have any attributes attached to it.

* NCA encoded parameter strings may be succinct, as there are clear
  rules on taking default values for partitions.

Disadvantages
~~~~~~~~~~~~~

* Software which creates and reads NCA files in their entirety needs
  to be able to cope with master data arrays. `cf-python
  <http://cfpython.bitbucket.org>`_ can do this.

.. _NcMl:

Comparison with NcML aggregation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`NetCDF Markup Language (NcML) aggregation
<http://www.unidata.ucar.edu/software/netcdf/ncml/v2.2/Aggregation.html>`_
doesn't allow:

* Simultaneous aggregation across arbitrarily positioned dimensions.

* Aggregation of arbitrary parts of arrays.

* Aggregation arrays stored in arbitrary formats (in-memory, netCDF,
  PP, *etc.*).

Recommended usage
~~~~~~~~~~~~~~~~~

It is recommended, though not necessary to write the following types
of variable as normal (non-NCA) netCDF variables:

* 1-dimensional coordinates and their bounds (to facilitate
  discovery).

* Master data arrays with only one partition whose data array would be
  written to NCA file as a NCA private variable (to avoid unnecessary
  obfuscation).

Examples
--------

Example 3
~~~~~~~~~

A simple NCA file::

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
               tas:cf_role = "nca_variable" ;
               tas:nca_dimensions = "time lat lon" ;
               tas:nca_array = "{'directions': {'lat': false,
                                                'time': true,
                                                'lon': true
                                               },
                                 'pmshape': [2],
                                 'pmdimensions': ['time'],
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
* The NCA variable stores the master data array's dimensions in a
  separate attribute (``nca_dimensions``) to facilitate reconstruction
  of a multidimensional variable without having to decode the
  ``nca_array`` string.
* The ``nca_array`` string has been split over many lines for enhanced
  readability. Arbitrary new lines are permitted in JSON strings.
* The NCA variable defines its data type and units in the normal
  manner, so that these parameters of the master array may be omitted
  from the ``nca_array`` attribute.
* The NCA variable may have any CF-netCDF attributes, with no
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
                   tas:cf_role = "nca_variable" ;
                   tas:nca_dimensions = "time lat lon" ;
                   tas:nca_array = "{directions': {'lat': false,
                                                   'time': true,
                                                   'lon': true
                                                  },
                                     'pmshape': [2],
                                     'pmdimensions': ['time'],
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
                   nca_45sdf83745:cf_role = "nca_private" ;
   
               
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

* The in-memory partition data array has been written to the file with
  an automatically generated variable name (``nca_45sdf83745``), which
  has an attribute ``nca_private`` to mark it as a private variable
  according to the NCA convention.

* The in-memory array had different units and dimension order relative
  to the master array.

* The time dimension of the in-memory array is decreasing, but the
  other dimensions run in the same sense as the master array.

* The NCA private variable has dimensions which are only used by it
  and other NCA private variables.

.. rubric:: Footnotes

.. [#f1] The shape is required since the shape of a multi-character
         string array in memory may be different to the shape of the
         array stored in a netCDF file, which may be stored as a
         character array with an extra trailing dimension.

.. [#f2] In this case, though, the file name may be omitted, in which
         case the name of the NCA file is assumed. See the :ref:`file
         <file>` attribute.
