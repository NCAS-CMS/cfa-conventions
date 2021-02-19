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
fragment is contained in an external file, or may be another contained
in the same file as the aggregated variable (i.e the *parent file*).

An aggregated variable should be a scalar (i.e. it has no dimensions)
and the value of its single element is immaterial. It acts as a
container for the usual attributes that define the data (such as
`standard_name` and `units`), with the addition of special attributes
that provide instructions on how to create the aggregated data.

The dimensions of the aggregated data, called the *aggregated
dimensions* must exist as dimensions in the parent file and must be
stored with the `aggregated_dimensions` attribute, and the presence of
an `aggregated_dimensions` attribute will identify an aggregated
variable. Therefore the `aggregated_data` attribute must not be
present on any variables that do not have aggregated data. The value
of the dimensions attribute is a blank separated list of the
aggregated dimension names given in the order which matches the
dimensions of the aggregated data. If the aggregated data is scalar
then the value of the `aggregated_dimensions` attribute must be an
empty string.

If the aggregated variable is a data variable then any coordinate
variable that shares its name with an aggregation dimension given by
the `aggregated_dimensions` attribute is considered as part of the
definition of the data variable's domain.

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
aggregate them, i.e. the *aggregation definition*, are provided by the
`aggregated_data` attribute. This attribute takes a string value, the
string being comprised of blank-separated elements of the form `"term:
variable"`, where `term` is a case-insensitive keyword that identifies
a component of the aggregation defintion, and `variable` is the name
of a variable that contains the values that configure the component
for each fragment. The order of elements is not significant.

Each variable referenced by the `aggregated_data` attribute must span
the fragment dimensions in the same relative order as the aggregated
dimensions, so that its values are easily associated with both
individual fragments and the aggregated data. If an instruction
requires multiple values to be provided per fragment then a variable
may also have one or more extra trailing dimensions.

The terms defining the aggregation instructions are defined in
Appendix A. Additional terms are allowed with the understanding that
application programs should ignore terms that they do not recognise or
which are irrelevant for their purposes.

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
      float temp ;
        temp:standard_name = "air_temperature" ;
        temp:units = "K" ;
        temp:aggregated_dimensions = "time level latitude longitude" ;
        temp:aggregated_data = "index: aggregation_index 
                                location: aggregation_location
                                file: aggregation_file
                                address: aggregation_address" ;
      // Aggregation definition variables			 	  
      int aggregation_index(p_time, f_level, f_latitude, f_longitude, i) ;
      int aggregation_location(f_time, f_level, f_latitude, f_longitude, i, j) ;
      string aggregation_file(f_time, f_level, f_latitude, f_longitude) ;
      string aggregation_address(f_time, f_level, f_latitude, f_longitude) ;
      // Coordinate variables
      float time(time) ;
        time:standard_name = "time" ;
        time:units = "days since 2000-01-01" ;
      float level(level) ;
        level:standard_name = "height_above_mean_sea_level" ;
        level:units = "m" ;
      float latitude(latitude) ;
        latitude:standard_name = "latitude" ;
        latitude:units = "degrees_north" ;
      float longitude(longitude) ;
        longitude:standard_name = "longitude" ;
        longitude:units = "degrees_east" ;
    data:
      temp = _ ;
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

Each fragment provides the values for a unique part of the aggregated
data and has a generic form for which

* each dimension of the fragment's data corresponds to a unique
  aggregated dimension;

* the fragment's data dimensions have the same relative order as the
  aggregated dimensions in the aggregated data;

* each dimension of the fragment's data has the same sense of
  directionality (i.e. the sense in which it increasing) as its
  corresponding aggregated dimension;

* the entirety of the fragment's data contributes to the aggregated
  data;

* the fragment has the same physical units as the aggregated variable.

In limited circumstances, however, a fragment may deviate from these
requirements, providing that it is possible to unambiguously convert
the fragment to its generic form prior to it being used within the
aggregated data. This manipulation of the fragment is carried out by
the application program that is managing the aggregation.

Note that the only fragment metadata that are taken into consideration
in the aggregation process are the units and the calendar. In
particular, metadata that defines the physical nature of the data
(such as the `standard_name` attribute) and metadata that provides
coordinate values for the data (such as association coordinate
variables) are ignored, if present.

The conventions support the following fragment manipulations:

### Units and calendar

A fragment may have any units that are equivalent to the units of the
aggregated variable. The fragment's units must be changed to the
aggregated variable's units, unless they are already equal, by
applying the appropriate additive offset and/or multiplicative scale
factor to the fragment's data. For reference time units, the calendars
of the aggregated variable and a fragment must also be equivalent. For
fragments contained in the parent file or in external netCDF files,
the units attribute must be defined for data that represent
dimensional quantities. Specification of the units and calendar for
fragments stored in other file formats is not described in these
conventions.

