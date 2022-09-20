.. role:: raw-html(raw)
   :format: html

.. _CFA-ref:

CFA-netCDF properties
=====================

+-----------------------+-------------------------------------+-----------------------------------+------------------------------------+
| netCDF  properties    | cfa_array attributes                | Partitions attributes             | subarray attributes                |
+=======================+=====================================+===================================+====================================+
| :ref:`cf_role`,       | :ref:`directions   <directions>`,   | :ref:`index       <index>`,       | :ref:`shape        <shape>`,       |
| :ref:`cfa_dimensions`,| :ref:`pmdimensions <pmdimensions>`, | :ref:`location    <location>`,    | :ref:`file         <file>`,        |
| :ref:`cfa_array`      | :ref:`pmdimensions <pmdimensions>`, | :ref:`pdimensions <pdimensions>`, | :ref:`dtype        <dtype>`,       |
|                       | :ref:`pmshape      <pmshape>`,      | :ref:`pdirections <pdirections>`, | :ref:`ncvar        <ncvar>`,       |
|                       | :ref:`base         <base>`,         | :ref:`format      <format>`,      | :ref:`varid        <varid>`,       |
|                       | :ref:`Partitions   <Partitions>`,   | :ref:`punits  	  <punits>`,      | :ref:`file_offset  <file_offset>`, |
|                       |                                     | :ref:`pcalendar   <pcalendar>`,   | :ref:`scale_factor <scale_factor>`,| 
|                       |                                     | :ref:`part        <part>`,        | :ref:`add_offset   <add_offset>`,  |
|                       |                                     | :ref:`subarray 	  <subarray>`     | :ref:`lbpack       <lbpack>`,      | 
|                       |                                     |                                   | :ref:`endian       <endian>`,      | 
|                       |                                     |                                   | :ref:`_FillValue   <_FillValue>`   | 
+-----------------------+-------------------------------------+-----------------------------------+------------------------------------+

.. _cf_role:

cf_role
-------

``cf_role``
   This standard CF property *must* be used to mark the netCDF
   variable as either an :ref:`CFA variable <CFA-variable>` or a
   :ref:`CFA private variable <CFA-private-variable>`.

   It takes values of ``"cfa_variable"`` or ``"cfa_private"`` for a
   CFA variable or CFA private variable respectively.

   ==================  ==
   Examples
   ==================  ==
   ``"cfa_variable"``
   ``"cfa_private"``
   ==================  ==

.. _cfa_dimensions:

cfa_dimensions
--------------

``cfa_dimensions``
   A netCDF property whose value is a string containing the ordered,
   space delimited set of the master array's dimension names. The
   specified dimensions are all defined as netCDF dimensions in the
   CFA-netCDF file.
   
   If missing, an empty sting or a string containing only spaces then
   the master array is assumed to be scalar.

   ==================  ==
   Examples  
   ==================  ==
   ``"time lat lon"``
   ``""``
   ``"  "``
   ==================  ==

   The master array's dimensions are stored outside of the
   :ref:`cfa_array` property in order to provide useful information
   about the master array without having to decode the ``cfa_array``
   string (which may not be possible in all APIs).

.. _cfa_array:

cfa_array
---------

