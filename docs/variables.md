

## Variables

Every dataset contains information about one or more features of interest (e.g., average household size, watershed boundaries, monthly precipitation rates, etc). The information usually comes in the form of varying values that these features can take on, and hence are referred to as *variables*. Because the variables contain information that makes a dataset "interesting", we tend to use variables' names/descriptions to describe the contents of the dataset semantically (i.e. what is the dataset _about_?). 

Unfortunately, using natural language to describe the dataset contents makes it difficult to compare and share different datasets. MINT Data Catalog tackles this problem in two ways. First, we allow variables to be associated with [standard variables](), making it easier to discover and use standardized vocabulary to describe variable's semantics. Second, we capture quantitative descriptions of variables (e.g., units of measure, handling of missing values, etc) using a small set of metadata attributes. 

The number of possible variables is infinite, but all them can be placed into two types, each with its own two subtypes: "categorical" with "nominal" and "ordinal" or "numerical" with "discrete" and "continuous". The required metadata depends on the (sub)type of a variable and is described in more detail below.


### Categorical Variables
A categorical variable is a variable that takes on a value from a list of possible values. 

#### Nominal Variables
A nominal variable is a subtype of a categorical variable where the way the values are ordered does not matter. For example, a variable “eye color” with the following list of possible values: [“amber”, “blue”, “brown”, “green”] can be considered a nominal variable because there is no semantically (as opposed to e.g., alphabetically) meaningful way to order them (e.g., “blue” > “green”). 

When registering a nominal variable, please provide the following metadata:
```json
"metadata": {
	"variable_type": "categorical.nominal",
	"values": ["a", "list", "of", "possible", "values"],
	"data_type": "string|float|integer"
}

```

#### Coded variable values
In case the variables are coded as numbers, please provide a mapping of codes to variables using the format {code: “value”}. For example:

```json
"metadata": {
	"variable_type": "categorical.nominal",
	"values": [{0: "a"}, {1:"list"}, {2:"of"}, {3:"possible"}, {4:"values"}],
	"data_type": "string|float|integer"
}
```


The purpose of the variables
#### Ordinal Variables
An ordinal variable is a subtype of a categorical variable where the values have a natural order to them (although the distances between them are not known). For example, a survey response variable that is based on Likert Scale could be one of the following: [“strongly disagree”, “disagree”, “neither agree nor disagree”, “agree”, “strongly agree”]. These values can be ordered from “worst” to “best”, hence the name “ordinal”.

When registering ordinal variables, please provide the following metadata:
```json
"metadata": {
	"variable_type": "categorical.ordinal",
	"values": ["ordered", "list", "of", "possible", "values"]
}

```

Although this metadata format is similar to nominal variable metadata, here the order of elements in the list matters.

#### Coded variable values
In case the variables are coded as numbers, please provide a mapping of codes to variables using the format {code: “value”}. For example:
```json
"metadata": {
	"variable_type": "categorical.ordinal",
	"values": [{0:"ordered"}, {1:"list"}, {2:"of"}, {3:"possible"}, {4:"values"}]
}
```

### Numerical Variables
A numerical variable is a variable that takes on numerical values.

### Discrete Variables
A discrete variable is a subtype of a numerical variable where the values take on integer values. For example, a variable “household size” can be 4.

When registering discrete variables, please provide the following metadata:
```json
"metadata": {
	"variable_type": "numerical.discrete",
	"min_value": 0,
	"max_value": e.g. 100,
	"no_data_value": -1
}

```

### Continuous Variables
A continuous variable is a subtype of a numerical variable where the values take on any value within some range. For example, a medical survey variable “weight” can take on any value from 2.5 to 635 kg. 

When registering continuous variables, please provide the following metadata:
```json
"metadata": {
	"variable_type": "numerical.continuous",
	"min_value": e.g. 0,
	"max_value": e.g. 100,
	"no_data_value": e.g. -999,
	"units": ""
}
```