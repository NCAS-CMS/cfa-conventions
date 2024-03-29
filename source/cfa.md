# NetCDF Climate and Forecast Aggregation (CFA) Conventions

David Hassell, Jonathan Gregory, Neil Massey, Bryan Lawrence, Sadie
Bartholomew

**Version 0.6.2**, 2023-10-04

## Contents

* [Introduction](#Introduction)
* [Terminology](#Terminology)
* [Identification of Conventions](#Identification-of-Conventions)
* [Aggregation variables](#Aggregation-variables)
* [Aggregation instructions](#Aggregation-instructions)
  * [Standardized aggregation instructions](#Standardized-aggregation-instructions)
  * [Non-standardized terms](#Non-standardized-terms)
* [Fragment Storage](#Fragment-Storage)
  * [Units](#Units)
  * [Size 1 dimensions](#Size-1-dimensions)
  * [Compression](#Compression)
  * [Missing values](#Missing-values)
* [Revision History](#Revision-History)
* [References](#References)

#### Examples

* [Example 1a](#Example-1a)
* [Example 1b](#Example-1b)
* [Example 1c](#Example-1c)
* [Example 2](#Example-2)
* [Example 3](#Example-3)
* [Example 4](#Example-4)
* [Example 5](#Example-5)
* [Example 6](#Example-6)
* [Example 7](#Example-7)


## Introduction <a name="Introduction"></a>


The CFA (Climate and Forecast Aggregation) conventions describe how a
netCDF [NetCDF] file can be used to describe a dataset distributed
across multiple other data files. A CFA-compliant aggregation can be
described in netCDF in such way that the describing file does not
contain the data of selected variables ("aggregation variables"),
rather it contains variables with special attributes that provide
instructions on how to create the aggregated variable data as an
aggregation of data from other sources, each of which may be
self-describing datasets in their own right.

In general, netCDF variables always contain their own data and
dimensions. An aggregation variable, however, does not contain its
data&mdash;and therefore nor its dimensions&mdash;in the usual manner
and yet still needs to be viewable as if it were a usual netCDF
variable. This is achieved by encoding the aggregation variable as a
scalar variable (with arbitrary single value) and providing extra
variable attributes from which the variable's true dimensionality can
be inferred, and the variable's aggregated data can be constructed.

The CFA conventions only apply to the data definition of selected
variables, so the CFA conventions have been designed to work alongside
the CF (Climate and Forecast) conventions [CF] that specify the
geophysical meaning of all variables in the file, whether their data
are defined as aggregations or not. The CFA conventions do not
duplicate, extend, nor re-define any of the metadata elements defined
by the CF conventions. However, when CF-compliant software is used for
reading the discovery metadata of a CFA-netCDF file, with no
expectation of reading the data of aggregated variables, a small
extension is needed to allow the correct interpretation of the
dimensionality of aggregation variables (effectively a duplication of
the functionality introduced in CF-1.9 for the CF domain variable,
which has dimensions but no data).


## Terminology <a name="Terminology"></a>

**aggregation variable**

A netCDF variable that does not contain its own data, rather it
contains instructions on how to create its data as an aggregation of
data from other sources.

**aggregated data**

The data of an *aggregation variable* that exists as a set of
instructions on how to build an array from one or more other arrays
stored elsewhere.

**aggregated dimension**

A netCDF dimension that defines a dimension of the aggregated data.

**fragment**

An independent, possibly self-describing, array that defines a
contiguous part of the *aggregated data*. The aggregated data is
composed from a multi-dimensional orthogonal array of fragments.

**fragment dimension**

A dimension of the multi-dimensional orthogonal array of fragments
that defines the *aggregated data*.


## Identification of Conventions <a name="Identification-of-Conventions"></a>

Files that follow this version of the CFA Conventions must indicate
this by setting the NetCDF User's Guide [NUG] defined global attribute
**`Conventions`** to a string value that contains "`CFA-0.6.2`", in
addition to any other conventions that define other aspects of the
file structure and metadata. For instance, a dataset which follows
CF-1.10 and also CFA-0.6.2 could have a **`Conventions`** attribute of
"`CF-1.10 CFA-0.6.2`".


## Aggregation variables <a name="Aggregation-variables"></a>

An *aggregation variable* does not contain its own data, as is usual
for netCDF variables, instead it contains instructions on how to
create its data as an aggregation of data from other sources.

When created by an application program, the data of an aggregation
variable is called its *aggregated data*. The aggregated data is
composed of one or more *fragments*, each of which provides the values
for a unique part of the aggregated data. Each fragment is contained
in a file that is either external to, or shared by, the dataset
containing the aggregation variable; or else is assumed to contain
wholly missing values.

An aggregation variable should be a scalar (i.e. it has no dimensions)
and the value of its single element is immaterial. It acts as a
container for the usual CF attributes that define the data (such as
**`standard_name`** and **`units`**), with the addition of special
attributes that provide instructions on how to create the aggregated
data. The data type of the aggregated data is the same as the data
type of the aggregated variable.

The dimensions of the aggregated data, called the *aggregated
dimensions*, must be stored with the scalar aggregation variable's
**`aggregated_dimensions`** attribute. The value of the
**`aggregated_dimensions`** attribute is a blank separated list of the
aggregated dimension names given in the order which matches the
dimensions of the aggregated data, and following the CF group search
algorithms [CF]. If the aggregated data is scalar then the value of the
**`aggregated_dimensions`** attribute must be an empty string. The
named aggregated dimensions must exist as dimensions in the netCDF
dataset containing the aggregation variable.

The effective dimensions of the aggregation variable are therefore
found by inspecting the **`aggregated_dimensions`** attribute, rather
than the variable's netCDF dimensions, as is usual for CF-netCDF
variables. The presence of an **`aggregated_dimensions`** attribute
will identify an aggregation variable, therefore the
**`aggregated_dimensions`** attribute must not be present on any
variables that do not have aggregated data.

When the correct interpretation of an aggregation variable requires
knowledge of its dimensions, the dimensions listed by the
**`aggregated_dimensions`** attribute should be treated as if they
were the variable dimensions encoded in the usual netCDF manner.

A variable that follows the CF conventions often has attributes that
reference other variables which contain metadata that describes the
parent variable. The dimensions spanned by such metadata variables are
constrained by the parent variable dimensions. For instance, all
variables named by the **`cell_measures`** attribute of a CF data
variable must span a subset of zero or more of the parent variable
dimensions; or the variable named by the **`bounds`** attribute of a
CF coordinate variable must span all of the parent variable dimensions
in the same order, as well as the trailing bounds dimension. An
aggregated variable that follows the CF conventions applies these
rules in a similar manner, except that the metadata variable
dimensions are constrained by the aggregated dimensions, i.e. the
ordered list of dimensions given by the **`aggregated_dimensions`**
attribute. In particular, note that any CF coordinate variable that
shares its name with an aggregated dimension of an aggregated CF data
variable is considered to be part of that variable's CF domain
definition.

The fragments are organised into an orthogonal multidimensional array
with the same number of dimensions as the aggregated data. Each
dimension of this array is called a *fragment dimension*, and
corresponds to the aggregated dimension with the same relative
position in the aggregated data. The size of a fragment dimension is
the number of fragments that span its corresponding aggregated
dimension. For instance, if an aggregated dimension of size 100 has
been fragmented into three fragments spanning 20 values each and one
fragment spanning 40 values, then the corresponding fragment dimension
will have size 4. An aggregated dimension of any size may be
associated with a fragment dimension of size 1. The fragments must be
arranged in the same relative multidimensional order as their
positions in the aggregated data.

## Aggregation instructions <a name="Aggregation-instructions"></a>

The definitions of the fragments and the instructions on how to
aggregate them are provided by the **`aggregated_data`**
attribute. This attribute takes a string value comprising
blank-separated elements of the form "`term: variable`", where `term`
is a case-insensitive keyword that identifies a particular aggregation
instruction, and `variable` is the name of a variable that configures
that instruction for each fragment, following the CF group search
algorithms. The order of elements is not significant.

A variable referenced by the **`aggregated_data`** attribute must span
the fragment dimensions in the same relative order as the aggregated
dimensions, with the possibility of extra trailing dimensions if these
are allowed or required by the aggregation instruction. No other
dimensions may be spanned by variables containing aggregation
instructions.

### Standardized aggregation instructions <a name="Standardized-aggregation-instructions"></a>

The standardized aggregation instruction `term` tokens, all of which are mandatory, define the following variables:

`location`

* For each fragment, identifies the part of the aggregated data for
  which the fragment provides values.

* Names the integer-valued variable containing the sizes, in index
  space, of the fragments along each fragment dimension. From these
  sizes, the location of each fragment in the aggregated data can be
  determined unambiguously.

* When there is at least one aggregated dimension, the `location`
  variable has two dimensions. The size of the first dimension is the
  number of fragment dimensions, and the size of the second dimension
  is the size of the largest fragment dimension.

* The first dimension indexes the fragment dimensions in the same
  order as their corresponding aggregated dimensions, as given by the
  **`aggregated_dimensions`** attribute.

* For each position of a fragment dimension, the second dimension of
  the `location` variable gives, in increasing index space order, the
  number of elements spanned by the fragments that occupy that
  position. For each fragment dimension that is smaller than the
  largest fragment dimension, the second dimension is padded with
  missing values.

* When there are no aggregated dimensions (i.e. when the aggregated
  data is a scalar), the `location` variable must be one-dimensional
  and of size one, and contain the value `1`.

`file`

* For each fragment, identifies the file in which it is stored.

* Names the string-valued variable containing the names of the files containing the fragments.
  Each value identifies the resource which contains the fragment.

  A file name may contain a string substitution defined by the **`substitutions`** attribute of the `file` variable.
  This attribute takes a string value comprising blank-separated elements of the form "`base: substitution`", where `base` is a case-sensitive keyword that defines the part of the name which is to be replaced by the string defined by `substitution`.
  The value of `base` must have the form `${...}`, where `...` represents one or more letters, digits, and underscores.
  The order of elements is not significant.
  The use of substitutions can save space in the file and, in the event that the fragment files have moved from their original locations, the creation of the aggregated data can be facilitated by changing the substitutions rather than the file names given by the `file` variable.

  A file name must be either a fully qualified URI that resolves to a file, or else a file path that is relative to the location of the CFA-netCDF file.
  Which one of these applies is ascertained after any substitutions have been applied, and if it is not a URI then it is assumed to be a relative path.
  Note that relative file paths are taken as being relative to the location of the CFA-netCDF file at the time of inspection, rather than its original location, which may be different.

* An extra trailing dimension may be included to describe multiple
  URIs for the same fragment, any one of which may equally be used to
  fill its position in the aggregated data. In this case, it is up to
  application programs to choose the version of the fragment that it
  finds most preferable. If a fragment has fewer versions than others
  then the trailing dimension must be padded with missing values.

* A missing value in conjunction with a non-missing value in the
  corresponding location of the `address` variable indicates that the
  fragment is stored within a file shared by the dataset containing
  the aggregation variable. If there is a trailing dimension then all
  of that dimension must comprise missing values.

* A fragment that is assumed to contain wholly missing values, and so
  has no file representation, is indicated by a missing value in
  conjunction with a missing value in the corresponding location of
  the `address` variable. If there is a trailing dimension then all of
  that dimension must comprise missing values.

`format`

* For each fragment, identifies the format of the file in which it is
  stored.

* Names the string-valued variable containing the case-insensitive
  file formats for fragments stored in external files.

* The `format` variable must either be scalar or span exactly the same
  dimensions in the same order as the `file` variable.
  
* When the `format` variable spans the same dimensions as the `file`
  variable, missing values must be used whenever the corresponding
  location in the `file` variable is a missing value.
  
* A scalar `format` variable is a convenience feature that may be used
  when all named fragment files have the same format. In this case the
  single value is assumed to apply to all fragments that have a file
  representation, i.e. those fragments that correspond to non-missing
  values in the `file` variable. If the `file` variable contains only
  missing values, then the `format` variable is not used, and so may
  take an arbitrary value.
    
* A fragment in an external netCDF file is signified by a value of
  `nc`.

* Specification of other file formats is allowed, but not described in
  these conventions.

`address`

* For each fragment, identifies the address of the fragment within its
  file.

* The `address` variable must span exactly the same dimensions in the
  same order as the `file` variable.
  
* For a fragment stored in a file external to the dataset containing
  the aggregation variable, addresses must be provided that correspond
  to each named file in the `file` variable. If there is a trailing
  dimension then it must be padded with missing values.

* For a fragment that is stored in a file shared by the dataset
  containing the aggregation variable, exactly one address must be
  provided, and if there is a trailing dimension then the address must
  be stored in its first element and the trailing dimension must be
  padded with missing values.

* A fragment that is assumed to contain wholly missing values, and so
  has no file representation, is indicated by a missing value. If
  there is a trailing dimension then all of that dimension must
  comprise missing values.
  
* If the fragment is stored in a file shared by the dataset containing
  the aggregation variable then the address is that variable's name,
  otherwise addresses are dependent on the format of the fragment's
  external file.

* For a fragment in a netCDF file, the address is the name of the
  variable that contains the fragment, following the CF group search
  algorithms [CF] within the file that contains the fragment.

* A scalar `address` variable is a convenience feature that may be
  used when all named fragment files have identical addresses. In this
  case the single value is assumed to apply to all fragments that have
  a file representation, i.e. those fragments that correspond to
  non-missing values in the `file` variable. If the `file` variable
  contains only missing values, then the `address` variable is not
  used, and so may take an arbitrary value.
    
* Addressing for other file formats is allowed, but not described in
  these conventions.


#### Example 1a <a name="Example-1a"></a>

*An aggregated data variable whose aggregated data comprises two
fragments. Each fragment spans half of the aggregated `time` dimension
and the whole of the other three aggregated dimensions, and is stored
in an external netCDF file in a variable call `temp`. The fragment
URIs define file locations relative to the CFA-netCDF file. Both
fragment files have the same format, so the `format` variable can be
stored as a scalar variable.*

    dimensions:
      // Aggregated dimensions
      time = 12 ;
      level = 1 ;
      latitude = 73 ;
      longitude = 144 ;
      // Fragment dimensions
      f_time = 2 ;
      f_level = 1 ;
      f_latitude = 1 ;
      f_longitude = 1 ;
      // Extra dimensions
      i = 4 ;
      j = 2 ;
    variables:
      // Data variable
      double temp ;
        temp:standard_name = "air_temperature" ;
        temp:units = "K" ;
        temp:cell_methods = "time: mean" ;
        temp:aggregated_dimensions = "time level latitude longitude" ;
        temp:aggregated_data = "location: aggregation_location
                                file: aggregation_file
                                format: aggregation_format
                                address: aggregation_address" ;
      // Coordinate variables
      double time(time) ;
        time:standard_name = "time" ;
        time:units = "days since 2001-01-01" ;
      double level(level) ;
        level:standard_name = "height_above_mean_sea_level" ;
        level:units = "m" ;
      double latitude(latitude) ;
        latitude:standard_name = "latitude" ;
        latitude:units = "degrees_north" ;
      double longitude(longitude) ;
        longitude:standard_name = "longitude" ;
        longitude:units = "degrees_east" ;
      // Aggregation definition variables			 	  
      int aggregation_location(i, j) ;
      string aggregation_file(f_time, f_level, f_latitude, f_longitude) ;
      string aggregation_format ;
      string aggregation_address(f_time, f_level, f_latitude, f_longitude) ;

    // global attributes:
      :Conventions = "CF-1.10 CFA-0.6.2" ;
    data:
      temp = _ ;
      time = 0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334 ;
      aggregation_location = 6, 6,
                             1, _,
                             73, _,
                             144, _ ;
      aggregation_file = "January-June.nc", "July-December.nc" ;
      aggregation_format = "nc" ;
      aggregation_address = "temp", "temp" ;

### Non-standardized terms <a name="Non-standardized-terms"></a>

Any number of non-standardized `term` tokens are allowed, on the understanding that an application reading the aggregation variable may choose to ignore any terms that it does not understand or which are irrelevant for its purpose.
Example use cases for non-standardized terms could be:

* To provide extra aggregation instructions that enable the aggregation of fragments stored in a file format for which the standardized `address` term alone is insufficient to identify the fragment's data.

* To provide a means of storing metadata that relate to the fragments, but which are not necessary for the creation of the aggregated data.
  The corresponding variable does not comprise metadata as recognized by the CF data model [CF], because it spans the fragment dimensions rather than those of the aggregated data.
  However, if there are no extra trailing dimensions then an application could choose to implement it as a field ancillary or auxilary coordinate construct of the CF data model, by broadcasting the scalar values corresponding to each fragment across the aggregated dimensions.
  For instance, it may be convenient for a global attribute that is common to each fragment file to be made available to the aggregated variable, without having to open and inspect the fragment files themselves.


#### Example 1b <a name="Example-1b"></a>

*As for [example 1a](#Example-1a), but with the inclusion of the non-standard aggregation instruction `tracking_id: fragment_id` that defines an attribute for each fragment stored in the `fragment_id` variable.*

    dimensions:
      // Aggregated dimensions
      time = 12 ;
      level = 1 ;
      latitude = 73 ;
      longitude = 144 ;
      // Fragment dimensions
      f_time = 2 ;
      f_level = 1 ;
      f_latitude = 1 ;
      f_longitude = 1 ;
      // Extra dimensions
      i = 4 ;
      j = 2 ;
    variables:
      // Data variable
      double temp ;
        temp:standard_name = "air_temperature" ;
        temp:units = "K" ;
        temp:cell_methods = "time: mean" ;
        temp:aggregated_dimensions = "time level latitude longitude" ;
        temp:aggregated_data = "location: aggregation_location
                                file: aggregation_file
                                format: aggregation_format
                                address: aggregation_address
                                tracking_id: fragment_id" ;		
      // Coordinate variables
      double time(time) ;
        time:standard_name = "time" ;
        time:units = "days since 2001-01-01" ;
      double level(level) ;
        level:standard_name = "height_above_mean_sea_level" ;
        level:units = "m" ;
      double latitude(latitude) ;
        latitude:standard_name = "latitude" ;
        latitude:units = "degrees_north" ;
      double longitude(longitude) ;
        longitude:standard_name = "longitude" ;
        longitude:units = "degrees_east" ;
      // Aggregation definition variables			 	  
      int aggregation_location(i, j) ;
      string aggregation_file(f_time, f_level, f_latitude, f_longitude) ;
      string aggregation_format ;
      string aggregation_address(f_time, f_level, f_latitude, f_longitude) ;
      // Fragment metadata variables
      string fragment_id(f_time, f_level, f_latitude, f_longitude) ;

    // global attributes:
      :Conventions = "CF-1.10 CFA-0.6.2" ;
    data:
      temp = _ ;
      time = 0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334 ;
      aggregation_location = 6, 6,
                             1, _,
                             73, _,
                             144, _ ;
      aggregation_file = "January-June.nc", "July-December.nc" ;
      aggregation_format = "nc" ;
      aggregation_address = "temp", "temp" ;
      fragment_id = "764489ad-7bee-4228", "a4f8deb3-fae1-26b6" ;


#### Example 1c <a name="Example-1c"></a>

*As for [example 1a](#Example-1a), but with the the fragment URIs given as file URIs that are partly defined by a substitution.*

    dimensions:
      // Aggregated dimensions
      time = 12 ;
      level = 1 ;
      latitude = 73 ;
      longitude = 144 ;
      // Fragment dimensions
      f_time = 2 ;
      f_level = 1 ;
      f_latitude = 1 ;
      f_longitude = 1 ;
      // Extra dimensions
      i = 4 ;
      j = 2 ;
    variables:
      // Data variable
      double temp ;
        temp:standard_name = "air_temperature" ;
        temp:units = "K" ;
        temp:cell_methods = "time: mean" ;
        temp:aggregated_dimensions = "time level latitude longitude" ;
        temp:aggregated_data = "location: aggregation_location
                                file: aggregation_file
                                format: aggregation_format
                                address: aggregation_address" ;
      // Coordinate variables
      double time(time) ;
        time:standard_name = "time" ;
        time:units = "days since 2001-01-01" ;
      double level(level) ;
        level:standard_name = "height_above_mean_sea_level" ;
        level:units = "m" ;
      double latitude(latitude) ;
        latitude:standard_name = "latitude" ;
        latitude:units = "degrees_north" ;
      double longitude(longitude) ;
        longitude:standard_name = "longitude" ;
        longitude:units = "degrees_east" ;
      // Aggregation definition variables			 	  
      int aggregation_location(i, j) ;
      string aggregation_file(f_time, f_level, f_latitude, f_longitude) ;
        aggregation_file:substitutions = "${BASE}: file://data1/" ;
      string aggregation_format ;
      string aggregation_address(f_time, f_level, f_latitude, f_longitude) ;

    // global attributes:
      :Conventions = "CF-1.9 CFA-0.6" ;
    data:
      temp = _ ;
      time = 0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334 ;
      aggregation_location = 6, 6,
                             1, _,
                             73, _,
                             144, _ ;
      aggregation_file = "${BASE}January-June.nc", ""${BASE}July-December.nc" ;
      aggregation_format = "nc" ;
      aggregation_address = "temp", "temp" ;

## Fragment Storage <a name="Fragment-Storage"></a>

Each fragment has a canonical form for which:

* The fragment's data provides the values for a unique part of the
  aggregated data, as defined by the `location` term of the
  aggregation instructions.

* The entirety of the fragment's data contributes to the aggregated
  data.

* The fragment's data has the same number of dimensions in the same
  order as the aggregated data.

* The fragment has the same data type as the aggregated variable.

* Each dimension of the fragment's data has the same sense of
  directionality (i.e. the sense in which it increasing in physical
  space) as its corresponding aggregated dimension.

* The fragment has the same physical units as the aggregated variable.

* Any other fragment attributes, as well as all associated metadata
  variables associated with the fragment (such as coordinate
  variables), are ignored by the aggregation variable.

In particular circumstances, however, a fragment may deviate from
these requirements providing that it is possible to unambiguously
convert the fragment to its canonical form prior to it being used
within the aggregated data. This is useful for cases when the
fragments were created independently of the CFA-netCDF file and were
encoded with forms which are equivalent, but not equal to, the
canonical form. The manipulation of a fragment to its canonical form
is carried out by the application program that is managing the
aggregation.

The following fragment manipulations are allowed:

### Units <a name="Units"></a>
 
If a fragment has no defined units then its data is assumed to have
the same units as the aggregated variable.

When a fragment has units that are physically equivalent, but not
identical, to the units of the aggregated variable, then the
fragment's units must be changed to the aggregated variable's
units. This is done by applying the appropriate multiplicative scale
factor and/or additive offset to the fragment's data. For instance, if
the aggregated variable units are degrees Fahrenheit and a fragment
has units of degrees Celsius, then the fragment's units are changed to
degrees Fahrenheit by multiplying the fragment's data by 1.8 and then
adding 32.

For reference time units, the calendar of the aggregation variable and
the calendars of the fragments must also be equivalent.

For instance, if the aggregation variable units are "`days since
2001-01-01`" in the CF standard calendar and a fragment has units of
"`days since 2002-01-01`" in the same calendar, then the reference
time of the fragment's units are changed to the earlier date by adding
365 to the fragment's data.


### Size 1 dimensions <a name="Size-1-dimensions"></a>

A fragment may omit from its data any size 1 dimension for which the
size of the fragment's location along the corresponding aggregated
dimension is also size 1. Any missing size 1 dimensions must be
inserted into the fragment's data in the appropriate positions. For
instance, if the fragment's shape defined by the `location` term of
the aggregation instructions is `(6, 1, 73, 144)`, then the fragment's
data could have shape `(6, 1, 73, 144)` or `(6, 73, 144)`.


### Compression <a name="Compression"></a>

A fragment may be stored in any compressed form, i.e. stored using
fewer bits than its original uncompressed representation, for which
the uncompression algorithm is encoded as part of the fragment's
metadata and so is available to the application program that is
managing the aggregation. The fragment's data must be uncompressed
prior to insertion into the aggregated data. In this case, the
`location` term of the aggregation instructions describes the shape of
the uncompressed fragment and the data type of the fragment is the
data type of its uncompressed data.


### Missing values <a name="Missing-values"></a>

A fragment may use any valid means for defining missing
values. Missing values must be changed to values that the aggregation
variable recognises as missing. It is up to the creator of the dataset
to ensure that non-missing values in a fragment are not registered as
missing in the aggregated data.


#### Example 2 <a name="Example-2"></a>

*An aggregated data variable whose aggregated data comprises two
fragments. Each fragment spans half of the aggregated `time` dimension
and the whole of the other three aggregated dimensions. One fragment
is stored in an external file and the other is stored in the same
dataset as variable `temp2`. The fragment stored in the same dataset
has different but equivalent units to the aggregation variable, and
omits the size 1 `level` dimension.*

    dimensions:
      // Aggregated dimensions
      time = 12 ;
      level = 1 ;
      latitude = 73 ;
      longitude = 144 ;
      // Fragment dimensions
      f_time = 2 ;
      f_level = 1 ;
      f_latitude = 1 ;
      f_longitude = 1 ;
      // Extra dimensions
      i = 4 ;
      j = 2 ;
    variables:
      // Data variable
      double temp ;
        temp:standard_name = "air_temperature" ;
        temp:units = "K" ;
        temp:cell_methods = "time: mean" ;
        temp:aggregated_dimensions = "time level latitude longitude" ;
        temp:aggregated_data = "location: aggregation_location
                                file: aggregation_file
                                format: aggregation_format
                                address: aggregation_address" ;
      // Coordinate variables
      double time(time) ;
        time:standard_name = "time" ;
        time:units = "days since 2001-01-01" ;
      double level(level) ;
        level:standard_name = "height_above_mean_sea_level" ;
        level:units = "m" ;
      double latitude(latitude) ;
        latitude:standard_name = "latitude" ;
        latitude:units = "degrees_north" ;
      double longitude(longitude) ;
        longitude:standard_name = "longitude" ;
        longitude:units = "degrees_east" ;
      // Aggregation definition variables			 	  
      int aggregation_location(i, j) ;
      string aggregation_file(f_time, f_level, f_latitude, f_longitude) ;
      string aggregation_format ;
      string aggregation_address(f_time, f_level, f_latitude, f_longitude) ;
      // Fragment variable
      double temp2(time, latitude, longitude) ;
        temp:units = "degreesC" ;

    // global attributes:
      :Conventions = "CF-1.10 CFA-0.6.2" ;
    data:
      temp = _ ;
      time = 0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334 ;
      aggregation_location = 6, 6,
                             1, _,
                             73, _,
                             144, _ ;
      aggregation_file = "January-June.nc", _ ;
      aggregation_format = "nc" ;
      aggregation_address = "temp", "temp2" ;
      temp2 = 4.5, 3.0, 0.0, -2.6, -5.6, -10.2, ... ;


#### Example 3 <a name="Example-3"></a>

*An aggregated data variable whose aggregated data comprises two
fragments. Each fragment is stored in the same dataset and spans half
of the aggregated `time` dimension and the whole of the `latitude` and
`longitude` dimensions, but does not span the size 1 `level`
dimension. As there are no external files, the `file` variable
contains only missing values, and therefore the `format` variable may
also be a scalar missing value. The fragments and aggregation
definition variables in this case are stored in a child group called
`aggregation`. The `temp2` fragment has different but equivalent units
to the aggregation variable.*

    dimensions:
      // Aggregated dimensions
      time = 12 ;
      level = 1 ;
      latitude = 73 ;
      longitude = 144 ;
     variables:
      // Data variable
      double temp ;
        temp:standard_name = "air_temperature" ;
        temp:units = "K" ;
        temp:cell_methods = "time: mean" ;
        temp:aggregated_dimensions = "time level latitude longitude" ;
        temp:aggregated_data = "location: /aggregation/location
                                file: /aggregation/file
                                format: /aggregation/format
                                address: /aggregation/address" ;
      // Coordinate variables
      double time(time) ;
        time:standard_name = "time" ;
        time:units = "days since 2001-01-01" ;
      double level(level) ;
        level:standard_name = "height_above_mean_sea_level" ;
        level:units = "m" ;
      double latitude(latitude) ;
        latitude:standard_name = "latitude" ;
        latitude:units = "degrees_north" ;
      double longitude(longitude) ;
        longitude:standard_name = "longitude" ;
        longitude:units = "degrees_east" ;

    // global attributes:
      :Conventions = "CF-1.10 CFA-0.6.2" ;
    data:
      temp = _ ;
      time = 0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334 ;

    group: aggregation {
      dimensions:
        // Fragment dimensions
        f_time = 2 ;
        f_level = 1 ;
        f_latitude = 1 ;
        f_longitude = 1 ;
        // Extra dimensions
        i = 4 ;
        j = 2 ;
      variables:
        // Aggregation definition variables
        int location(i, j) ;
        string file(f_time, f_level, f_latitude, f_longitude) ;
        string format ;
        string address(f_time, f_level, f_latitude, f_longitude) ;
        // Fragment variables
        double temp1(time, latitude, longitude) ;
          temp:units = "Kelvin" ;
        double temp2(time, latitude, longitude) ;
          temp2:units = "degreesC" ;

      data:    	   
        location = 6, 6,
                   1, _,
                   73, _,
                   144, _ ;
       file = _, _ ;
       format = _ ;
       address = "temp1", "temp2" ;
       temp1 = 270.3, 272.5, 274.1, 278.5, 280.3, 283.6, ... ;
       temp2 = 4.5, 3.0, 0.0, -2.6, -5.6, -10.2, ... ;
    }

#### Example 4 <a name="Example-4"></a>

*An aggregated data variable whose aggregated data comprises four
fragments. Each fragment spans half of the aggregated `time`
dimension, either the northern or southern hemisphere, and the whole
of the other two aggregated dimensions. The fragments are stored in
external netCDF files. The aggregation definition variables are stored
in a child group called `aggregation`. One of the fragments has been
defined by two different external resources (one "local" and one
"remote"), each of which is provided with its own address within its
file (`temp3` and `t3` respectively). Either of these resources, but
not both, may be used in the aggregated data.*

    dimensions:
      // Aggregated dimensions
      time = 12 ;
      level = 1 ;
      latitude = 73 ;
      longitude = 144 ;
    variables:
      // Data variable
      double temp ;
        temp:standard_name = "air_temperature" ;
        temp:units = "K" ;
        temp:cell_methods = "time: mean" ;
        temp:aggregated_dimensions = "time level latitude longitude" ;
        temp:aggregated_data = "location: /aggregation/location
                                file: /aggregation/file
                                format: /aggregation/format
                                address: /aggregation/address" ;
      // Coordinate variables
      double time(time) ;
        time:standard_name = "time" ;
        time:units = "days since 2001-01-01" ;
      double level(level) ;
        level:standard_name = "height_above_mean_sea_level" ;
        level:units = "m" ;
      double latitude(latitude) ;
        latitude:standard_name = "latitude" ;
        latitude:units = "degrees_north" ;
      double longitude(longitude) ;
        longitude:standard_name = "longitude" ;
        longitude:units = "degrees_east" ;

    // global attributes:
      :Conventions = "CF-1.10 CFA-0.6.2" ;
    data:
      temp = _ ;
      time = 0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334 ;
      latitude = -90.0, -87.5, -85.0, -82.5, -80.0, -77.5, -75.0, ...,
                 ..., 0.0, 2.5, 5.0, 7.5, 10.0, 12.5, 15.0, ...,
                 ..., 75.0, 77.5, 80.0, 82.5, 85.0, 87.5, 90.0 ;

    group: aggregation {
      dimensions:
        // Fragment dimensions
        f_time = 2 ;
        f_level = 1 ;
        f_latitude = 2 ;
        f_longitude = 1 ;
        // Extra dimensions
        i = 4 ;
        J = 2 ;
        k = 2 ;
      variables:
        // Aggregation definition variables			 	  
        int location(i, j) ;
        string file(f_time, f_level, f_latitude, f_longitude, k) ;
        string format ;
        string address(f_time, f_level, f_latitude, f_longitude, k) ;
        // Fragment variable
        double temp2(time, latitude, longitude) ;
          temp2:long_name = "July-December, southern hemisphere" ;
          temp2:units = "degreesC" ;
	      
      data:    	   
        location = 6, 6,
                   1, _,
                   36, 37,
                   144, _ ;
       file = "/remote/January-June_SH.nc", _,
              _, _,
              "/local/January-June_NH.nc", "/remote/January-June_NH.nc",
              "/remote/July-December_NH.nc", _ ;
       format = "nc" ;
       address = "temp1", _,
                 "temp2", _,
                 "temp3", "t3",
                 "temp4", _ ;
       temp2 = 4.5, 3.0, 0.0, -2.6, -5.6, -10.2, ... ;
    }

#### Example 5 <a name="Example-5"></a>

*An aggregated data variable and an aggregated coordinate variable in
the same dataset. There are two external netCDF files, each of which
contains a fragment for each aggregation variable. The aggregation
definition variables for each aggregation variable are stored in
different groups (`aggregation_temp` and `aggregation_time`), but the
`file` terms of the **`aggregated_data`** attributes refer to a
variable in the root group that stores the external file names that
apply to both aggregation variables.*

    dimensions:
      // Aggregated dimensions
      time = 12 ;
      level = 1 ;
      latitude = 73 ;
      longitude = 144 ;
      // Fragment dimensions
      f_time = 2 ;
      f_level = 1 ;
      f_latitude = 1 ;
      f_longitude = 1 ;
      // Extra dimensions
      i = 4 ;
      ii = 1 ;
      j = 2 ;
    variables:
      // Data variable
      double temp ;
        temp:standard_name = "air_temperature" ;
        temp:units = "K" ;
        temp:cell_methods = "time: mean" ;
        temp:aggregated_dimensions = "time level latitude longitude" ;
        temp:aggregated_data = "location: /aggregation_temp/location
                                file: /aggregation_temp/aggregation_file
                                format: aggregation_format
                                address: /aggregation_temp/address" ;
      // Coordinate variables
      double time ;
        time:standard_name = "time" ;
        time:units = "days since 2001-01-01" ;
        time:aggregated_dimensions = "time" ;
        time:aggregated_data = "location: /aggregation_time/location
                                file: /aggregation_time/file
                                format: aggregation_format
                                address: /aggregation_time/address" ;
      double level(level) ;
        level:standard_name = "height_above_mean_sea_level" ;
        level:units = "m" ;
      double latitude(latitude) ;
        latitude:standard_name = "latitude" ;
        latitude:units = "degrees_north" ;
      double longitude(longitude) ;
        longitude:standard_name = "longitude" ;
        longitude:units = "degrees_east" ;
      // Aggregation definition variables
      string aggregation_format ;

    // global attributes:
      :Conventions = "CF-1.10 CFA-0.6.2" ;
    data:
      temp = _ ;
      time = _ ;
      aggregation_format = "nc" ;

    group: aggregation_temp {
      variables:
        // Temperature aggregation definition variables
        int location(i, j) ;
        string file(f_time, f_level, f_latitude, f_longitude) ;
        string address(f_time, f_level, f_latitude, f_longitude) ;

      data:    	   
        location = 6, 6,
                   1, _,
                   73, _,
                   144, _ ;
        file = "January-June.nc", "July-December.nc" ;
        address = "temp", "temp" ;
    }
    
    group: aggregation_time {
      variables:
        // Time aggregation definition variables
        int location(ii, j) ;
        string aggregation_file(f_time) ;
        string address(f_time) ;

      data:    	   
        location = 6, 6 ;
        file = "January-June.nc", "July-December.nc" ;
        address = "time", "time" ;
    }


#### Example 6 <a name="Example-6"></a>

*An aggregation data variable for a collection of discrete sampling
geometry timeseries features that have been compressed by use of a
contiguous ragged array. The three timeseries of air temperature are
each from different geographical locations and comprise four, five,
and six observations respectively, giving a total of fifteen
observations. The timeseries from each location is stored in a
separate external file.*

    dimensions:
      // Aggregated dimensions
      station = 3 ;
      obs = 15 ;
      // Fragment dimensions
      f_station = 3 ;
      // Extra dimensions
      i = 1 ;
      j = 3 ;
    variables:
      // Data variable
      float temp(obs) ;
        temp:standard_name = "air_temperature" ;
        temp:units = "Celsius" ;
        temp:coordinates = "time lat lon alt station_name" ;
        temp:aggregated_dimensions = "obs" ;
        temp:aggregated_data = "location: aggregation_location
                                file: aggregation_file
                                format: aggregation_format
                                address: aggregation_address_temp" ;  
      // Coordinate variables
      float time ;
        time:standard_name = "time" ;
        time:long_name = "time of measurement" ;
        time:units = "days since 1970-01-01" ;
        time:aggregated_dimensions = "obs" ;
        time:aggregated_data = "location: aggregation_location
                                file: aggregation_file
                                format: aggregation_format
                                address: aggregation_address_time" ;     
      float lon(station) ;
        lon:standard_name = "longitude";
        lon:long_name = "station longitude";
        lon:units = "degrees_east";
        lon:aggregated_dimensions = "obs" ;
        lon:aggregated_data = "location: aggregation_location_latlon
                               file: aggregation_file
                               format: aggregation_format
                               address: aggregation_address_lon" ;
      float lat(station) ;
        lat:standard_name = "latitude";
        lat:long_name = "station latitude" ;
        lat:units = "degrees_north" ;
        lat:aggregated_dimensions = "obs" ;
        lat:aggregated_data = "location: aggregation_location_latlon
                               file: aggregation_file
                               format: aggregation_format
                               address: aggregation_address_lat" ;
      // Compression encoding variable
      int row_size(station) ;
        row_size:long_name = "number of observations per station" ;
        row_size:sample_dimension = "obs" ;
      // Aggregation definition variables			 	  
      int aggregation_location(i, j) ;
      int aggregation_location_latlon(i, j) ;
      string aggregation_file(f_station) ;
      string aggregation_format ;
      string aggregation_address_temp(f_station) ;
      string aggregation_address_time(f_station) ;
      string aggregation_address_lat(f_station) ;
      string aggregation_address_lon(f_station) ;

    // global attributes:
      :Conventions = "CF-1.10 CFA-0.6.2" ;
      :featureType = "timeSeries";
    data:
      temp = _ ;    
      time = _ ;
      row_size = 4, 5, 6 ;
      aggregation_location = 3, 4, 5 ;
      aggregation_location_latlon = 1, 1, 1 ;
      aggregation_file = "Harwell.nc", "Abingdon.nc", "Lambourne.nc" ;
      aggregation_format = "nc" ;
      aggregation_address_temp = "tas", "tas", "tas" ;
      aggregation_address_time = "time", "time", "time" ;
      aggregation_address_lat = "lat", "lat", "lat" ;
      aggregation_address_lon = "lon", "lon", "lon" ;


#### Example 7 <a name="Example-7"></a>

*An aggregation data variable whose aggregated data represents 32-bit
floats packed into 16-bit integers. When created, the aggregated data
contains the 16-bit integer values 0, 5958,..., 65539. These may be
subsequently unpacked to the 32-bit float values 270.0, 270.1, ...,
271.10007, which approximate the original, pre-packed 32-bit float
values 270.0, 270.1, ... 271.1.*

    dimensions:
      // Aggregated dimensions
      time = 12 ;
    variables:
      // Data variable
      short temp ;
        temp:standard_name = "air_temperature" ;
        temp:units = "K" ;
        temp:cell_methods = "time: mean" ;
        temp:scale_factor = 1.6785949e-05f
        temp:add_offset = 270.0f ;
        temp:aggregated_dimensions = "time" ;
        temp:aggregated_data = "location: /aggregation/location
                                file: /aggregation/file
                                format: /aggregation/format
                                address: /aggregation/address" ;
      // Coordinate variables
      float time(time) ;
        time:standard_name = "time" ;
        time:units = "days since 2001-01-01" ;

    // global attributes:
      :Conventions = "CF-1.10 CFA-0.6.2" ;
    data:
      temp = _ ;
      time = 0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334 ;
    
    group: aggregation {
      dimensions:
        // Time dimension
        t = 6 ;	
        // Fragment dimensions
        f_time = 2 ;
        // Extra dimensions
        i = 1 ;
        j = 2 ;
      variables:
        // Fragment variables
        short temp1(t) ;
        short temp2(t) ;
        // Aggregation definition variables
        int location(i, j) ;
        string file(f_time) ;
        string format ;	
        string address(f_time) ;
      
      data:    	  
        temp1 = 0, 5958, 11916, 17874, 23832, 29790 ;
        temp2 = 35749, 41707, 47665, 53623, 59581, 65539 ; 
        location = 6, 6 ;
        file = _, _ ;
        format = _ ;
        address = "/aggregation/temp1", "/aggregation/temp2" ;
    }

## Revision History <a name="Revision-History"></a>

**Version 0.6.2**, 2023-10-04

* Relative path names for fragments (https://github.com/NCAS-CMS/cfa-conventions/issues/36)
* Clarify group searches (https://github.com/NCAS-CMS/cfa-conventions/issues/40)
* Clarified the use of non-standardized aggregation instructions, including a new example (https://github.com/NCAS-CMS/cfa-conventions/issues/41)
* Allow the `address` aggregation instruction variable to be scalar (https://github.com/NCAS-CMS/cfa-conventions/issues/45)

**Version 0.6.1**, 2021-12-02

* Corrected the CDL in examples 5 and 6.
* Added links for navigating the document navigation.

**Version 0.5**, 2021

* Prototype version. First introduction of `location`, `file`,
  `format` and `address` variables.

**Version 0.4**, 2014-02-27

* Prototype version.

**Versions 0.1 to 0.3**, 2012 to 2013

* Prototype versions.

## References <a name="References"></a>
[CF] NetCDF Climate and Forecast (CF) Metadata Conventions. https://cfconventions.org

[NetCDF] NetCDF Software Package. UNIDATA Program Center of the University Corporation for Atmospheric Research. http://www.unidata.ucar.edu/netcdf/index.html

[NUG] NetCDF User’s Guide. https://www.unidata.ucar.edu/software/netcdf/documentation/NUG
