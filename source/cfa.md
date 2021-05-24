# Aggregated Variables

*v0.1 - v0.4 David Hassell and Jonathan Gregory, 2012 - 2019*

*v0.6b1 David Hassell, Jonathan Gregory, Neil Massey, Bryan
 Lawrence and Sadie Bartholomew 2021-02-18 - 2021-04-30*

An *aggregated variable* does not contain its own data, rather it
contains instructions on how to create its data as an aggregation of
data from other sources. When created by an application program, the
data of an aggregated variable is called its *aggregated data*. The
aggregated data is composed of one or more *fragments*, each of which
provides the values for a unique part of the aggregated data. Each
fragment is contained in either an external file; or is described by
another variable contained in the same file as the aggregated variable
(i.e. the *parent file*); or else is assumed to contain wholly missing
values.

An aggregated variable should be a scalar (i.e. it has no dimensions)
and the value of its single element is immaterial. It acts as a
container for the usual attributes that define the data (such as
**`standard_name`** and **`units`**), with the addition of special
attributes that provide instructions on how to create the aggregated
data. The data type of the aggregated data is the same as the data
type of the aggregated variable.

An aggregated variable must not define any of techniques for the
reduction of dataset size, such as (but not limited to) packing,
compression by gathering, discrete sampling geometry ragged array
representations, etc. However, any individual fragment may be packed
or compressed, with the understanding that the fragment's data will be
unpacked or uncompressed prior to use in the aggregated data.

The dimensions of the aggregated data, called the *aggregated
dimensions*, must exist as dimensions in the parent file and must be
stored with the **`aggregated_dimensions`** attribute. The presence of
an **`aggregated_dimensions`** attribute will identify an aggregated
variable, therefore the **`aggregated_dimensions`** attribute must not
be present on any variables that do not have aggregated data. The
value of the **`aggregated_dimensions`** attribute is a blank
separated list of the aggregated dimension names given in the order
which matches the dimensions of the aggregated data. If the aggregated
data is scalar then the value of the **`aggregated_dimensions`**
attribute must be an empty string.

The dimensions listed by the **`aggregated_dimensions`** attribute
constrain the dimensions that may be spanned by variables referenced
from any of the other attributes, in the same way that the array
dimensions perform that role for a non-aggregated variable. For
instance, all variables named by the **`cell_measures`** attribute of
an aggregated data variable must span a subset of zero or more of the
dimensions given by the **`aggregated_dimensions`** attribute; or the
variable named by the **`bounds`** attribute of an aggregated
coordinate variable must span all of the aggregated dimensions in the
same order, as well as the trailing bounds dimension. Any coordinate
variable that shares its name with an aggregated dimension of an
aggregated data variable will be considered as part of the data
variable's domain definition.

The fragments are organised into an orthogonal multidimensional array
with the same number of dimensions as the aggregated data. Each
dimension of this array is called a *fragment dimension*, and
corresponds to the aggregated dimension with the same relative
position in the aggregated data. The size of a fragment dimension is
the number of fragments that span its corresponding aggregated
dimension. For instance, if an aggregated dimension of size 100 has
been fragmented into three fragments spanning 20 values each and one
fragment spanning 40 values, then the corresponding fragment dimension
will have size 4; an aggregated dimension of any size may be
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

The value of a `term` token identifying an aggregation instruction
may be standardized or non-standardized, with the understanding that
application programs should ignore terms that they do not recognise or
which are irrelevant for their purposes. The standardized aggregation
instruction terms, all of which are mandatory, are:

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

* A fragment stored in the parent file is represented by a missing
  value in conjunction with a non-missing value in the corresponding
  location of the `address` variable. If there is a trailing dimension
  then all of that dimension must comprise missing values.
  
* A fragment that is assumed to contain wholly missing values, and so
  has no external files nor exists in the parent file, is indicated by
  a missing value in conjunction with a missing value in the
  corresponding location of the `address` variable. If there is a
  trailing dimension then all of that dimension must comprise missing
  values.
  
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
  
* For a fragment stored in an external file, an addresses must be
  provided that correspond to each named file in the `file`
  variable. If there is a trailing dimension then it must be padded
  with missing values.

* For a fragment that is stored in the parent file exactly one address
  must be provided, and if there is a trailing dimension then the
  address must be stored in its first element and the trailing
  dimension must be padded with missing values.

* A fragment that is assumed to contain wholly missing values, and so
  has no external files nor exists in the parent file, is indicated by
  a missing value. If there is a trailing dimension then all of that
  dimension must comprise missing values.
  
* If the fragment is a variable in the parent file then the address is
  that variable's name, otherwise addresses are dependent on the
  format of the fragment's external file.

* For an external netCDF file, the address is the name of the variable
  that contains the fragment.

* Addressing for other file formats is allowed, but not described in
  these conventions.

### Example 1

An aggregated data variable whose aggregated data comprises two
fragments. Each fragment spans half of the aggregated `time` dimension
and the whole of the other three aggregated dimensions, and is stored
in an external netCDF file in a variable call `temp`. As all of the
external files are netCDF files, the `format` term of the
**`aggregated_data`** attribute is not required.

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
      aggregation_address = "temp", "temp" ;


## Fragment Storage

Each fragment has a generic form for which:

* The fragment's data provides the values for a unique part of the
  aggregated data, as defined by the `location` term of the
  aggregation instructions.

* The entirety of the fragment's data contributes to the aggregated
  data.

* The fragment's data has the same number of dimensions in the same
  order as the aggregated data.

* The fragment's data has the same data type as the aggregated data.

* Each dimension of the fragment's data has the same sense of
  directionality (i.e. the sense in which it increasing in physical
  space) as its corresponding aggregated dimension.