``cfa_array``
   A netCDF property whose value is a JSON encoded associative array
   containing the attributes required for constructing the aggregated
   data array.

   The **dimensions**, **shape**, **dtype**, **units** and
   **calendar** :ref:`master data array parameters
   <Master-data-array-parameters>` are specified elsewhere (by
   properties of the netCDF CFA variable or inferrable from the netCDF
   dimensions in CFA-netCDF file) and so are not required.

   The encoded string may be relatively succinct, as many attributes
   have well defined default values.

   ====================================================================================================================================================================================================================  ==
   Examples  
   ====================================================================================================================================================================================================================  ==
   ``"{'directions': {'lat': false, 'lon': true }, 'Partitions':[{'index': [0], 'data': {'file': 'test1.nc', 'shape': [64, 128], 'ncvar': 'tas', 'varid': 3}, 'location': [[0, 64], [0, 128]], 'format': 'netCDF'}]}"``
   ====================================================================================================================================================================================================================  ==
 
   The JSON *decoded* attributes are described here:

   .. _directions:

   **directions**
      An associative array mapping each dimension of the master array
      to a direction. Each direction is described as ``true``
      (increasing) or ``false`` (decreasing).

      If the master array is a scalar then a boolean rather than an
      associative array is given.

      =============================================  ==
      Examples  
      =============================================  ==
      ``{'time', true, 'lat': false, 'lon', true}``
      ``true``
      ``false``
      =============================================  ==

      .. note:: A scalar master array may have an implied direction,
                e.g. if there are bounds associated with it or if it
                contains a pressure datum.

   .. _pmdimensions:

   **pmdimensions**

      An ordered list of the dimensions along which the master array
      is partitioned. Each of these dimensions is one those specified
      by the :ref:`cfa_dimensions` property (and is therefore defined
      as a netCDF dimension in the CFA-netCDF file).

      A shape of ``[]`` means that the partition matrix is a scalar,
      i.e. the master array comprises a single partition.

      If missing then it is assumed that the master array comprises a
      single partition.

      ===================  ==
      Examples  
      ===================  ==
      ``['lat', 'time']``
      ``['height']``
      ``[]``	
      ===================  ==

   .. _pmshape:

   **pmshape**
      An ordered list containing the number of partitions along each
      partitioned dimension of the master array. The sizes correspond
      to the :ref:`pmdimensions <pmdimensions>` list. This is the
      shape of the partition matrix.

      A shape of ``[]`` means that the partition matrix is a scalar,
      i.e. the master array comprises a single partition.

      If missing then it is assumed that the master array comprises a
      single partition.

      ==========  ==
      Examples  
      ==========  ==
      ``[2, 3]``
      ``[87]``
      ``[]``
      ==========  ==

   .. _base:

   **base**
      A string giving the base for relative file names given by
      :ref:`file <file>`. May be an absolute or relative URL or
      location on the local system. If it is a relative location then
      it assumed to be relative to the local directory or URL base
      containing the CFA-netCDF file.
     
      If set then all file names are assumed to have relative
      paths. If it is an empty string (``''``) then the base is taken
      as the local directory or URL base containing the CFA-netCDF
      file.
 
      If missing then it is assumed that all file names are absolute
      paths (local files or URLs).

      ==========================  ==
      Examples  
      ==========================  ==
      ``'/data/archive'``
      ``'../archive/'``
      ``'http://archive/files'``
      ``''``
      ==========================  ==

   .. _Partitions:

   **Partitions**
      A list whose elements define each of the master array's
      partitions. The order of the list is arbitrary since each
      element contains its (possibly multidimensional) index in the
      partition matrix.
 
      ========================================================================================================================  ==
      Examples  
      ========================================================================================================================  ==
      ``[{'index': [0], 'subarray': {'shape': [2, 3], 'varid': 0, 'ncvar': 'cfa_1bE8EBC2c3'}, 'location': [[0, 1], [0, 2]]}]``
      ========================================================================================================================  ==
 
      Each element of the list is an associative array which specifies
      a partition with the :ref:`following attributes
      <Partition-attributes>`:

.. _Partition-attributes:

