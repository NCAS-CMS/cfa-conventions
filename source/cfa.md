# Aggregated Variables

*v0.1 - v0.4 David Hassell and Jonathan Gregory, 2012 - 2019*

*v0.6b1 David Hassell, Jonathan Gregory, Neil Massey, and Bryan
 Lawrence 2021-02-18*

An *aggregated variable* does not contain its own data, rather it
contains instructions on how to create its data as an aggregation of
data from other sources. When created by an application program, the
data of an aggregated variable is called its *aggregated data*. The
aggregated data is composed of one or more *fragments*, each of which
provides the values for a unique part of the aggregated data. Each
fragment is contained in either an external file or else is described
by another variable contained in the same file as the aggregated
variable (i.e the *parent file*).

An aggregated variable should be a scalar (i.e. it has no dimensions)
and the value of its single element is immaterial. It acts as a
container for the usual attributes that define the data (such as
`standard_name` and `units`), with the addition of special attributes
that provide instructions on how to create the aggregated data.

The dimensions of the aggregated data, called the *aggregated
dimensions* must exist as dimensions in the parent file and must be
stored with the `aggregated_dimensions` attribute, and the presence of
an `aggregated_dimensions` attribute will identify an aggregated
variable. Therefore the `aggregated_dimensions` attribute must not be
present on any variables that do not have aggregated data. The value
of the `aggregated_dimensions` attribute is a blank separated list of
the aggregated dimension names given in the order which matches the
dimensions of the aggregated data. If the aggregated data is scalar
then the value of the `aggregated_dimensions` attribute must be an
empty string.

The dimensions listed by the `aggregated_dimensions` attribute
constrain the dimensions that may be spanned by variables referenced
from any of the other attributes, in the same way that the array
dimensions perform that role for a non-aggregated variable. For
instance, all variables named by the `cell_measures` attribute of an
aggregated data variable must span a subset of zero or more of the
dimensions given by the `aggregated_dimensions` attribute; or the
variable named by the `bounds` attribute of an aggregated coordinate
variable must span all of the aggregated dimensions in the same order,
as well as the trailing bounds dimension. Any coordinate variable that
shares its name with an aggregated dimension of an aggregated data
variable will be considered as part of the data variable's domain
definition.

The fragments are organised into an orthogonal multidimensional array
with the same number of dimensions as the aggregated data. Therefore
each aggregated dimension has a corresponding *fragment dimension*
whose size is the number of fragments along an aggregated dimension,
and this size must be no greater than the aggregated dimension
size. For instance, if an aggregated dimension of size 100 has been
fragmented into three fragments spanning 20 values each and one
fragment spanning 40 values, then the corresponding fragment dimension
will have size 4. An aggregated dimension of any size may be
associated with a fragment dimension of size 1.

The definitions of the fragments and the instructions on how to
aggregate them are provided by the `aggregated_data` attribute. This
attribute takes a string value, the string being comprised of
blank-separated elements of the form `"term: variable"`, where `term`
is a case-insensitive keyword that identifies a particular aggregation
instruction, and `variable` is the name of a variable that configures
that instruction for each fragment. The order of elements is not
significant.

A variable referenced by the `aggregated_data` attribute must span
only the fragment dimensions in the same relative order as the
aggregated dimensions, with the possibility of extra trailing
dimensions if these are allowed by the aggregation instruction.

The terms of the aggregation instructions, with descriptions of their
variables, are as follows:

`index`

* For each fragment, identifies its position in the orthogonal
  multidimensional array of fragments.

* Names the integer-valued variable containing the indices of each
  fragment along the fragment dimensions. For each fragment, this
  variable stores the zero-based index of its position along each
  fragment dimension. Therefore, the variable requires one extra
  trailing dimension whose size is equal to the number of fragment
  dimensions.

`location`

* For each fragment, identifies the part of the aggregated data for
  which the fragment provides values.

* Names the integer-valued variable containing the index ranges of the
  aggregated dimensions that correspond to each fragment. For each
  fragment and each aggregated dimension, this variable stores the two
  zero-based indices for the beginning and end of the range of indices
  that defines the position of the fragment along the aggregated
  dimension. Therefore, the variable requires two extra trailing
  dimensions. The size of the first is equal to the number of fragment
  dimensions, and the second has size 2.

`file`

* For each fragment, identifies the file in which it is stored.

* Names the string-valued variable containing the URIs of the files
  containing the fragments. Each value identifies the external
  resource which contains the fragment.

* Fragments stored in the parent file must be represented by missing
  values.

* The `file` term may be omitted if all fragments are stored as
  variables in the parent file.

