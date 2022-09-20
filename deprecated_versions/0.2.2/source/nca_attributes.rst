.. role:: raw-html(raw)
   :format: html

.. _NCA_ref:

A complete description of netCDF NCA attributes describing a master data array
==============================================================================

+-----------------------+-------------------------------------+-----------------------------------+------------------------------------+
| netCDF  properties    | nca_array attributes                | Partitions properties             | subarray properties                |
+=======================+=====================================+===================================+====================================+
| :ref:`cf_role`,       | :ref:`directions   <directions>`,   | :ref:`index       <index>`,       | :ref:`pshape       <pshape>`,      |
| :ref:`nca_dimensions`,| :ref:`pmdimensions <pmdimensions>`, | :ref:`location    <location>`,    | :ref:`file         <file>`,        |
| :ref:`nca_array`      | :ref:`pmdimensions <pmdimensions>`, | :ref:`pdimensions <pdimensions>`, | :ref:`pdtype       <pdtype>`,      |
|                       | :ref:`pmshape      <pmshape>`,      | :ref:`pdirections <pdirections>`, | :ref:`ncvar        <ncvar>`,       |
|                       | :ref:`base         <base>`,         | :ref:`format      <format>`,      | :ref:`file_offset  <file_offset>`, |
|                       | :ref:`Partitions   <Partitions>`,   | :ref:`units  	  <units>`,       | :ref:`scale_factor <scale_factor>`,|
|                       |                                     | :ref:`calendar    <calendar>`,    | :ref:`add_offset   <add_offset>`,  | 
|                       |                                     | :ref:`part        <part>`,        | :ref:`lbpack       <lbpack>`       |
|                       |                                     | :ref:`subarray 	  <subarray>`     |                                    |
+-----------------------+-------------------------------------+-----------------------------------+------------------------------------+

.. _cf_role:

cf_role
-------

``cf_role``
   This standard CF property *must* be used to mark the netCDF variable
   as either an :ref:`NCA variable <NCA-variable>` or a :ref:`private
   NCA variable <NCA-private-variable>`.

   It takes values of ``"nca_variable"`` or ``"nca_private"`` for an
   NCA variable or a NCA private variable respectively.

   ==================  ==
   Examples
   ==================  ==
   ``"nca_variable"``
   ``"nca_private"``
   ==================  ==

.. _nca_dimensions:

nca_dimensions
--------------

``nca_dimensions``
   A netCDF property whose value is a string containing the ordered,
   space delimited set of the master array's dimension names. The
   specified dimensions are all defined as netCDF dimensions in the
   NCA file.
   
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
   :ref:`nca_array` attribute in order to provide useful information
   about the master array without having to decode the ``nca_array``
   string (which may not be possible in all APIs).

.. _nca_array:

nca_array
---------

