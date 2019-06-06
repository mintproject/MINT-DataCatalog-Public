## Resources

Regardless of dataset semantics, its actual data will be physically stored a sequence of 1's and 0's on disk. On a local computer, we would refer to them as files and identify them using a file path, e.g. `/path/to/file`. More generally, the files could be located on a remote server and require different protocols (i.e. `http://` or `file://`) to access. Therefore, we chose the more proper term **resource** to refer to the physical (as opposed to semantic) container that holds the data.

A resource represents an atomic data unit of a dataset. A dataset usually has only a single resource associated with it, but can have many resources (e.g., FLDAS dataset is organized as a collection of resources going back to 1981, each containing data for one month worth of observations). 

Data catalog currently supports the following resource types: **NetCDF**, **GeoTIFF**, and **CSV**. Each type has its own set of metadata, but the following list applies to all of them:

### Mandatory metadata

##### record_id [mandatory]
`record_id` is resource's unique identifier (in the form of [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier))

##### dataset_id [mandatory]
`dataset_id` is the uuid of the dataset to which this resources belongs

##### name [mandatory]
`name` is what resource is called. Typically, it's the name at the end of the URI path. For example, for a resource located at `http://www.example.com/path/to/my/data_file.txt`, its name would be `data_file`

##### data_url [mandatory]
`data_url` is the direct download link for the resource

##### variable_ids [mandatory]
`variable_ids` is an array of variable ids (`uuid`s) 

##### resource type [mandatory]

`resource_type` describes the file type of the resource
```json
{
	"resource_type": "NetCDF|GeoTIFF|CSV"
}

```

### [Highly] recommended and optional metadata


#### Arbitrary metadata
In addition to the mandatory attributes above, data catalog also allows to provide arbitrary key-value metadata. You can provide any valid JSON meaning that the value can be a string, number, array, or another object.


```json
{
	"metadata": {
		"<other_fields>": "...",
		"myCustomKey1": "myCustomValue1",
		"myCustomKey2": ["list", "of", "custom", "values"],
		"<other_fields>": "...",
	}
}
```

The `metadata` object is where you should put other recommended metadata in order to take full advantage of data catalog's capabilities. Description of these recommended fields is provided below.

#### Recommended metadata

##### temporal coverage [recommended]
`temporal_coverage` provides the length and granularity information on temporal coverage of the resource. Dates follow [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format `YYYY-MM-DDThh:mm:ss`. Providing this information makes it possible to find the data using temporal queries (e.g., is there rainfall data for 2015-2017 years?)

```json
{
	"metadata": {
		"temporal_coverage": {
			"start_date": "1980-01-01T00:00:00",
			"end_date": "1999-12-31T23:59:59",
			"resolution": {
				"value": 3,
				"units": "year|month|week|day|hour|minute|second"
			}

		}
	}
}
```

##### spatial coverage [recommended]
`spatial_coverage` provides information about the spatial coverage of the resource (in WGS84 coordinate system). Here, `x` refers to longitude and `y` refers to latitude (in degrees). Providing this information makes it possible to find the data using geospatial queries (e.g., what kind of rainfall-related datasets are available for South Sudan?)


```json
{
	"metadata": {
		"spatial_coverage": {
			"type": "BoundingBox",
		    "value": {
		        "xmin": 33.9605286,
		        "ymin": -118.4253354,
		        "xmax": 33.9895077,
		        "ymax": -118.4093589
		    }

		}
	}
}

```

##### resource size [recommended]
`size` - number of bytes

```json
{
	"metadata": {
		"size": 12345
	}
}
```

##### resource created at date [recommended]
`date_created` is a timestamp of when this particular resource was created (as opposed to registered in the data catalog)
```json
{
	"metadata": {
		"date_created": "2000-01-01T01:23:45"
	}
}
```


### Resource type-specific metadata


#### NetCDF
##### dimensions [recommended]
`dimensions` is JSON object describing the sizes of each dimension. NetCDF should already contain that information and you can view it using e.g. `ncdump`: `ncdump -h my_netcdf_file.nc` and the output will look something like 
```
dimensions:
	time = UNLIMITED ; // (1 currently)
	Y = 348 ;
	X = 294 ;
	bnds = 2 ;
```
In this case, `dimensions` object for data catalog would look like
```json

{
	"metadata": {
		"dimensions": {
			"time": "UNLIMITED ; (1 currently)",
			"Y": 348,
			"X": 294,
			"bnds": 2
		}
	}
}
```
We treat any non-integer value as "unlimited"


##### geospatial metadata [recommended]
`spatial_coverage` asks for the bounding box that describes the spatial extent (in WGS84 coordinates) that the data within the resource covers. But the actual data can be stored in a different spatial reference system (SRS). In order to visualize the contents of the resource on a map, we need to know the SRS, which dimensions map to latitude and longitude, and the spatial resolution of cell/pixel. So, for the example above with dimensions `"time"`, `"Y"`, `"X"`, and `"bnds"`, geospatial metadata might look something like this:

```json
{
	"metadata": {
		"geospatial_metadata": {
			"srs": {
				"srid": "EPSG:4326"
			},
			"resolution": {
				"latitude": {
					"dimension": "X",
					"value": 10,
					"units": "m"
				},
				"longitude": {
					"dimension": "Y",
					"value": 10,
					"units": "m"
				}
			}
		}
	}
}
```

#### GeoTIFF
##### dimensions [recommended]
`dimensions` is JSON object describing the sizes of each dimension. Typically, a GeoTIFF is an image with a certain width, height, and the number of bands.
```json
{
	"metadata": {
		"dimensions": {
			"height": 123,
			"width": 456,
			"bands": 3
		}
	}
}

```

`spatial_coverage` asks for the bounding box that describes the spatial extent (in WGS84 coordinates) that the data within the resource covers. But the actual data can be stored in a different spatial reference system (SRS). In order to visualize the contents of the resource on a map, we need to know the SRS, which dimensions map to latitude and longitude, and the spatial resolution of cell/pixel. So, for the example above with dimensions `"height"`, `"width"`, and `"bands"`, geospatial metadata might look something like this:

```json
{
	"metadata": {
		"geospatial_metadata": {
			"srs": {
				"srid": "EPSG:4326"
			},
			"resolution": {
				"latitude": {
					"dimension": "width",
					"value": 10,
					"units": "m"
				},
				"longitude": {
					"dimension": "height",
					"value": 10,
					"units": "m"
				}
			}
		}
	}
}
```


### CSV
##### dimensions [recommended]
`dimensions` is a JSON object describing the sizes of each dimension. Typically, a CSV consists of two dimensions: rows and columns
```json
{
	"metadata": {
		"dimensions": {
			"rows": 100,
			"cols": 10
		}
	}
}
```

##### delimiter [recommended]
`delimiter` describes how fields are separated
```json
{
	"metadata": {
		"delimiter": ","
	}
}
```

##### has header [recommended]
`has_header` is a boolean flag that should be `true` if the first row of the CSV file is a header and `false` otherwise
```json

{
	"metadata": {
		"has_header": true
	}
}
```