* An extra trailing dimension may be included to describe multiple
  URIs for the same fragment, any one of which may equally be used to
  fill its position in the aggregated data. In this case, it is up to
  application programs to choose the version of the fragment that it
  finds most preferable. If a fragment has fewer versions than others
  then the trailing dimension must be padded with missing values. When
  a fragment is stored in the parent file alternative fragments in
  external files are not allowed, so the extra trailing dimension for
  such a fragment must always be completely filled with missing
  values.
  
`format`

* For each fragment, identifies the format of the file in which it is
  stored.

* Names the string-valued variable containing the case-insensitive
  file formats for fragments stored in external files.

* The `format` term must be omitted if there is no `file` term.

* If all fragments are stored in the parent file or in external netCDF
  files then the `format` term may be omitted.
	  
* A fragment stored in the parent file is represented by a missing
  value.
  
* A fragment in an external netCDF file is signified by a value of
  `"nc"`.

* The `format` variable must span exactly the same dimensions in the
  same order as the `file` variable. Missing values should be used
  whenever the corresponding location in the `file` variable is a
  missing value.

* Specification of other file formats is allowed, but not described in
  these conventions.

`address`

* For each fragment, identifies the address of the fragment within its
  file.

* Names the variable containing each fragment's address within its
  file. If the fragment is in the parent file then the address is the
  variable name, otherwise the address is dependent on the format of
  the fragment's external file. For an external netCDF file, the
  address is also the name of the variable that contains the fragment.

* If there is a `file` variable then the `address` variable must span
  exactly the same dimensions in the same order. Missing values should
  be used whenever the corresponding location in the `file` variable
  has a missing value, except for fragments stored in the parent
  file. For a fragment stored in the parent file, exactly one address
  must be provided, and if there is a trailing dimension then it must
  be padded with missing values.
	  
* If there is no `file` term then all fragments must be stored in the
  parent file, and so no extra trailing dimensions nor missing values
  are allowed.
     
* Addressing for other file formats is allowed, but not described in
  these conventions.

Additional terms are allowed with the understanding that application
programs should ignore terms that they do not recognise or which are
irrelevant for their purposes.

### Example 1

An aggregated data variable whose aggregated data comprises two
fragments. Each fragment spans half of the aggregated `time` dimension
and the whole of the other three aggregated dimensions, and is stored
in an external netCDF file in a variable call `temp`. As all of the
external files are netCDF files, the `format` term of the
`aggregated_data` attribute is not required.

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
        temp:aggregated_data = "index: aggregation_index 
                                location: aggregation_location
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
      int aggregation_index(p_time, f_level, f_latitude, f_longitude, i) ;
      int aggregation_location(f_time, f_level, f_latitude, f_longitude, i, j) ;
      string aggregation_file(f_time, f_level, f_latitude, f_longitude) ;
      string aggregation_address(f_time, f_level, f_latitude, f_longitude) ;
    data:
      temp = _ ;
      time = 0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334 ;
      aggregation_index = 0, 0, 0, 0,
                          1, 0, 0, 0 ;
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

Each fragment has a generic form for which

* the fragment's data provides the values for a unique part of the
  aggregated data, as defined by the `location` term of the
  aggregation instructions,

* the entirety of the fragment's data contributes to the aggregated
  data,

* the fragment's data has the same number of dimensions in the same
  order as the aggregated data,

* each dimension of the fragment's data has the same sense of
  directionality (i.e. the sense in which it increasing in physical
  space) as its corresponding aggregated dimension,

* the fragment has the same physical units as the aggregated variable,

* any other fragment attributes, as well as all associated metadata
  variables associated with the fragment (such as coordinate
  variables), are ignored.

For fragments contained in the parent file or in external netCDF
files, the units must be defined for data that represent dimensional
quantities. Specification of the units for fragments stored in other
file formats is not described in these conventions.

In limited circumstances, however, a fragment may deviate from these
requirements, providing that it is possible to unambiguously convert
the fragment to its generic form prior to it being used within the
aggregated data. This manipulation of the fragment is carried out by
the application program that is managing the aggregation.

The following fragment manipulations are allowed:

### Units and calendar

A fragment may have any units that are equivalent, but not necessarily
equal, to the units of the aggregated variable. The fragment's units
will be changed to the aggregated variable's units, if required, by
applying the appropriate multiplicative scale factor and/or additive
offset to the fragment's data.

For instance, if the aggregated variable units are degrees Fahrenheit
and a fragment has units of degrees Celsius, then the fragment's units
are changed to degrees Fahrenheit by multiplying the fragment's data
by 1.8 and then adding 32.

For reference time units, the calendars of the aggregated variable and
a fragment must also be equivalent.

For instance, if the aggregated variable units are `"days since
2001-01-01"` in the Gregorian calendar and a fragment has units of
`"days since 2002-01-1"` in the same calendar, then the reference time
of the fragment's units are changed to the earlier date by subtracting
365 from the fragment's data.

