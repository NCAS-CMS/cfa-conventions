# NetCDF Climate and Forecast Aggregation (CFA) Conventions

### [The CFA-netCDF conventions document](https://github.com/NCAS-CMS/cfa-conventions/blob/main/source/cfa.md)

### Introduction

The CFA (Climate and Forecast Aggregation) conventions describe how a
netCDF file can be used to describe a dataset distributed across
multiple other data files. A CFA-compliant aggregation can be
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
the CF (Climate and Forecast) conventions that specify the geophysical
meaning of all variables in the file, whether their data are defined
as aggregations or not. The CFA conventions do not duplicate, extend,
nor re-define any of the metadata elements defined by the CF
conventions. However, when CF-compliant software is used for reading
the discovery metadata of a CFA-netCDF file, with no expectation of
reading the data of aggregated variables, a small extension is needed
to allow the correct interpretation of the dimensionality of
aggregation variables.

