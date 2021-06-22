# NetCDF Climate and Forecast Aggregation (CFA) Conventions

David Hassell, Jonathan Gregory, Neil Massey, Bryan Lawrence, Sadie
Bartholomew

**Version 0.6b1**


## Introduction

This document describes the CFA (Climate and Forecast Aggregation)
conventions for storing a file created with the netCDF Application
Programmer Interface [NetCDF] which does not contain the data of
selected variables ("aggregation variables"), rather those variables
contain special attributes that provide instructions on how to create
their data as an aggregation of data from other sources, which may be
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


## Terminology

**aggregation variable**

A netCDF variable that does not contain its own data, rather it
contains instructions on how to create its data as an aggregation of
data from other sources.

**aggregated data**

The data of an *aggregation variable* that exists as a set of
instructions on how to build an array from one or more other arrays
stored elsewhere.

**fragment**

An independent, possibly self-describing, array that defines a
contiguous part of the *aggregated data*. The aggregated data is
composed from a multi-dimensional orthogonal array of fragments.

**fragment dimension**

A dimension of the multi-dimensional orthogonal array of fragments
that defines the *aggregated data*.


## Identification of Conventions

Files that follow this version of the CFA Conventions must indicate
this by setting the NetCDF User's Guide [NUG] defined global attribute
**`Conventions`** to a string value that contains `"CFA-0.6"`, in
addition to any other conventions that define other aspects of the
file structure and metadata. For instance, a dataset which follows
CF-1.9 and also CFA-0.6 could have a **`Conventions`** attribute of
`"CF-1.9 CFA-0.6"`.


## Aggregation variables

A *aggregation variable* does not contain its own data, as is usual
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
dimensions of the aggregated data. If the aggregated data is scalar
then the value of the **`aggregated_dimensions`** attribute must be an
empty string. The named aggregated dimensions must exist as dimensions
in the netCDF dataset containing the aggregation variable.

The effective dimensions of the aggregation variable are therefore
found by inspecting the **`aggregated_dimensions`** attribute, rather
than the variable's netCDF dimensions, as is usual for CF-netCDF
variables. The presence of an **`aggregated_dimensions`** attribute
will identify an aggregation variable, therefore the
**`aggregated_dimensions`** attribute must not be present on any
variables that do not have aggregated data.

The dimensions listed by the **`aggregated_dimensions`** attribute
constrain the dimensions that may be spanned by variables referenced
from any of the other attributes, in the same way that the array
dimensions perform that role for a usual CF-netCDF variable. For
instance, all variables named by the **`cell_measures`** attribute of
an aggregated data variable must span a subset of zero or more of the
dimensions given by the **`aggregated_dimensions`** attribute; or the
variable named by the **`bounds`** attribute of an aggregated CF
coordinate variable must span all of the aggregated dimensions in the
same order, as well as the trailing bounds dimension. Any CF
coordinate variable that shares its name with an aggregated dimension
of an aggregated data variable will be considered as part of the data
variable's CF domain definition.

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

The definitions of the fragments and the instructions on how to
aggregate them are provided by the **`aggregated_data`**
attribute. This attribute takes a string value comprising
blank-separated elements of the form `"term: variable"`, where `term`
is a case-insensitive keyword that identifies a particular aggregation
instruction, and `variable` is the name of a variable that configures
that instruction for each fragment. The order of elements is not
significant.

A variable referenced by the **`aggregated_data`** attribute must span
the fragment dimensions in the same relative order as the aggregated
dimensions, with the possibility of extra trailing dimensions if these
are allowed or required by the aggregation instruction. No other
dimensions may be spanned by variables containing aggregation
instructions.

The value of a `term` token identifying an aggregation instruction may
be standardized or non-standardized, with the understanding that
application programs should ignore terms that they do not recognise or
which are irrelevant for their purposes. The purpose of allowing
non-standardized tokens is to facilitate the aggregation of fragments
stored in other file formats to those described by these
conventions. The standardized aggregation instruction terms, all of
which are mandatory, are:

`location`

* For each fragment, identifies the part of the aggregated data for
  which the fragment provides values.

* Names the integer-valued variable containing the index ranges of the
  aggregated dimensions that correspond to each fragment. For each
  fragment and each aggregated dimension, this variable stores the
  first and last of the range of zero-based indices that defines the
  position of the fragment along the aggregated dimension. Therefore,
  the `location` variable requires two extra trailing dimensions. The
  size of the first is equal to the number of aggregation dimensions,
  and the second has size 2.

`file`

* For each fragment, identifies the file in which it is stored.