### Missing size 1 dimensions

A fragment may omit any size 1 dimension for which the size of the
fragment's location along the corresponding aggregated dimension is
also size 1. Any missing size 1 dimensions will be inserted into the
fragment's data in the appropriate positions.

For instance, if a fragment's shape given by the `location` term of
the aggregation instructions is `(6, 1, 73, 144)`, then the
fragment's data could have shape `(6, 1, 73, 144)` or `(6, 73, 144)`.


### Example 2

An aggregated data variable whose aggregated data comprises two
fragments. Each fragment spans half of the aggregated `time` dimension
and the whole of the other three aggregated dimensions. One fragment
is stored in an external file and the other is stored the parent file
as variable `temp2`. As all of the external files are netCDF files,
the `format` term of the `aggregated_data` attribute is not
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
        temp:aggregated_data = "index: aggregation_index 
                                location: aggregation_location
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
      int aggregation_index(p_time, f_level, f_latitude, f_longitude, i) ;
      int aggregation_location(f_time, f_level, f_latitude, f_longitude, i, j) ;
      string aggregation_file(f_time, f_level, f_latitude, f_longitude) ;
      string aggregation_address(f_time, f_level, f_latitude, f_longitude) ;
      // Fragment variable
      double temp2(time, latitude, longitude) ;
        temp:units = "degreesC" ;
	
    data:
      temp = _ ;
      time = 0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334 ;
      aggregation_index = 0, 0, 0, 0,
                          1, 0, 0, 0 ;
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
fragments. Each fragment spans half of the aggregated `time` dimension
and the whole of the other three aggregated dimensions, and is stored
in the parent file. As there are no external files, the `file` and
`format` terms of the `aggregated_data` attribute are not
required. The fragments and aggregation definition variables in this
case are stored in a child group called `aggregation`.

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
        temp:aggregated_data = "index: /aggregation/index 
                                location: /aggregation/location
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
        int index(f_time, f_level, f_latitude, f_longitude, i) ;
        int location(f_time, f_level, f_latitude, f_longitude, i, j) ;
        string address(f_time, f_level, f_latitude, f_longitude) ;
        // Fragment variables
        double temp1(time, latitude, longitude) ;
          temp:units = "Kelvin" ;
        double temp2(time, latitude, longitude) ;
          temp2:units = "degreesC" ;

      data:    	   
        index = 0, 0, 0, 0,
                1, 0, 0, 0 ;
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
the `format` term of the `aggregated_data` attribute is not
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
        temp:aggregated_data = "index: /aggregation/index 
                                location: /aggregation/location
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
        int index(f_time, f_level, f_latitude, f_longitude, i) ;
        int location(f_time, f_level, f_latitude, f_longitude, i, j) ;
        string file(f_time, f_level, f_latitude, f_longitude, k) ;
        string address(f_time, f_level, f_latitude, f_longitude, k) ;
        // Fragment variable
        double temp2(time, latitude, longitude) ;
          temp2:long_name = "July-December, southern hemisphere" ;
          temp2:units = "degreesC" ;
	      
      data:    	   
        index = 0, 0, 0, 0,
                1, 0, 0, 0,
                0, 0, 1, 0,
                1, 0, 1, 0 ;
        location = 0, 5,
                   0, 0,
                   0, 35,
                   0, 143,
                   6, 11,
                   0, 0,
                   0, 35,
                   0, 143,
                   0, 5,
                   0, 0,
                   36, 72,
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
`aggregation_time`), but the `file` terms of the `aggregated_data`
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
        temp:aggregated_data = "index: /aggregation_temp/index 
                                location: /aggregation_temp/location
                                file: aggregation_file	
                                address: /aggregation_temp/address" ;
      // Coordinate variables
      double time ;
        time:standard_name = "time" ;
        time:units = "days since 2001-01-01" ;
        temp:aggregated_dimensions = "time" ;
        temp:aggregated_data = "index: /aggregation_time/index 
                                location: /aggregation_time/location
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
        int index(f_time, f_level, f_latitude, f_longitude, i) ;
        int location(f_time, f_level, f_latitude, f_longitude, i, j) ;
        string address(f_time, f_level, f_latitude, f_longitude) ;

      data:    	   
        index = 0, 0, 0, 0,
                1, 0, 0, 0 ;
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
        int index(f_time, f_level, f_latitude, f_longitude, i) ;
        int location(f_time, f_level, f_latitude, f_longitude, i, j) ;
        string address(f_time, f_level, f_latitude, f_longitude) ;

      data:    	   
        index = 0,
                1,
        location = 0, 5,
                   6, 11 ;
        address = "time", "time" ;


