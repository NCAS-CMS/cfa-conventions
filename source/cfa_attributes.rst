Aggregated Variables
====================

*v0.1 - v0.4 David Hassell and Jonathan Gregory, 2012 - 2019*

*v0.6b1 David Hassell and Neil Massey, 2021-02-19*

An *aggregated variable* does not contain its own data, rather it
contains instructions on how to create its data as an aggregation of
data from other sources. <some fluff about motivation here>. When
created by an application program, the data of an aggregated variable
is called its *aggregated data*. The aggregated data is composed of
one or more *fragments*, each of which provides the values for a
unique part of the aggregated data. Each fragment is contained in an
external file, or may be another contained in the same file as the
aggregated variable (i.e the *parent file*).

An aggregated variable should be a scalar (i.e. it has no dimensions)
and the value of its single element is immaterial. It acts as a
container for the usual attributes that define the data (such as
``standard_name`` and ``units``), with the addition of special
attributes that provide instructions on how to create the aggregated
data.

The dimensions of the aggregated data, called the *aggregated
dimensions* must exist as dimensions in the parent file and must be
stored with the ``aggregated_dimensions`` attribute, and the presence
of a ``aggregated_dimensions`` attribute will identify an aggregated
variable. Therefore the ``aggregated_data`` attribute must not be
present on any variables that do not have aggregated data. The value
of the dimensions attribute is a blank separated list of the
aggregated dimension names given in the order which matches the
dimensions of the aggregated data. If the aggregated data is scalar
then the value of the ``aggregated_dimensions`` attribute must be an
empty string.

The fragments must be arrangeable as an orthogonal multidimensional
array with the same number of dimensions as the aggregated
data. Therefore each aggregated dimension has a corresponding
*fragment dimension*, whose size is the number of fragments along an
aggregated dimension, and this size must be no greater than the
aggregated dimension size. For instance, if an aggregated dimension of
size 100 has been fragmented into three fragments spanning 20 values
each and one fragment spanning 40 values, then the corresponding
fragment dimension will have size 4. An aggregated dimension of any
size may be associated with a fragment dimension of size 1.

The descriptions of the fragments and their locations in the
aggregated data are provided by the ``aggregated_data`` attribute. The
``aggregated_data`` attribute takes a string value, the string being
comprised of blank-separated elements of the form ``"term:
variable"``, where ``term`` is a case-insensitive keyword that
identifies an aggregation instruction, and ``variable`` is the name of
a variable that contains the values that configure the instruction for
each fragment. The order of elements is not significant.

Each variable referenced by the ``aggregated_data`` attribute must
span the fragment dimensions in the same relative order as the
aggregated dimensions, so that its values are easily associated with
both individual fragments and the aggregated data. If an instruction
requires multiple values to be provided per fragment then a variable
may also have one or more extra trailing dimensions.

The recognised terms are defined as follows:

| Term      | Definition                                              | 
| ----------| :-------------------------------------------------- | 
| ``index`` |  Names the integer-valued variable containing the
               indices of each fragment along the fragment
               dimensions. For each fragment, this variable stores the
               zero-based index of its position along each fragment
               dimension. Therefore, the variable requires one extra
               trailing dimension whose size is equal to the number of
               fragment dimensions.                                   | 

``location``   Names the integer-valued variable containing the index
               ranges of the aggregated data that correspond to each
               fragment. For each fragment and each aggregated
               dimension, this variable stores the two zero-based
               indices giving the beginning and end of the range of
               indices that defines the position of the fragment
               along the aggregated dimension. Therefore, the variable
               requires two extra trailing dimensions. The size of the
               first is equal to the number of fragment dimensions,
               and the second has size 2.

``file``       Names the string-valued variable containing the URIs of
               the fragments. Each value identifies the external
               resource which contains the fragment. Fragments stored
               as variables in the parent file are represented by
               missing values.

	       The ``file`` term must be omitted if all fragments are
               stored as variables in the parent file.

	       An extra trailing dimension may included to describe
	       multiple URIs for the same fragment, any one of which
	       may equally be used to fill its position in the
	       aggregated data. In this case, it is up to application
	       programs to choose the version of the fragment that it
	       finds the most preferable. If a fragment has fewer
	       versions than others then the trailing dimension must
	       be padded with missing values.
	       
``format``     Names the string-valued variable containing the
               case-insensitive file formats for fragments stored in
               external files.

	       The ``format`` term must be omitted if there is no
	       ``file`` variable.

	       If all fragments are in external netCDF files then the
               ``format`` term may be omitted.
		  
	       A fragment stored as a variable in the parent file is
               represented by a missing value.

	       A fragment in an external netCDF file is signified by a
               value of ``"nc"``.
	       
	       The ``format`` variable must span exactly the same
               dimensions in the same order as the ``file``
               variable. Missing values should be used whenever the
               corresponding location in the ``file`` variable is a
               missing value.
	       
	       Specification of other file formats is allowed, but not
               described in these conventions.

``address``    Names the variable containing each fragment's address
               within its file. If the fragment is in the parent file
               then the address is the variable name, otherwise the
               address is dependent on the format of the fragment's
               external file. For an external netCDF file, the address
               is also the name of the variable that contains the
               fragment.

	       If the ``file`` variable exists then the ``address``
               variable must span the same dimensions in the same
               order, and missing values should be used whenever the
               corresponding location in the ``file`` variable has a
               missing value. Otherwise no extra trailing dimensions
               nor missing values are allowed.
	       
	       Addressing for other file formats is allowed, but not
               described in these conventions.
=============  =======================================================

Additional terms are allowed, with the understanding that application
programs should ignore terms that they do not recognise or which are
irrelevant for their purposes.
   