* Names the string-valued variable containing the URIs of the files,
  which may be fully qualified URLs, containing the fragments. Each
  value identifies the external resource which contains the fragment.

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

* The `format` variable must span exactly the same dimensions in the
  same order as the `file` variable. Missing values must be used
  whenever the corresponding location in the `file` variable is a
  missing value.
  
* A fragment in an external netCDF file is signified by a value of
  `"nc"`.

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

* For an external netCDF file, the address is the name of the variable
  that contains the fragment.

* Addressing for other file formats is allowed, but not described in
  these conventions.


#### Example 1

An aggregated data variable whose aggregated data comprises two
fragments. Each fragment spans half of the aggregated `time` dimension
and the whole of the other three aggregated dimensions, and is stored
in an external netCDF file in a variable call `temp`.

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
      int aggregation_location(f_time, f_level, f_latitude, f_longitude, i, j) ;
      string aggregation_file(f_time, f_level, f_latitude, f_longitude) ;
      string aggregation_address(f_time, f_level, f_latitude, f_longitude) ;
    data:
      temp = _ ;
      time = 0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334 ;
      aggregation_location = 0, 5,
                             0, 0,
                             0, 72,
                             0, 143,
                             6, 11,
                             0, 0,
                             0, 72,
                             0, 143 ;
      aggregation_file = "January-June.nc", "July-December.nc" ;
      aggregation_format = "nc", "nc" ;
      aggregation_address = "temp", "temp" ;


## Fragment Storage

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

### Units
 
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

For instance, if the aggregation variable units are `"days since
2001-01-01"` in the Gregorian calendar and a fragment has units of
`"days since 2002-01-1"` in the same calendar, then the reference time
of the fragment's units are changed to the earlier date by adding 365
to the fragment's data.


### Missing size 1 dimensions

A fragment may omit from its data any size 1 dimension for which the
size of the fragment's location along the corresponding aggregated
dimension is also size 1. Any missing size 1 dimensions must be
inserted into the fragment's data in the appropriate positions. For
instance, if the fragment's shape defined by the `location` term of
the aggregation instructions is `(6, 1, 73, 144)`, then the fragment's
data could have shape `(6, 1, 73, 144)` or `(6, 73, 144)`.


### Compression

A fragment may be stored in any compressed form, i.e. stored using
fewer bits than its original uncompressed representation, for which
the uncompression algorithm is encoded as part of the fragment's
metadata and so is available to the application program that is
managing the aggregation. The fragment's data must be uncompressed
prior to insertion into the aggregated data. In this case, the
`location` term of the aggregation instructions describes the shape of
the uncompressed fragment and the data type of the fragment is the
data type of its uncompressed data.


### Missing values

A fragment may use any valid means for defining missing
values. Missing values must be changed to values that the aggregation
variable recognises as missing. It is up to the creator of the dataset
to ensure that non-missing values in a fragment are not registered as
missing in the aggregated data.


#### Example 2

An aggregated data variable whose aggregated data comprises two
fragments. Each fragment spans half of the aggregated `time` dimension
and the whole of the other three aggregated dimensions. One fragment
is stored in an external file and the other is stored in the same
dataset as variable `temp2`. The fragment stored in the same dataset
has different but equivalent units to the aggregation variable, and
omits the size 1 `level` dimension.

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
      int aggregation_location(f_time, f_level, f_latitude, f_longitude, i, j) ;
      string aggregation_file(f_time, f_level, f_latitude, f_longitude) ;
      string aggregation_format(f_time, f_level, f_latitude, f_longitude) ;
      string aggregation_address(f_time, f_level, f_latitude, f_longitude) ;
      // Fragment variable
      double temp2(time, latitude, longitude) ;
        temp:units = "degreesC" ;
	
    data:
      temp = _ ;
      time = 0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334 ;
      aggregation_location = 0, 5,
                             0, 0,
                             0, 72,
                             0, 143,
                             6, 11,
                             0, 0,
                             0, 72,
                             0, 143 ;
      aggregation_file = "January-June.nc", _ ;
      aggregation_format = "nc", _ ;
      aggregation_address = "temp", "temp2" ;
      temp2 = 4.5, 3.0, 0.0, -2.6, -5.6, -10.2, ... ;


#### Example 3