* The fragment has the same canonical physical units as the aggregated
  variable, and if the fragment has no defined units then its data is
  assumed to have the same units as the aggregated variable.

* Any other fragment attributes, as well as all associated metadata
  variables associated with the fragment (such as coordinate
  variables), are ignored by the aggregated variable.

In limited circumstances, however, a fragment may deviate from these
requirements providing that it is possible to unambiguously convert
the fragment to its generic form prior to it being used within the
aggregated data. This manipulation of the fragment is carried out by
the application program that is managing the aggregation.

The following fragment manipulations are allowed:

### Units and calendar
 
When a fragment has units that are equivalent, but not equal, to the
units of the aggregated variable, then the fragment's units must be
changed to the aggregated variable's units. This is done by applying
the appropriate multiplicative scale factor and/or additive offset to
the fragment's data. For instance, if the aggregated variable units
are degrees Fahrenheit and a fragment has units of degrees Celsius,
then the fragment's units are changed to degrees Fahrenheit by
multiplying the fragment's data by 1.8 and then adding 32.

For reference time units, the calendar of the aggregated variable and
the calendars of the fragments must also be equivalent.

For instance, if the aggregated variable units are `"days since
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

### Missing values

A fragment may use any valid means for defining missing
values. Missing values must be changed to values that the aggregated
variable recognises as missing. It is up to the creator of the dataset
to ensure that non-missing values in a fragment are not registered as
missing in the aggregated data.

### Packing

A fragment may be packed. The fragment's data must be unpacked prior
to insertion into the aggregated data.

### Compression

A fragment may be compressed. The fragment's
data must be uncompressed prior to insertion into the aggregated
data. In this case, the `location` term of the aggregation
instructions describes the shape of the uncompressed fragment.

### Example 2

An aggregated data variable whose aggregated data comprises two
fragments. Each fragment spans half of the aggregated `time` dimension
and the whole of the other three aggregated dimensions. One fragment
is stored in an external file and the other is stored the parent file
as variable `temp2`. As all of the external files are netCDF files,
the `format` term of the **`aggregated_data`** attribute is not
required. The fragment stored in the parent file has different but
equivalent units to the aggregated variable, and omits the size 1
`level` dimension.

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
      aggregation_address = "temp", "temp2" ;
      temp2 = 4.5, 3.0, 0.0, -2.6, -5.6, -10.2, ... ;


### Example 3

An aggregated data variable whose aggregated data comprises two
fragments. Each fragment is stored in the parent file and spans half
of the aggregated `time` dimension and the whole of the `latitude` and
`longitude` dimensions, but does not span the size 1 `level`
dimension. As there are no external files, the `file` and `format`
terms of the **`aggregated_data`** attribute are not required. The
fragments and aggregation definition variables in this case are stored
in a child group called `aggregation`. The `temp2` fragment has
different but equivalent units to the aggregated variable.

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
       address = "temp1", "temp2" ;
       temp1 = 270.3, 272.5, 274.1, 278.5, 280.3, 283.6, ... ;
       temp2 = 4.5, 3.0, 0.0, -2.6, -5.6, -10.2, ... ;


### Example 4

An aggregated data variable whose aggregated data comprises four
fragments. Each fragment spans half of the aggregated `time`
dimension, either the northern or southern hemisphere, and the whole
of the other two aggregated dimensions. The fragments are stored in
external netCDF files. As all of the external files are netCDF files,
the `format` term of the **`aggregated_data`** attribute is not
required. The aggregation definition variables are stored in a child
group called `aggregation`. One of the fragments has been defined by
two different external resources (one "local" and one "remote"), each
of which is provided with its own address within its file (`temp3` and
`t3` respectively). Either of these resources, but not both, may be
used in the aggregated data.

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
       address = "temp1", _,
                 "temp2", _,
                 "temp3", "t3",
                 "temp4", _ ;
       temp2 = 4.5, 3.0, 0.0, -2.6, -5.6, -10.2, ... ;

### Example 5

An aggregated data variable and an aggregated coordinate variable in
the same parent file. There are two external netCDF files, each of
which contains a fragment for each aggregated variable. The
aggregation definition variables for each aggregated variable are
stored in different groups (`aggregation_temp` and
`aggregation_time`), but the `file` terms of the **`aggregated_data`**
attributes refer to a variable in the root group that stores the
external file names that apply to both aggregation variables.

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
                                address: /aggregation_temp/address" ;
      // Coordinate variables
      double time ;
        time:standard_name = "time" ;
        time:units = "days since 2001-01-01" ;
        temp:aggregated_dimensions = "time" ;
        temp:aggregated_data = "location: /aggregation_time/location
                                file: aggregation_file	
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

    group: aggregation_time {
      variables:
        // Aggregation definition variables
        int location(f_time, f_level, f_latitude, f_longitude, i, j) ;
        string address(f_time, f_level, f_latitude, f_longitude) ;

      data:    	   
        location = 0, 5,
                   6, 11 ;
        address = "time", "time" ;
        

## Glossary

**aggregated data**

The data of an *aggregated variable* that exists as a set of
instructions on how to build an array from one or more other arrays
stored elsewhere.

**aggregated variable**

A netCDF variable that does not contain its own data, rather it
contains instructions on how to create its data as an aggregation of
data from other sources.

**fragment**

An independent, possibly self-describing, array that defines a
contiguous part of the *aggregated data*. The aggregated data is
composed from a multi-dimensional orthogonal array of fragments.

**fragment dimension**

A dimension of the multi-dimensional orthogonal array of fragments
that defines the *aggregated data*.

**parent file**

The netCDF file that contains the *aggregated variable*, and may also
contain some or all of the *fragments*.

