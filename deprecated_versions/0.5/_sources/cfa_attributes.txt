.. role:: raw-html(raw)
   :format: html

.. _CFA-ref:

CFA-netCDF properties
=====================

+-----------------------+-------------------------------------+-----------------------------------+------------------------------------+
| netCDF  properties    | cfa_array attributes                | Partitions attributes             | subarray attributes                |
+=======================+=====================================+===================================+====================================+
| :ref:`cf_role`,       | :ref:`Partitions   <Partitions>`,   | :ref:`index       <index>`,       | :ref:`shape        <shape>`,       |
| :ref:`cfa_dimensions`,| :ref:`pmdimensions <pmdimensions>`, | :ref:`location    <location>`,    | :ref:`file         <file>`,        |
| :ref:`cfa_array`      | :ref:`pmdimensions <pmdimensions>`, | :ref:`pdimensions <pdimensions>`, | :ref:`dtype        <dtype>`,       |
|                       | :ref:`pmshape      <pmshape>`,      | :ref:`reverse     <reverse>`,     | :ref:`ncvar        <ncvar>`,       |
|                       | :ref:`base         <base>`,         | :ref:`subarray 	  <subarray>`,    | :ref:`varid        <varid>`,       |
|                       |                                     | :ref:`punits  	  <punits>`,      | :ref:`file_offset  <file_offset>`, |
|                       |                                     | :ref:`pcalendar   <pcalendar>`,   | :ref:`format       <format>`       |
|                       |                                     | :ref:`part        <part>`,        |                                    |
|                       |                                     |                                   |                                    |
|                       |                                     |                                   |                                    |
|                       |                                     |                                   |                                    | 
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

   ============================================================================================================================================  ==
   Examples  
   ============================================================================================================================================  ==
   ``'{"Partitions":[{"index": [0], "subarray": {"file": "test1.nc", "shape": [64, 128], "ncvar": "tas"}, "location": [[0, 64], [0, 128]]}]}'``
   ============================================================================================================================================  ==
 
   The JSON *decoded* attributes are described here:

   .. _pmdimensions:

   **pmdimensions** (*optional*)
      An ordered list of strings containing the names of the
      dimensions of the partition martrix. Each of these dimensions is
      one those specified by the :ref:`cfa_dimensions` property (and
      is therefore defined as a netCDF dimension in the CFA-netCDF
      file).
      
      The value ``[]`` means that the partition matrix is a scalar,
      i.e. the master array comprises a single partition.

      If missing then it is assumed that the partition matrix is a
      scalar.

      ===================  ==
      Examples  
      ===================  ==
      ``["lat", "time"]``
      ``["height"]``
      ``[]``	
      ===================  ==

   .. _pmshape:

   **pmshape** (*optional*)
      An ordered list of integers containing shape of the partition
      matrix. The integers sizes correspond to the dimensions of the
      partition matrix given by :ref:`pmdimensions <pmdimensions>`.

      The value ``[]`` means that the partition matrix is a scalar,
      i.e. the master array comprises a single partition.

      If missing then it is assumed that each dimension of the
      partition matrix has size ``1``. For example, if
      :ref:`pmdimensions <pmdimensions>` is ``[]`` then the default
      shape is ``[]`` and if :ref:`pmdimensions <pmdimensions>` is
      ``["time", "height"]`` then the default shape is ``[1, 1]``.

      ==========  ==
      Examples  
      ==========  ==
      ``[2, 3]``
      ``[87]``
      ``[1, 1]``
      ``[1]``
      ``[]``
      ==========  ==

   .. _base:

   **base** (*optional*)
      A string giving the base for relative file names given by
      :ref:`file <file>`. May be an absolute or relative URL or
      location on the local system. If it is a relative location then
      it assumed to be relative to the local directory or URL base
      containing the CFA-netCDF file.
     
      If set then all file names are assumed to have relative
      paths. If it is an empty string (``""``) then the base is taken
      as the local directory or URL base containing the CFA-netCDF
      file.
 
      If missing then it is assumed that all file names are absolute
      paths (local files or URLs).

      ==========================  ==
      Examples  
      ==========================  ==
      ``"/data/archive"``
      ``"../archive/"``
      ``"http://archive/files"``
      ``""``
      ==========================  ==

   .. _Partitions:

   **Partitions**
      A list whose elements define each of the master array's
      partitions. The order of the list is arbitrary since each
      element contains its (possibly multidimensional) index in the
      partition matrix.
 
      ============================================================================================================  ==
      Examples  
      ============================================================================================================  ==
      ``[{"index": [0], "subarray": {"shape": [2, 3], "ncvar": "cfa_1bE8EBC2c3"}, "location": [[0, 1], [0, 2]]}]``
      ============================================================================================================  ==
 
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

   **location** (*optional*)
      An ordered list of ranges of indices, one for each dimension of
      the master data array, which describe the contiguous section of
      the master data array spanned by the partition's data array. The
      ranges correspond to the :ref:`cfa_dimensions <cfa_dimensions>`
      list.
    
      Each range is a two-element list giving a *start* and *stop*
      index for its dimension.  For example, the range ``[3, 5]`` is
      equivalent to indices ``3``, ``4`` and ``5``; and the range
      ``[6, 6]`` is equivalent to index ``6``.

      If the master data array is a scalar then it may be an empty list.

      If missing then it is assumed that the partition matrix contains
      exactly one partition whose data array spans the entire master
      array.

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
      An ordered list of the partition's sub-array dimension
      names. The specified dimensions are all defined as netCDF
      dimensions in the CFA-netCDF file.
    
      If there are any size 1 dimensions of the partition which are
      not spanned by the master array then the partition's dimensions
      *must* be specified.
    
      If the partition's data array is a scalar then it may be an
      empty list.

      If missing then it is assumed to be equal to dimensions of
      the master array.
    
      ==========================  ==
      Examples  
      ==========================  ==
      ``["lon", "time", "lat"]``
      ``[]``
      ==========================  ==
      
      .. note:: If a partition's data array dimensions are not
                specified and the sub-array is stored in another file
                then it is required *only* that the sub-array has the
                same number of dimensions, with the same physical
                meaning and in the same order as the master array. For
                example, if the sub-array were in a different netCDF
                file, its dimensions may have different names and
                sizes relative to the equivalent dimensions in the
                CFA-netCDF file.

   .. _reverse:
      
   **reverse** (*optional*)
      An arbitrarily ordered list of zero or more of the partition’s
      data array dimension names. The specified dimensions are all
      defined as netCDF dimensions in the CFA-netCDF file.

      Each specified dimension of the partition's sub-array is assumed
      to run in the opposite direction to the master data array and so
      will need to be reversed prior to use within the master data
      array.

      If missing, or an empty list, then it is assumed that no
      partition sub-array dimensions need to be reversed.

      ==========================  ==
      Examples  
      ==========================  ==
      ``["time"]``
      ``["lon", "time", "lat"]``
      ``[]``
      ==========================  ==

   .. _punits:

   **punits** (*optional*)
      A string containing the units of the partition's data array.
    
      If missing then it is assumed to be equal to units of the
      master array.

      ==========================  ==
      Examples  
      ==========================  ==
      ``"m s-1"``
      ``"days since 2000-12-1"``
      ``"1"``
      ``""``
      ==========================  ==
    
   .. _pcalendar:

   **pcalendar** (*optional*)
      A string containing the calendar of the partition's units.

      If missing then it is assumed to be equal to calendar of the
      master array.

      ============  ==
      Examples  
      ============  ==
      ``"noleap"``
      ============  ==
    
   .. _part:

   **part** (*optional*)
      A string defining the part of the partition's sub-array which
      comprises the partition's data array.

      For each of the partition's dimensions, the string describes the
      indices of the sub-array which define the partition's data
      array. The indices correspond to to the :ref:`pdimensions
      <pdimensions>` list.

      Indices are contained within round or square brackets. Round
      brackets specify a sequence of indices along that
      dimension. Square brackets provide a succinct method of
      describing a strictly monotonic and regularly spaced sequence of
      indices for the dimension via *start*, *stop* and *step*
      values. For example, ``[0, 3, 1]`` is equivalent to ``(0, 1, 2,
      3)`` and ``[10, 4, -2]`` is equivalent to ``(10, 8, 6, 4)``.
    
      If missing, or the string ``"[]"``, then it is assumed that the
      partition's data array spans the entire sub-array.

      ===========================================  ==
      Examples  
      ===========================================  ==
      ``"[[2, 5, 1], (1, 3, 4, 7), [0, 11, 2]]"``
      ``"[(4, 2, 1), [5, 1, -4], [0, 0, 1]]"``
      ``"[]"``
      ===========================================  ==
	
      .. note:: Indices count from zero.

   .. _subarray:

   **subarray** 
      An associative array giving the attributes required to define
      the sub-array containing the partition's data array. Only a
      subset of these will be required, depending on the storage
      format of the sub-array.

      ==================================================================================  ==
      Examples
      ==================================================================================  ==
      ``{"shape": [4, 7, 3], "ncvar": "cfa_345eA9001D", "varid": 10}``
      ``{"file": "temp.nc", "ncvar": "tas", "shape": [1200], "format": "netCDF"}``
      ``{"dtype": "float", "file": "temp.nc", "shape": [1200], "varid": 6}``
      ``{"shape": [73, 96], "file": "../data.pp", "file_offset": 7348, "format": "PP"}``
      ``{"shape": [73, 96], "file": "data.pp", "file_offset": 0, "format": "PP"}``
      ==================================================================================  ==

      The keys specify a sub-array with the :ref:`following attributes
      <Sub-array-attributes>`:

.. _Sub-array-attributes:

Sub-array attributes
~~~~~~~~~~~~~~~~~~~~
 
   .. _format:

   **format** (*optional*)
      A string naming the format of the file containing the
      partition's sub-array.
    
      If missing then the format is assumed to the same as the
      CFA-netCDF file.
    
      ============  ==
      Examples  
      ============  ==
      ``"netCDF"``
      ``"PP"``
      ============  ==
    
   .. _shape:

   **shape**
      An ordered list of the partition's sub-array dimension
      sizes. The sizes correspond to to the :ref:`pdimensions
      <pdimensions>` list.

      =============  ==
      Examples  
      =============  ==
      ``[4, 7, 3]``
      ``[1, 1]``
      ``[]``
      =============  ==

   .. _file:
	
   **file** (*optional*)
      A string naming the file which contains the partition's
      sub-array. May be a local file or a URL.

      If :ref:`base <base>` has been set then the file name is assumed
      to be relative to it. If :ref:`base <base>` is an empty string
      (``""``) then the base is taken as the local directory or URL
      base containing the CFA-netCDF file.

      If :ref:`base <base>` has not been set then the file name is
      assumed to be an absolute path (local file or URL).

      If missing or an empty string (``""``) then it is assumed to be
      the CFA-netCDF file itself.
	
      =================================  ==
      Examples  
      =================================  ==
      ``"/home/me/file.nc"``
      ``"../file2.pp"``
      ``"file3.nc"``
      ``"http://archive/data/file.nc"``
      ``"data/file.nc"``
      ``""``
      =================================  ==

   .. _dtype:

   **dtype** (*optional*)
      The data type of the partition's sub-array. Any of the `netCDF
      data type strings
      <http://cf-pcmdi.llnl.gov/documents/cf-conventions/1.6/cf-conventions.html#idp4767584>`_
      are allowed.
      
      If missing then the data type of the master array is assumed.

      ============  ==
      Examples  
      ============  ==
      ``"double"``
      ``"byte"``
      ``"char"``
      ============  ==
	
   .. _ncvar:

   **ncvar** (*for netCDF files only, optional*)
      A string containing the name of the netCDF variable containing
      the partition's sub-array.

      ====================  ==
      Examples  
      ====================  ==
      ``"tas"``
      ``"latitude"``
      ``"cfa_678df4g745"``
      ====================  ==

      If missing then the :ref:`varid <varid>` attribute must be set.

      If both the :ref:`ncvar <ncvar>` and :ref:`varid <varid>`
      attributes are set then :ref:`varid <varid>` is ignored.

   .. _varid:

   **varid** (*for netCDF files only, optional*)
      The integer UNIDATA netCDF interface ID of the variable
      containing the partition's sub-array.

      ========  ==
      Examples  
      ========  ==
      ``24``
      ``0``
      ========  ==

      If missing then the :ref:`ncvar <ncvar>` attribute must be set.

      If both the :ref:`ncvar <ncvar>` and :ref:`varid <varid>`
      attributes are set then :ref:`varid <varid>` is ignored.

   .. _file_offset:

   **file_offset** (*not required for netCDF files*)
      The non-negative integer byte address of the file where the
      metadata of the partition's sub-array begins. The start of the
      file is addressed by zero.

      ===========  ==
      Examples  
      ===========  ==
      ``0``
      ``8460364``
      ===========  ==