An aggregated data variable whose aggregated data comprises two
fragments. Each fragment is stored in the same dataset and spans half
of the aggregated `time` dimension and the whole of the `latitude` and
`longitude` dimensions, but does not span the size 1 `level`
dimension. As there are no external files, the `file` and `format`
variables both contain missing data. The fragments and aggregation
definition variables in this case are stored in a child group called
`aggregation`. The `temp2` fragment has different but equivalent units
to the aggregation variable.

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
        int location(f_time, f_level, f_latitude, f_longitude, i, j) ;
        string file(f_time, f_level, f_latitude, f_longitude) ;
        string format(f_time, f_level, f_latitude, f_longitude) ;
        string address(f_time, f_level, f_latitude, f_longitude) ;
        // Fragment variables
        double temp1(time, latitude, longitude) ;
          temp:units = "Kelvin" ;
        double temp2(time, latitude, longitude) ;
          temp2:units = "degreesC" ;

      data:    	   
        location = 0, 5,
                   0, 0,
                   0, 72,
                   0, 143,
                   6, 11,
                   0, 0,
                   0, 72,
                   0, 143 ;
       file = _, _ ;
       format = _, _ ;
       address = "temp1", "temp2" ;
       temp1 = 270.3, 272.5, 274.1, 278.5, 280.3, 283.6, ... ;
       temp2 = 4.5, 3.0, 0.0, -2.6, -5.6, -10.2, ... ;


#### Example 4

An aggregated data variable whose aggregated data comprises four
fragments. Each fragment spans half of the aggregated `time`
dimension, either the northern or southern hemisphere, and the whole
of the other two aggregated dimensions. The fragments are stored in
external netCDF files. The aggregation definition variables are stored
in a child group called `aggregation`. One of the fragments has been
defined by two different external resources (one "local" and one
"remote"), each of which is provided with its own address within its
file (`temp3` and `t3` respectively). Either of these resources, but
not both, may be used in the aggregated data.

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
        int location(f_time, f_level, f_latitude, f_longitude, i, j) ;
        string file(f_time, f_level, f_latitude, f_longitude, k) ;
	string format(f_time, f_level, f_latitude, f_longitude, k) ;
        string address(f_time, f_level, f_latitude, f_longitude, k) ;
        // Fragment variable
        double temp2(time, latitude, longitude) ;
          temp2:long_name = "July-December, southern hemisphere" ;
          temp2:units = "degreesC" ;
	      
      data:    	   
        location = 0, 5,
                   0, 0,
                   0, 35,
                   0, 143,
                   0, 5,
                   0, 0,
                   36, 72,
                   0, 143,
                   6, 11,
                   0, 0,
                   0, 36,
                   0, 143,
                   6, 11,
                   0, 0,
                   36, 72,
                   0, 143 ;
       file = "/remote/January-June_SH.nc", _,
              _, _,
              "/local/January-June_NH.nc", "/remote/January-June_NH.nc",
              "/remote/July-December_NH.nc", _ ;
       format = "nc, _,
                _, _,
                "nc", "nc",
                "nc", _ ;
       address = "temp1", _,
                 "temp2", _,
                 "temp3", "t3",
                 "temp4", _ ;
       temp2 = 4.5, 3.0, 0.0, -2.6, -5.6, -10.2, ... ;


#### Example 5

An aggregated data variable and an aggregated coordinate variable in
the same dataset. There are two external netCDF files, each of which
contains a fragment for each aggregation variable. The aggregation
definition variables for each aggregation variable are stored in
different groups (`aggregation_temp` and `aggregation_time`), but the
`file` terms of the **`aggregated_data`** attributes refer to a
variable in the root group that stores the external file names that
apply to both aggregation variables.

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
        temp:aggregated_data = "location: /aggregation_temp/location
                                file: aggregation_file
                                format: aggregation_format
                                address: /aggregation_temp/address" ;
      // Coordinate variables
      double time ;
        time:standard_name = "time" ;
        time:units = "days since 2001-01-01" ;
        temp:aggregated_dimensions = "time" ;
        temp:aggregated_data = "location: /aggregation_time/location
                                file: aggregation_file
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
      // Aggregation definition variable
      string aggregation_file(f_time, f_level, f_latitude, f_longitude) ;

    data:
      temp = _ ;
      time = _ ;
      aggregation_file = "January-June.nc", "July-December.nc" ;
      aggregation_format = "nc", "nc" ;

    group: aggregation_temp {
      variables:
        // Aggregation definition variables
        int location(f_time, f_level, f_latitude, f_longitude, i, j) ;
        string address(f_time, f_level, f_latitude, f_longitude) ;

      data:    	   
        location = 0, 5,
                   0, 0,
                   0, 72,
                   0, 143,
                   6, 11,
                   0, 0,
                   0, 72,
                   0, 143 ;
       address = "temp", "temp" ;
    }
    
    group: aggregation_time {
      variables:
        // Aggregation definition variables
        int location(f_time, f_level, f_latitude, f_longitude, i, j) ;
        string address(f_time, f_level, f_latitude, f_longitude) ;

      data:    	   
        location = 0, 5,
                   6, 11 ;
        address = "time", "time" ;
    }