``nca_array``
   A netCDF property whose value is a JSON encoded string containing
   parameters required for constructing the aggregated data array.

   The **dimensions**, **shape**, **dtype**, **units** and
   **calendar** parameters are specified elsewhere (by attributes or
   properties of the netCDF NCA variable or inferrable from the netCDF
   dimensions in NCA file) and so are not required.


   ====================================================================================================================================================================================================================================  ==
   Examples  
   ====================================================================================================================================================================================================================================  ==
   ``"{'directions': {'lat': false, 'lon': true }, 'Partitions':[{'index': [0], 'data': {'file': 'test1.nc', 'shape': [64, 128], 'ncvar': 'tas'}, 'location': [[0, 64], [0, 128]], 'format': 'netCDF'}]}"``
   ====================================================================================================================================================================================================================================  ==
 
   The JSON *decoded* parameters are described here:

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
      by the :ref:`nca_dimensions` attribute (and is therefore defined
      as a netCDF dimension in the NCA file).

      If missing then it is assumed that the master array comprises a
      single partition.

      ===================  ==
      Examples  
      ===================  ==
      ``['lat', 'time']``
      ``['height']``
      ===================  ==

   .. _pmshape:

   **pmshape**
      An ordered list containing the number of partitions along each
      partitioned dimension of the master array. The sizes correspond
      to the :ref:`pmdimensions <pmdimensions>` list. This is the
      shape of the partition matrix.

      If missing then a shape of ``[1]`` is assumed, i.e. the master
      array comprises a single partition.

      ==========  ==
      Examples  
      ==========  ==
      ``[2, 3]``
      ``[87]``
      ==========  ==

   .. _base:

   **base**
      A string giving the base for relative file names given by
      :ref:`file <file>`. May be an absolute or relative URL or
      location on the local system. If it is a relative location then
      it assumed to be relative to the local directory or URL base
      containing the NCA file.
     
      If set then all file names are assumed to have relative
      paths. If it is an empty string (``''``) then the base is taken
      as the local directory or URL base containing the NCA file.
 
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
 
      ============================================================================================================  ==
      Examples  
      ============================================================================================================  ==
      ``[{'index': [0], 'subarray': {'shape': [2, 3], 'ncvar': 'nca_1bE8EBC2c3'}, 'location': [[0, 1], [0, 2]]}]``
      ============================================================================================================  ==
 
      Each element of the list is an associative array which specifies
      a partition with the following parameters:

      .. _index:

      **index**
         An ordered list of indices specifying the position of the
         partition in the partition matrix. The indices correspond to
         the :ref:`pmdimensions <pmdimensions>` list.
       
         ==========  ==
         Examples  
         ==========  ==
         ``[0]``
         ``[2, 1]``
         ==========  ==
         
         .. note:: Indices count from zero.

      .. _location:

      **location**
         An ordered list of ranges of indices, one for each dimension
         of the master data array, which describe the contiguous
         section of the master data array spanned by this
         partition. The ranges correspond to the :ref:`nca_dimensions
         <nca_dimensions>` list.
       
         Each range is a two-element list giving a *start* and *stop*
         index for its dimension.  For example, the range ``[3, 5]``
         is equivalent to indices ``3``, ``4`` and ``5``; and the
         range ``[6, 6]`` is equivalent to index ``6``.

	 If the master data array is a scalar then it is an empty
	 list.

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
         dimensions in the NCA file.
       
         If there are any size 1 dimensions of the partition which are
         not spanned by the master array then the partition's
         dimensions *must* be specified.
       
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
                   specified and the sub-array is stored in another
                   file then it is required *only* that the sub-array
                   has the same number of dimensions, with the same
                   physical meaning and in the same order as the
                   master array. For example, if the sub-array were in
                   a different netCDF file, its dimensions may have
                   different names and sizes relative to the
                   equivalent dimensions in the NCA file.

      .. _format:

      **format** (*optional*)
          A string naming the format of the file containing the
          partition's sub-array.
       
          If missing then the format is assumed to the same as the NCA
          file.
       
          ============  ==
          Examples  
          ============  ==
          ``'netCDF'``
          ``'PP'``
          ============  ==

      .. _pdirections:
         
      **pdirections** (*optional*)
         An associative array of the partition's data array dimension
         directions.
       
       	 Any dimension not specified is assumed to have the same
       	 direction as the corresponding master array dimension. If
       	 there are any size 1 dimensions of the partition which are
       	 not spanned by the master array then their directions *must*
       	 be specified. The specified dimensions are all defined as
       	 netCDF dimensions in the NCA file.

         =============================================  ==
         Examples  
         =============================================  ==
         ``{'time', true, 'lat': true, 'lon', false}``
         =============================================  ==

	 .. note:: A size 1 dimension may have an implied direction,
          	   e.g. if there are bounds associated with it or if
          	   it contains a pressure datum.
      .. _units:

      **units** (*optional*)
         A string containing the units of the partition's data array.
       
         If missing then it is assumed to be equal to units of the
         master array.

         ============  ==
         Examples  
         ============  ==
          ``'m s-1'``
          ``''``
         ============  ==
       
      .. _calendar:

      **calendar** (*optional*)
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
         A string defining the part of the sub-array which comprises
         the partition's data array.

	 For each of the partition's dimensions, the string describes
	 the indices of the sub-array which define the partition's
	 data array. The indices correspond to to the
	 :ref:`pdimensions <pdimensions>` list.

	 Indices are contained within square or round brackets. Square
         brackets specify a sequence of indices along that
         dimension. Round brackets provide a succinct method of
         describing a strictly monotonic and regularly spaced sequence
         of indices for the dimension via *start*, *stop* and *step*
         values. For example, ``(0, 3, 1)`` is equivalent to ``[0, 1,
         2, 3]`` and ``(10, 4, -2)`` is equivalent to ``[10, 8, 6,
         4]``.
       
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

      .. _subarray:

      **subarray** 
         Parameters required to define the sub-array containing the
         partition's data array. Only a subset of these will be
         required, depending on the storage format of the sub-array.

         .. _pshape:

         **pshape**
            An ordered list of the sub-array's dimension sizes. The
      	    sizes correspond to to the :ref:`pdimensions
      	    <pdimensions>` list.

  	    =============  ==
	    Examples  
	    =============  ==
            ``[4, 7, 3]``
            =============  ==

         .. _file:
	
         **file** (*optional*)
             A string naming the file which contains the
             sub-array. May be a local file or a URL.

	     If :ref:`base <base>` has been set then the file name is
	     assumed to be relative to it. If :ref:`base <base>` is an
	     empty string (``''``) then the base is taken as the local
	     directory or URL base containing the NCA file.

	     If :ref:`base <base>` has not been set then the file name
	     is assumed to be an absolute path (local file or URL).

     	     If missing then it is assumed to be the NCA file itself.
     	
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

         .. _pdtype:

         **pdtype** (*optional*)
            The data type of the sub-array. Any of the `netCDF data
            type strings
            <http://cf-pcmdi.llnl.gov/documents/cf-conventions/1.6/cf-conventions.html#idp4767584>`_
            are allowed.
          
     	    If missing then the data type of the master array is
     	    assumed.

	    ============  ==
	    Examples  
	    ============  ==
            ``'double'``
            ``'byte'``
            ``'char'``
            ============  ==
     	
         .. _ncvar:

         **ncvar** (*optional, but required for netCDF files*)
            The name of the netCDF variable containing the sub-array.

 	    =========  ==
	    Examples  
	    =========  ==
            ``'tas'``
            =========  ==

         .. _file_offset:

         **file_offset** (*optional*)
            The non-negative integer word address of the file where
            the sub-array starts. The start of the file is addressed
            by zero.

  	    ===========  ==
	    Examples  
	    ===========  ==
            ``8460364``
            ===========  ==

         .. _lbpack:

         **lbpack** (*optional, PP files only*)
            The `PP integer packing code
            <http://badc.nerc.ac.uk/help/formats/pp-format/files/header.txt>`_
            of the sub-array.
     
            If missing then it is assumed to be ``0`` (unpacked).

 	    ========  ==
	    Examples  
	    ========  ==
            ``1``
            ========  ==
     	
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
     	
            For netCDF files, it is assumed that the scale factor will
            be accounted for when reading the file. Otherwise, if
            missing then it is assumed to be ``1`` (unscaled).

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

     	    For netCDF files, it is assumed that the additive factor
            will be accounted for when reading the file. Otherwise, if
            missing then it is assumed to be ``0`` (no additive
            offset).