Partition attributes
~~~~~~~~~~~~~~~~~~~~

   .. _index:

   **index**
     An ordered list of indices specifying the position of the
     partition in the partition matrix. The indices correspond to the
     :ref:`pmdimensions <pmdimensions>` list.
      
     An index of ``[]`` means that the partition matrix is a scalar,
     i.e. the master array comprises a single partition.

     If missing then it is assumed that the master array comprises a
     single partition.

     ==========  ==
     Examples  
     ==========  ==
     ``[0]``
     ``[2, 1]``
     ``[]``
     ==========  ==
     
     .. note:: Indices count from zero.

   .. _location:

   **location**
      An ordered list of ranges of indices, one for each dimension of
      the master data array, which describe the contiguous section of
      the master data array spanned by this partition. The ranges
      correspond to the :ref:`cfa_dimensions <cfa_dimensions>` list.
    
      Each range is a two-element list giving a *start* and *stop*
      index for its dimension.  For example, the range ``[3, 5]`` is
      equivalent to indices ``3``, ``4`` and ``5``; and the range
      ``[6, 6]`` is equivalent to index ``6``.

      If the master data array is a scalar then it is an empty list.

      =============================  ==
      Examples  
      =============================  ==
      ``[[2, 2], [3, 5], [2, 56]]``
      ``[[0, 0]]``
      ``[]``
      =============================  ==
      
      .. note:: Indices count from zero.

   .. _pdimensions:

   **pdimensions** (*optional*)
      An ordered list of the partition's data array dimension
      names. The specified dimensions are all defined as netCDF
      dimensions in the CFA-netCDF file.
    
      If there are any size 1 dimensions of the partition which are
      not spanned by the master array then the partition's dimensions
      *must* be specified.
    
      If the partition's data array is a scalar then it may be an
      empty list.

      If missing then it is assumed to be equal to dimensions of
      the master array.
    
      =========================  ==
      Examples  
      =========================  ==
      ``['lon', 'time', lat']``
      ``[]``
      =========================  ==
      
      .. note:: If a partition's data array's dimensions are not
                specified and the sub-array is stored in another file
                then it is required *only* that the sub-array has the
                same number of dimensions, with the same physical
                meaning and in the same order as the master array. For
                example, if the sub-array were in a different netCDF
                file, its dimensions may have different names and
                sizes relative to the equivalent dimensions in the
                CFA-netCDF file.

   .. _pdirections:
      
   **pdirections** (*optional*)
      An associative array of the partition's data array dimension
      directions.
    
      Any dimension not specified is assumed to have the same
      direction as the corresponding master array dimension. If there
      are any size 1 dimensions of the partition which are not spanned
      by the master array then their directions *must* be
      specified. The specified dimensions are all defined as netCDF
      dimensions in the CFA-netCDF file.

      =============================================  ==
      Examples  
      =============================================  ==
      ``{'time', true, 'lat': true, 'lon', false}``
      =============================================  ==

      .. note:: A size 1 dimension may have an implied direction,
        	e.g. if there are bounds associated with it or if it
        	contains a pressure datum.

   .. _punits:

   **punits** (*optional*)
      A string containing the units of the partition's data array.
    
      If missing then it is assumed to be equal to units of the
      master array.

      ============  ==
      Examples  
      ============  ==
      ``'m s-1'``
      ``''``
      ============  ==
    
   .. _pcalendar:

   **pcalendar** (*optional*)
      A string containing the calendar of the partition's data
      array.

      If missing then it is assumed to be equal to calendar of the
      master array.

      =============  ==
      Examples  
      =============  ==
      ``'noleap'``
      =============  ==
    
   .. _part:

   **part** (*optional*)
      A string defining the part of the sub-array which comprises the
      partition's data array.

      For each of the partition's dimensions, the string describes the
      indices of the sub-array which define the partition's data
      array. The indices correspond to to the :ref:`pdimensions
      <pdimensions>` list.

      Indices are contained within square or round brackets. Square
      brackets specify a sequence of indices along that
      dimension. Round brackets provide a succinct method of
      describing a strictly monotonic and regularly spaced sequence of
      indices for the dimension via *start*, *stop* and *step*
      values. For example, ``(0, 3, 1)`` is equivalent to ``[0, 1, 2,
      3]`` and ``(10, 4, -2)`` is equivalent to ``[10, 8, 6, 4]``.
    
      If missing or the string ``'[]'`` then it is assumed that the
      partition's data array is the whole of the sub-array.

      ===========================================  ==
      Examples  
      ===========================================  ==
      ``'[(2, 5, 1), [1, 3, 4, 7], (0, 11, 2)]'``
      ``'[[5, 2, 1], (5, 1, -4), (0, 0, 1)]'``
      ``'[]'``
      ===========================================  ==
	
      .. note:: Indices count from zero.

   .. _format:

   .. _subarray:

   **subarray** 

      An associative array giving the attributes required to define
      the sub-array containing the partition's data array. Only a
      subset of these will be required, depending on the storage
      format of the sub-array.

      ==========================================================================================================  ==
      Examples
      ==========================================================================================================  ==
      ``{'shape': [4, 7, 3], 'ncvar': 'cfa_345eA9001D', 'varid': 10}``
      ``{'file': 'temp.nc', 'ncvar': 'tas', 'shape': [1200], 'format': 'netCDF', 'varid': 1}``
      ``{'dtype': 'float', 'file': 'temp.nc', 'ncvar': 'tas', 'shape': [1200], 'format': 'netCDF', 'varid': 6}``
      ``{'shape': [73, 96], 'file': '../data.pp', 'file_offset': 734826, 'lbpack': 1, 'format': 'PP'}``
      ``{'shape': [73, 96], 'file': 'data.pp', 'file_offset': 0, 'scale_factor': 0.01, 'format': 'PP'}``
      ==========================================================================================================  ==

      The keys specify a sub-array with the :ref:`following attributes
      <Sub-array-attributes>`:

.. _Sub-array-attributes:

Sub-array attributes
~~~~~~~~~~~~~~~~~~~~
 
   **format** (*optional*)
      A string naming the format of the file containing the
      partition's sub-array.
    
      If missing then the format is assumed to the same as the
      CFA-netCDF file.
    
      ============  ==
      Examples  
      ============  ==
      ``'netCDF'``
      ``'PP'``
      ============  ==

   .. _shape:

   **shape**
      An ordered list of the sub-array's dimension sizes. The sizes
      correspond to to the :ref:`pdimensions <pdimensions>` list.

      =============  ==
      Examples  
      =============  ==
      ``[4, 7, 3]``
      ``[]``
      =============  ==

   .. _file:
	
   **file** (*optional*)
      A string naming the file which contains the sub-array. May be a
      local file or a URL.

      If :ref:`base <base>` has been set then the file name is assumed
      to be relative to it. If :ref:`base <base>` is an empty string
      (``''``) then the base is taken as the local directory or URL
      base containing the CFA-netCDF file.

      If :ref:`base <base>` has not been set then the file name is
      assumed to be an absolute path (local file or URL).

      If missing or an empty string (``''``) then it is assumed to be
      the CFA-netCDF file itself.
	
      =================================  ==
      Examples  
      =================================  ==
      ``'/home/me/file.nc'``
      ``'../file2.pp'``
      ``'file3.nc'``
      ``'http://archive/data/file.nc'``
      ``'data/file.nc'``
      ``''``
      =================================  ==

   .. _dtype:

   **dtype** (*optional*)
      The data type of the sub-array. Any of the `netCDF data type
      strings
      <http://cf-pcmdi.llnl.gov/documents/cf-conventions/1.6/cf-conventions.html#idp4767584>`_
      are allowed.
      
      If missing then the data type of the master array is assumed.

      ============  ==
      Examples  
      ============  ==
      ``'double'``
      ``'byte'``
      ``'char'``
      ============  ==
	
   .. _ncvar:

   **ncvar** (*optional, but required for netCDF files*)
      A string containing the name of the netCDF variable containing
      the sub-array.

      =========  ==
      Examples  
      =========  ==
      ``'tas'``
      =========  ==

      The attributes :ref:`ncvar <ncvar>` and :ref:`varid <varid>`
      encode exactly the same information, but are provided to allow
      flexibility in the choice of library being used to interpret the
      CFA-netCDF file.

   .. _varid:

   **varid** (*optional, but required for netCDF files*)
      The integer UNIDATA netCDF interface ID of the variable
      containing the sub-array.

      ========  ==
      Examples  
      ========  ==
      ``24``
      ``0``
      ========  ==

      The attributes :ref:`ncvar <ncvar>` and :ref:`varid <varid>`
      encode exactly the same information, but are provided to allow
      flexibility in the choice of library being used to interpret the
      CFA-netCDF file.
      
   .. _file_offset:

   **file_offset** (*optional, not required for netCDF files*)
      The non-negative integer word address of the file where the
      sub-array starts. The start of the file is addressed by zero.

      ===========  ==
      Examples  
      ===========  ==
      ``8460364``
      ===========  ==

      This is not required for a sub-array in a netCDF file, since the
      sub-array's file location is given by the :ref:`ncvar <ncvar>`
      string.

   .. _lbpack:

   **lbpack** (*optional, PP files only*)
      The integer `PP packing code
      <http://badc.nerc.ac.uk/help/formats/pp-format/files/header.txt>`_
      of the sub-array.

      If missing then it is assumed to be ``0`` (unpacked).

      ========  ==
      Examples  
      ========  ==
      ``1``
      ========  ==
	
   .. _endian:

   **endian** (*optional*)
      A string describing the byte order of the sub-array. It takes
      values of ``'big'`` (most-significant byte first) or
      ``'little'`` (least-significant byte first).

      If missing then it is assumed to be ``'big'`` .

      ============  ==
      Examples  
      ============  ==
      ``'little'``
      ``'big'``
      ============  ==

   .. __FillValue:

   **_FillValue** (*optional, not required for netCDF files*)
      The missing data value for the sub-array.

      If missing then it is assumed that there is no missing data in
      the sub-array.

      ===============  ==
      Examples  
      ===============  ==
      ``-1073741824``
      ===============  ==

      For netCDF files, it is assumed that the missing data value will
      be accounted for when reading the file.

   .. _scale_factor:

   **scale_factor** (*optional, non-netCDF files only*)
      The numeric scale factor (`in the CF sense
      <http://cf-pcmdi.llnl.gov/documents/cf-conventions/1.6/cf-conventions.html#packed-data>`_)
      of the sub-array.

      =========  ==
      Examples  
      =========  ==
      ``100.0``
      =========  ==
	
      For netCDF files, it is assumed that the scale factor will be
      accounted for when reading the file. Otherwise, if missing then
      it is assumed to be ``1`` (unscaled).

   .. _add_offset:

   **add_offset** (*optional, non-netCDF files only*)
      The numeric additive offset (`in the CF sense
      <http://cf-pcmdi.llnl.gov/documents/cf-conventions/1.6/cf-conventions.html#packed-data>`_)
      of the sub-array.

      ==========  ==
      Examples  
      ==========  ==
      ``273.15``
      ==========  ==

      For netCDF files, it is assumed that the additive factor will be
      accounted for when reading the file. Otherwise, if missing then
      it is assumed to be ``0`` (no additive offset).