For instance, if the aggregated variable units are `"days since
2001-01-01"` in the Gregorian calendar, and a fragment has units of
`"days since 2002-01-1"` in the same calendar then the reference time
of the fragment's units are changed to the earlier date by subtracting
365 from the fragment's data.

### Missing dimensions

If a fragment has fewer dimensions than the aggregated data then the
missing dimensions must be inserted into the fragment's data as size 1
dimensions.

For instance, in Example 1 each fragment's data in the external files
could have shape `(6, 73, 144)`, instead of the shape `(6, 1, 73,
144)` required by the generic form.

### Example 2

An aggregated data variable whose aggregated data comprises two
fragments. Each fragment spans half of the aggregated `time` dimension
and the whole of the other three aggregated dimensions. One fragment
is stored in an external file and the other is stored the parent file
as variable `temp2`. As all of the external files are netCDF files,
the `format` term of the `aggregated_data` attribute is not
required. The fragment stored in the parent file has different but
equivalent units, and omits the size 1 `level` dimension.

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
      float temp ;
        temp:standard_name = "air_temperature" ;
        temp:units = "K" ;
        temp:aggregated_dimensions = "time level latitude longitude" ;
        temp:aggregated_data = "index: aggregation_index 
                                location: aggregation_location
                                file: aggregation_file
                                address: aggregation_address" ;
      // Aggregation definition variables			 	  
      int aggregation_index(p_time, f_level, f_latitude, f_longitude, i) ;
      int aggregation_location(f_time, f_level, f_latitude, f_longitude, i, j) ;
      string aggregation_file(f_time, f_level, f_latitude, f_longitude) ;
      string aggregation_address(f_time, f_level, f_latitude, f_longitude) ;
      // Fragment variable
      float temp2(time, latitude, longitude) ;
        temp:units = "degreesC" ;
      // Coordinate variables
      float time(time) ;
        time:standard_name = "time" ;
        time:units = "days since 2000-01-01" ;
      float level(level) ;
        level:standard_name = "height_above_mean_sea_level" ;
        level:units = "m" ;
      float latitude(latitude) ;
        latitude:standard_name = "latitude" ;
        latitude:units = "degrees_north" ;
      float longitude(longitude) ;
        longitude:standard_name = "longitude" ;
        longitude:units = "degrees_east" ;
    data:
      temp = _ ;
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
      temp2 = 4.5, 3.0, 0,0, -2.6, -5.6, -10.2, ... ;


### Example 3

An aggregated data variable whose aggregated data comprises two
fragments. Each fragment spans half of the aggregated `time` dimension
and the whole of the other three aggregated dimensions, and is stored
in the parent file. As there are no external files, the `file` and
`format` terms of the `aggregated_data` attribute are not
required. The fragments and aggregation defintion variables in this
case are stored in a child group called `aggregation`.

    dimensions:
      // Aggregated dimensions
      time = 12 ;
      level = 1 ;
      latitude = 73 ;
      longitude = 144 ;
     variables:
      // Data variable
      float temp ;
        temp:standard_name = "air_temperature" ;
        temp:units = "K" ;
        temp:aggregated_dimensions = "time level latitude longitude" ;
        temp:aggregated_data = "index: /aggregation/index 
                                location: /aggregation/location
                                address: /aggregation/address" ;
      // Coordinate variables
      float time(time) ;
        time:standard_name = "time" ;
        time:units = "days since 2000-01-01" ;
      float level(level) ;
        level:standard_name = "height_above_mean_sea_level" ;
        level:units = "m" ;
      float latitude(latitude) ;
        latitude:standard_name = "latitude" ;
        latitude:units = "degrees_north" ;
      float longitude(longitude) ;
        longitude:standard_name = "longitude" ;
        longitude:units = "degrees_east" ;
    data:
      temp = _ ;

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
        float temp1(time, latitude, longitude) ;
          temp:units = "Kelvin" ;
        float temp2(time, latitude, longitude) ;
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
       temp2 = 4.5, 3.0, 0,0, -2.6, -5.6, -10.2, ... ;


### Example 4

