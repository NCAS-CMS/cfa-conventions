cfa
===

The dimensions of the variable must be stored with the
``cfa_dimensions`` attribute, and the presence of a ``cfa_dimensions``
attribute will identify the variable as one with an aggregated
array. Therefore the ``cfa_dimensions`` attribute must not be present
on any variables that are do not have an aggregated array. The value
of the dimensions attribute is a blank separated list of the dimension
names given in the order which matches the dimensions of the
aggregated array. If the variable is scalar then the value of the
``cfa_dimensions`` attribute must be an empty string.

The aggregated array is composed of one or more fragments, each of
which provides the data for a unique part of the aggregated array. The
aggregated array is fragmented along each of its dimensions, even if a
dimension has size one, so the variable is associated with as many
*fragment dimensions* as it has dimensions. The size of a fragment
dimension gives the number of fragments along its corresponding
dimension. Each fragment is either represented by another variable in
the file, or is contained in an external file.

The locations and arrangement of the fragments is provided by the
``cfa_array`` attribute. The ``cfa_array`` attribute takes a string
value, the string being comprised of blank-separated elements of the
form ``"term: variable"``, where ``term`` is a case-insensitive
keyword that represents a terms in the definition of the aggregation,
and ``variable`` is the name of the variable in the file that contains
data for each fragment. The order of elements is not significant.

Each variable referenced by the ``cfa_array`` attribute defines one
aspect of the all of the fragments, and so must span the fragment
dimensions in the same relative order as the dimensions. Some of the
variable may also have extra trailing dimensions, if required. The
recognised terms are defined as follows:

=============  =======================================================
Term           Definition
=============  =======================================================
``fragment``   Names the integer-valued variable containing the
               position of each fragment along each fragment
               dimension. For each fragment, this variable stores
               the zero-based index of its position along each
               fragment dimension. Therefore, the variable
               requires one extra trailing dimension whose size is
               equal to the number of fragment dimensions.

``index``      Names the integer-valued variable containing the
               indices of the aggregated array corresponding to
               each fragment. For each fragment, this variable
               stores the zero-based first and last of the range of
               indices that defines its position along each
               dimension. Therefore, the variable requires two
               extra trailing dimensions. The size of the first is
               equal to the number of fragment dimensions, and the
               second has size 2.

``file``       Names a string-valued variable containing the URIs of
               the fragments. Each value identifies the external
               resource which contains the fragment. A string-valued
               variable containing the URIs of fragments stored in
               external files. Fragments stored as variables in this
               file are represented by missing values, as they have no
               external representation.

	       An extra trailing dimension may included to describe
	       multiple URIs for the same fragment, any one of which
	       may equally be used to fill its position in the
	       aggregated array. In this case, it is up to application
	       programs to choose the version of the fragment it finds
	       the most preferable. If a fragment has fewer versions
	       than some others, then missing values should be used
	       instead of URIs.
		  
	       If all fragments are stored as variables in
               this file then ``file`` is optional.

``format``     Names a string-valued variable describing the
               case-insensitive file formats for fragments stored
               in external files. Fragments stored as variables in
               this file are represented by missing values. A
               fragment in a netCDF file is signified by an entry
               of ``"nc"``.
	       
	       The ``format`` variable must span exactly the same
               dimensions in the same order as the ``file`` variable,
               if the latter is present.

	       Specification of other file formats is allowed, but not
               described in these conventions.

	       If all fragments are stored as variables in this file
               or in external netCDF files then ``format`` is
               optional.
		  
``address``    Names a variable containing, for each fragment, its
               address within its file. If the fragment is in the same
               file then the address is the variable name, otherwise
               the address is dependent on the format of the
               fragment's external files. For each external netCDF
               file, the address is the name of the variable
               containing that fragment's array.

	       Addressing for other file formats is allowed, but not
               described in these conventions.

	       The ``address`` variable must span the same dimensions
               in the same order as the ``file`` variable, if the
               latter is present. Missing values should be used when
               there is no corresponding URI.	       
=============  =======================================================

Additional terms are allowed, with the understanding that application
programs should ignore terms that they do not recognise or which are
irrelevant for their purposes.
   