#### Example 6

An aggregation data variable for a collection of discrete sampling
geometry timeseries features that have been compressed by use of a
contiguous ragged array. The three timeseries of air temperature are
each from different geographical locations and comprise four, five,
and six observations respectively, giving a total of fifteen
observations. The timeseries from each location is stored in a
separate external file.

    dimensions:
      // Aggregated dimensions
      station = 3 ;
      obs = 15 ;
      // Fragment dimensions
      f_station = 3 ;
      // Extra dimensions
      i = 1 ;
      j = 2 ;
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
        temp:aggregated_dimensions = "obs" ;
        temp:aggregated_data = "location: aggregation_location
                                file: aggregation_file
                                format: aggregation_format
                                address: aggregation_address_time" ;     
      float lon(station) ;
        lon:standard_name = "longitude";
        lon:long_name = "station longitude";
        lon:units = "degrees_east";
        temp:aggregated_dimensions = "obs" ;
        temp:aggregated_data = "location: aggregation_location_latlon
                                file: aggregation_file
                                format: aggregation_format
                                address: aggregation_address_lon" ;
      float lat(station) ;
        lat:standard_name = "latitude";
        lat:long_name = "station latitude" ;
        lat:units = "degrees_north" ;
        temp:aggregated_dimensions = "obs" ;
        temp:aggregated_data = "location: aggregation_location_latlon
                                file: aggregation_file
                                format: aggregation_format
                                address: aggregation_address_lat" ;
      // Compression encoding variable
      int row_size(station) ;
        row_size:long_name = "number of observations per station" ;
        row_size:sample_dimension = "obs" ;
      // Aggregation definition variables			 	  
      int aggregation_location(f_station, i, j) ;
      string aggregation_file(f_station) ;
      string aggregation_address_temp(f_station) ;
      string aggregation_address_time(f_station) ;
      string aggregation_address_lat(f_station) ;
      string aggregation_address_lon(f_station) ;

    // global attributes:
      :featureType = "timeSeries";
    data:
      temp = _ ;    
      time = _ ;
      row_size = 4, 5, 6 ;
      aggregation_location = 0, 3,
                             4, 8,
                             9, 14 ;
      aggregation_location_latlon = 0, 0
                                    1, 1
                                    2, 2 ;
      aggregation_file = "Harwell.nc", "Abingdon.nc", "Lambourne.nc" ;
      aggregation_format = "nc", "nc", "nc" ;
      aggregation_address_temp = "tas", "tas", "tas" ;
      aggregation_address_time = "time", "time", "time" ;
      aggregation_address_lat = "lat", "lat", "lat" ;
      aggregation_address_lon = "lon", "lon", "lon" ;


#### Example 7

An aggregation data variable whose aggregated data represents 32-bit
floats packed into 16-bit integers. When created, the aggregated data
contains the 16-bit integer values 0, 5958,..., 65539. These may be
subsequently unpacked to the 32-bit float values 270.0, 270.1, ...,
271.10007, which approximate the original, pre-packed 32-bit float
values 270.0, 270.1, ... 271.1.

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
        int location(f_time, i, j) ;
        string file(f_time) ;
        string format(f_time) ;	
        string address(f_time) ;
      
      data:    	  
        temp1 = 0, 5958, 11916, 17874, 23832, 29790 ;
        temp2 = 35749, 41707, 47665, 53623, 59581, 65539 ; 
        location = 0, 5,
                   6, 11,
        file = _, _ ;
        format = _, _ ;
        address = "/aggregation/temp1", "/aggregation/temp2" ;
    }

## Revision History

**Versions 0.1 to 0.3, 2012 to 2013**

Prototype versions

**Version 0.4, 2014-02-27**

Prototype version

**Version 0.5, 2021**

Prototype version. First introduction of `location`, `file`, `format`
and `address` variables.

**Version 0.6, *Date to be fixed*

First stable release.

## References

[CF] NetCDF Climate and Forecast (CF) Metadata Conventions. https://cfconventions.org

[NetCDF] NetCDF Software Package. UNIDATA Program Center of the University Corporation for Atmospheric Research. http://www.unidata.ucar.edu/netcdf/index.html

[NUG] NetCDF User’s Guide. https://www.unidata.ucar.edu/software/netcdf/documentation/NUG