An aggregated data variable whose aggregated data comprises four
fragments. Each fragment spans half of the aggregated `time`
dimension, either the northern or southern, and the whole of the other
two aggregated dimensions. The fragments are stored in external netCDF
files. As all of the external files are netCDF files, the `format`
term of the `aggregated_data` attribute is not required. The
aggregation defintion variables are stored in a child group called
`aggregation`. One of the fragments has been defined by two different
external resources (one "local" and one "remote"), each of which is
provided with its own address within its file (`temp3` and `t3`
respectively). Either of resources, but not both, may be used in the
aggregated data.

    dimensions:
      // Aggregated dimensions
      time = 12 ;
      level = 1 ;
      latitude = 73 ;
      longitude = 144 ;
    variables:
      // Data variable
      float temp ;
        temp:standard_name = "air_temperature" ;
        temp:units = "K" ;
        temp:aggregated_dimensions = "time level latitude longitude" ;
        temp:aggregated_data = "index: /aggregation/index 
                                location: /aggregation/location
                                file: /aggregation/file
                                address: /aggregation/address" ;
      // Coordinate variables
      float time(time) ;
        time:standard_name = "time" ;
        time:units = "days since 2000-01-01" ;
      float level(level) ;
        level:standard_name = "height_above_mean_sea_level" ;
        level:units = "m" ;
      float latitude(latitude) ;
        latitude:standard_name = "latitude" ;
        latitude:units = "degrees_north" ;
      float longitude(longitude) ;
        longitude:standard_name = "longitude" ;
        longitude:units = "degrees_east" ;
    data:
      temp = _ ;

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
        string address(f_time, f_level, f_latitude, f_longitude) ;
        float temp2(time, latitude, longitude) ;
          temp2:long_name = "January-June, northern hemisphere" ;
          temp2:units = "degreesC" ;
	      
      data:    	   
        index = 0, 0, 0, 0,
                1, 0, 0, 0 ;
                0, 0, 1, 0,
                1, 0, 1, 0 ;
        location = 0, 5,
                   0, 0,
                   0, 35,
                   0, 143,
                   6, 11,
                   0, 0,
                   0, 35,
                   0, 143 ;
                   0, 5,
                   0, 0,
                   36, 72,
                   0, 143 ;
                   6, 11,
                   0, 0,
                   36, 72,
                   0, 143 ;
       file = "/remote/January-June_SH.nc", _,
              _, _ ;
              "/local/July-December_NH.nc", "/remote/July-December_NH.nc";
              "/remote/July-December_NH.nc", _,
       address = "temp1", _,
                 "temp2", _
                 "temp3", "t3",
                 "temp4", _ ;
       temp2 = 4.5, 3.0, 0,0, -2.6, -5.6, -10.2, ... ;


# Appendix A

Terms defining the aggregation instructions given by the
`aggregated_data` attribute.

#### `index`

* Names the integer-valued variable containing the indices of each
  fragment along the fragment dimensions. For each fragment, this
  variable stores the zero-based index of its position along each
  fragment dimension. Therefore, the variable requires one extra
  trailing dimension whose size is equal to the number of fragment
  dimensions.

#### `location`

* Names the integer-valued variable containing the index ranges of the
  aggregated data that correspond to each fragment. For each fragment
  and each aggregated dimension, this variable stores the two
  zero-based indices giving the beginning and end of the range of
  indices that defines the position of the fragment along the
  aggregated dimension. Therefore, the variable requires two extra
  trailing dimensions. The size of the first is equal to the number of
  fragment dimensions, and the second has size 2.

#### `file`

* Names the string-valued variable containing the URIs of the
  fragments. Each value identifies the external resource which
  contains the fragment. Fragments stored as variables in the parent
  file must be represented by missing values.

* The `file` term must be omitted if all fragments are stored as
  variables in the parent file.

* An extra trailing dimension may included to describe multiple URIs
  for the same fragment, any one of which may equally be used to fill
  its position in the aggregated data. In this case, it is up to
  application programs to choose the version of the fragment that it
  finds the most preferable. If a fragment has fewer versions than
  others then the trailing dimension must be padded with missing
  values. When a fragment is stored in the parent file it is not
  possible to use an alternative fragment in external file, so in this
  case the trailing dimension for this fragment position must be
  completely filled with missing values.
  
#### `format`

* Names the string-valued variable containing the case-insensitive
  file formats for fragments stored in external files.

* The `format` term must be omitted if there is no `file` variable.

* If all fragments are in external netCDF files then the `format` term
  may be omitted.
	  
* A fragment stored as a variable in the parent file is represented by
  a missing value.

* A fragment in an external netCDF file is signified by a value of
  `"nc"`.

* The `format` variable must span exactly the same dimensions in the
  same order as the `file` variable. Missing values should be used
  whenever the corresponding location in the `file` variable is a
  missing value.

* Specification of other file formats is allowed, but not described in
  these conventions.

#### `address`

* Names the variable containing each fragment's address within its
  file. If the fragment is in the parent file then the address is the
  variable name, otherwise the address is dependent on the format of
  the fragment's external file. For an external netCDF file, the
  address is also the name of the variable that contains the fragment.

* If the `file` variable does not exist then no extra trailing
  dimensions nor missing values are allowed.

* If there is a `file` variable then the `adddress` variable must span
  the same dimensions in the same order. Missing values should be used
  whenever the corresponding location in the `file` variable has a
  missing value, except for fragments stored in the parent file. For a
  fragment stored in the parent file, exactly one address must be
  provided, and if there is a trailing dimension then it must be
  padded with missing values.
	       
* Addressing for other file formats is allowed, but not described in
  these conventions.

