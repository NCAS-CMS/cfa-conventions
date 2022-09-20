.. role:: raw-html(raw)
   :format: html

A complete description of netCDF NCA attributes describing the aggregated data
==============================================================================

.. _cf_role:

cf_role
-------

``cf_role``
   This standard CF property *must* be used to mark the netCDF variable
   as either an :ref:`NCA variable <NCA-variable>` or a :ref:`private
   NCA variable <private-NCA-variable>`.

   It takes values of ``"nca"`` or ``"nca_private"`` for an NCA
   variable or a private NCA variable respectively.

   =================  ==
   Examples
   =================  ==
   ``"nca"``
   ``"nca_private"``
   =================  ==

.. _nca_dimensions:

nca_dimensions
--------------

``nca_dimensions``
   A netCDF property whose value is a string containing the ordered,
   space delimited set of the master array's dimension names. The
   specified dimensions are all defined as netCDF dimensions in the
   NCA file.
   
   If missing, an empty sting or a string contianing only spaces then
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

   .. _shape:

   Note that the **dimensions**, **shape**, **dtype**, **units** and
   **calendar** parameters are specified elsewhere (by attributes of
   the netCDF NCA variable or inferrable from the netCDF dimensions in
   NCA file) and so are not required.

   The JSON *decoded* parameter names and types are described here.

   .. _base:

   **base**
      A string giving the base for any relative file names given by
      :ref:`file <file>`. May be a location on the local system or a
      URL.
 
      If missing then it is assumed that either file names are
      relative to the local directory or URL base containing the NCA
      file or they are absolute paths (local files or URLs).

      ==========================  ==
      Examples  
      ==========================  ==
      ``'/data/archive'``
      ``'../archive/'``
      ``'http://archive/files'``
      ==========================  ==
 
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

   **pmshape**
      An ordered list containing the number of partitions along each
      partitioned dimension of the master array. The sizes correspond
      to the :ref:`pmdimensions <pmdimensions>` list. This is the
      shape of the partition matrix. For

      If missing then a shape of ``[1]`` is assumed, i.e. the master
      array comprises a single partition.

      ==========  ==
      Examples  
      ==========  ==
      ``[2, 3]``
      ``[87]``
      ==========  ==

   **Partitions**
      A list whose elements define each of the master array's
      partitions. The order of the list is arbitrary since each
      element contains its (possibly multidimensional) index in the
      partition matrix.
 
      Each element of the list specifies a partition with the
      following parameters:

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
         An ordered list of the ranges of indices for each dimension
         of the master array which describe the section of the master
         array spanned by this partition's data array.
       
         Each range gives a *start* and *stop* index for its
         dimension.  For example, the range ``[3, 5]`` is equivalent
         to indices ``3``, ``4`` and ``5``; and the range ``[6, 6]``
         is equivalent to index ``6``.

         =============================  ==
         Examples  
         =============================  ==
         ``[[2, 2], [3, 5], [2, 56]]``
         =============================  ==
         
         .. note:: Indices count from zero.

      .. _pdimensions:

      **pdimensions** (*optional*)
         An ordered list of the partition's data array's dimension
         names. The specified dimensions are all defined as netCDF
         dimensions in the NCA file.
       
         If there are any size 1 dimensions of the partition which are
         not spanned by the master array then the partition's
         dimensions *must* be specified.
       
         If missing then it is assumed to be equal to dimensions of
         the master array.
       
         =========================  ==
         Examples  
         =========================  ==
         ``['lon', 'time', lat']``
         =========================  ==
         
         .. note:: Ifa partition's data array's dimensions are not
                   specified and the partition's sub-array is stored
                   in another file then it is required *only* that the
                   sub-array has the same number of dimensions, with
                   the same physical meaning and in the same order as
                   the master array. For example, if the sub-array
                   were in another netCDF file, its dimensions may
                   have different names and sizes relative to the
                   equivalent dimensions in the NCA file.

      **format** (*optional*)
          A string naming the format of the file containing the
          partition's data array.
       
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
         An associative array of the partition's data array's
         dimension directions.
       
       	 Any dimension not specified is assumed to have the same
       	 direction as the corresponding master array dimension. If
       	 there are any size 1 dimensions of the partition which are
       	 not spanned by the master array then their directions *must*
       	 be specified. The specified dimensions are all defined as
       	 netCDF dimensions in the NCA file.

         ==============================================  ==
         Examples  
         ==============================================  ==
          ``{'time', true, 'lat': true, 'lon', false}``
         ==============================================  ==

	 .. note:: A size 1 dimension may have an implied direction,
          	   e.g. if there are bounds associated with it or if
          	   it contains a pressure datum.
       
      **units** (*optional*)
         A string containing the units of the partition's data
         array.
       
         If missing then it is assumed to be equal to units of the
         master array.

         ============  ==
         Examples  
         ============  ==
          ``'m s-1'``
          ``''``
         ============  ==
       
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
         A string defining the part of the partition's sub-array which
         comprises the partition's data array.

	 For each of the partition's dimensions, the string describes
	 the indices which define the partition's data array. The
	 indices correspond to to the :ref:`pdimensions <pdimensions>`
	 list.

	 Indices are contained within square or round brackets. Square
         brackets specify a sequence of indices along that
         dimension. Round brackets describe a strictly monotonic
         sequence of indices for the dimension via *start*, *stop* and
         *step* values. For example, ``(10, 4, -2)`` is equivalent to
         ``[10, 8, 6, 4]``.
       
         If missing then it is assumed that the whole of the sub-array
         applies to this partition (i.e. the data array is identical
         to the sub-array).

         ===========================================  ==
         Examples  
         ===========================================  ==
         ``'[(2, 5, 1), [1, 3, 4, 7], (0, 11, 2)]'``
         ``'[(5, 1, -4), [5, 2, 1], (0, 0, 1)]'``
         ===========================================  ==
	
         .. note:: Indices count from zero.

      .. _data:

      **data**
         Parameters required to define the partition's sub-array. Only
         a subset of these will be required, depending on the storage
         format of the sub-array.
       
         **pshape**
            An ordered list of the partition's sub-array's dimension
      	    sizes. The sizes correspond to to the :ref:`pdimensions
      	    <pdimensions>` list.

  	    =============  ==
	    Examples  
	    =============  ==
            ``[4, 7, 3]``
            =============  ==

  	    .. note:: These are *not* the sizes of partition's data
           	      array.

         .. _file:
	
         **file** (*optional*)
             A string naming the file which holds the partition's
             sub-array. May be a local file or a URL.

 	     If the file name has a relative path (local file or URL)
             then it is assumed to be relative to :ref:`base <base>`,
             if it is set, otherwise to the local directory or URL
             base containing the NCA file.

	     If the file name can not be resolved as a relative path
	     then it is assumed to be an absolute path.

     	     If missing then it is assumed to be the NCA file itself.
     	
	     =================================  ==
	     Examples  
	     =================================  ==
             ``'/home/me/file.nc'``
             ``'../file2.pp'``
             ``'file3.nc'``
             ``'http://archive/data/file.nc'``
             ``'data/file.nc'``
	     =================================  ==

         **pdtype** (*optional*)
            The data type of the partition's sub-array. Any of the
            netCDF data type strings are allowed.
          
     	    If missing then the data type of the master array is
     	    assumed.

	    ============  ==
	    Examples  
	    ============  ==
            ``'double'``
            ``'byte'``
            ``'char'``
            ============  ==
     	
         **ncvar** (*optional*, but required for netCDF files*)
            The name of the netCDF variable containing the partition's
            sub-array.

 	    =========  ==
	    Examples  
	    =========  ==
            ``'tas'``
            =========  ==
     	
         **file_offset** (*optional*)
            The non-negative integer word address of the file where
            the partition's sub-array starts.

  	    ===========  ==
	    Examples  
	    ===========  ==
            ``8460364``
            ===========  ==
     	
         **lbpack** (*optional, PP files only*)
            The PP integer packing code of the array.
     
            If missing then it is assumed to be ``0``, i.e. unpacked.

 	    ========  ==
	    Examples  
	    ========  ==
            ``1``
            ========  ==
     	
         **scale_factor** (*optional, non-netCDF files only*)
            The numeric scale factor (in the CF sense) of the
            partition's sub-array.

            For netCDF files, it is assumed that the scale factor will
            be accounted for when reading the file. Otherwise, if
            missing then it is assumed to be ``1`` (unscaled).
     
 	    =========  ==
	    Examples  
	    =========  ==
            ``100.0``
            =========  ==
     	
         **add_offset** (*optional, non-netCDF files only*)
            The numeric additive offset (in the CF sense) of the
       	    partition's sub-array.
     
     	    For netCDF files, it is assumed that the additive factor
            will be accounted for when reading the file. Otherwise, if
            missing then it is assumed to be ``0`` (no additive
            offset).

 	    ==========  ==
	    Examples  
	    ==========  ==
            ``273.15``
            ==========  ==
