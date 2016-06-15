# Data conversion components <a id="dataconversioncomponents"></a>

The following components convert input data in csv format to the binary format required by the calculation components in the reference model;

Static data
* **[damagebintobin](#damagebins)** converts the damage bin dictionary. 
* **[footprinttobin](#footprint)** converts the event footprint.
* **[occurrencetobin](#occurrence)** converts the event occurrence data.
* **[randtobin](#rand)** converts a list of random numbers. 
* **[vulnerabilitytobin](#vulnerability)** converts the vulnerability data.

Input data
* **[coveragetobin](#coverage)** converts the coverages data.
* **[evetobin](#events)** converts a list of event_ids.
* **[itemtobin](#item)** converts the items data.
* **[gulsummaryxreftobin](#gulsummaryxref)** converts the gul summary xref data.
* **[fmpolicytctobin](#fmpolicytc)** converts the fm policytc data.
* **[fmprogrammetobin](#fmprogramme)** converts the fm programme data.
* **[fmprofiletobin](#fmprofile)** converts the fm profile data.
* **[fmsummaryxreftobin](#fmsummaryxref)** converts the fm summary xref data.
* **[fmxreftobin](#fmxref)** converts the fm xref data.

These components are intended to allow users to generate the required input binaries from csv independently of the original data store and technical environment. All that needs to be done is first generate the csv files from the data store (SQL Server database, etc).

The following components convert the binary input data required by the calculation components in the reference model into csv format;

Static data
* **[damagebintocsv](#damagebins)** converts the damage bin dictionary. 
* **[footprinttocsv](#footprint)** converts the event footprint.
* **[occurrencetocsv](#occurrence)** converts the event occurrence data.
* **[randtocsv](#rand)** converts a list of random numbers. 
* **[vulnerabilitytocsv](#vulnerability)** converts the vulnerability data.

Input data
* **[coveragetocsv](#coverage)** converts the coverages data.
* **[evetocsv](#events)** converts a list of event_ids.
* **[itemtocsv](#item)** converts the items data.
* **[gulsummaryxreftocsv](#gulsummaryxref)** converts the gul summary xref data.
* **[fmpolicytctocsv](#fmpolicytc)** converts the fm policytc data.
* **[fmprogrammetocsv](#fmprogramme)** converts the fm programme data.
* **[fmprofiletocsv](#fmprofile)** converts the fm profile data.
* **[fmsummaryxreftocsv](#fmsummaryxref)** converts the fm summary xref data.
* **[fmxreftocsv](#fmxref)** converts the fm xref data.

These components are provided for the convenience of viewing the data and debugging.

## Static data

### damage bins <a id="damagebins"></a>
The damage bin dictionary is a reference table in Oasis which defines how the effective damageability cdfs are discretized on a relative damage scale (normally between 0 and 1). It is required by getmodel and gulcalc and must have the following location and filename format;

* static/damage_bin_dict.bin

#### File format
The csv file should contain the following fields and include a header row.

| Name              | Type   |  Bytes | Description                                                   | Example     |
|:------------------|--------|--------| :-------------------------------------------------------------|------------:|
| bin_index         | int    |    4   | Identifier of the damage bin                                  |     1       |
| bin_from          | float  |    4   | Lower damage threshold for the bin                            |   0.01      |
| bin_to            | float  |    4   | Upper damage threshold for the bin                            |   0.02      |
| interpolation     | float  |    4   | Interpolation damage value for the bin (usually the mid-point)|   0.015     |
| interval_type     | int    |    4   | Identifier of the interval type, e.g. closed, open            |   1201      | 

The data should be ordered by bin_index ascending with bin_index starting from 1 and not contain nulls.

#### damagetobin
```
$ damagetobin < damage_bin_dict.csv > damage_bin_dict.bin
```

#### damagetocsv
```
$ damagetocsv < damage_bin_dict.bin > damage_bin_dict.csv
```
[Return to top](#dataconversioncomponents)

<a id="event footprint"></a>
### footprint
The event footprint is  required for the getmodel component, as well as an index file containing the starting positions of each event block. These must be in the following location with filename formats;

* static/eventfootprint.bin
* static/eventfootprint.idx

#### File format
The csv file should contain the following fields and include a header row.

| Name               | Type   |  Bytes | Description                                                   | Example     |
|:-------------------|--------|--------| :-------------------------------------------------------------|------------:|
| event_id           | int    |    4   | Oasis event_id                                                |     1       |
| areaperil_id       | int    |    4   | Oasis areaperil_id                                            |   4545      |
| intensity_bin_index| int    |    4   | Identifier of the intensity damage bin                        |     10      |
| prob               | float  |    4   | The probability mass for the intensity bin between 0 and 1    |    0.765    | 

The data should be ordered by event_id, areaperil_id with bin_index starting at 1, and not contain nulls. 

| vulnerability_id   | int    |    4   | Oasis vulnerability_id                                        |   345456    |

#### footprinttobin
```
$ footprinttobin -bin eventfootprint.bin -idx eventfootprint.idx < eventfootprint.csv
```
This command will create a binary file eventfootprint.bin and an index file eventfootprint.idx in the working directory.

In general the usage is;

```
$ footprinttobin -bin {binary file name} -idx {index file name} < input.csv
```
input.csv must conform to the csv format given above.

#### footprinttocsv
```
$ footprinttocsv < eventfootprint.bin > eventfootprint.csv
```

[Return to top](#dataconversioncomponents)

### occurrence
The occurrence file is required for certain output components which, in the reference model, are leccalc, pltcalc and aalcalc.  In general, some form of event occurence file is required for any output which involves the calculation of loss metrics over a period of time.  The occurrence file assigns occurrences of the event_ids to numbered periods. A period can represent any length of time, such as a year or 2 years, or 18 months. The output metrics such as mean, standard deviation or loss exceedance probabilities are with respect to the chosen period length.  Most frequently, the period of interest is a year.

The occurrence file also includes date fields.  There are two options;
* date_id: An integer representing an offset number of days to some real date.
* occ_year, occ_month, occ_day. These are all integers representing occurrence year, month and day.


<a id="events"></a>
## Events
One or more event binaries are required by eve and getmodel. It must have the following location and filename format;
* input/events.bin

#### File format
The csv file should contain a list of event_ids (integers) and no header.

| Name              | Type   |  Bytes | Description         | Example     |
|:------------------|--------|--------| :-------------------|------------:|
| event_id          | int    |    4   | Oasis event_id      |   4545      |

#### evetobin
```
$ evetobin < e_chunk_1_data.csv > e_chunk_1_data.bin
```

#### evetocsv
```
$ evetocsv < e_chunk_1_data.bin > e_chunk_1_data.csv
```
[Return to top](#dataconversioncomponents)



<a id="item"></a>
## Items
The items binary contains the list of exposure items for which ground up loss will be sampled in the kernel calculations. The data format is as follows. It is required by gulcalc and outputcalc and must have the following filename format and location;
* input/items.bin

#### File format
The csv file should contain the following fields and include a header row.

| Name              | Type   |  Bytes | Description                                                   | Example     |
|:------------------|--------|--------| :-------------------------------------------------------------|------------:|
| item_id           | int    |    4   | Identifier of the exposure item                               |     1       |
| coverage _id      | int    |    4   | Identifier of the coverage                                    |   1         |
| areaperil_id      | int    |    4   | Identifier of the locator and peril of the item               |   4545      |
| vulnerability_id  | int    |    4   | Identifier of the vulnerability distribution of the item      |   345456    |
| group_id          | int    |    4   | Identifier of the correlation group of the item               |    1        |  

The data should be ordered by areaperil_id, vulnerability_id ascending and not contain nulls.

#### itemtobin
```
$ itemtobin < items.csv > items.bin
```

#### itemtocsv
```
$ itemtocsv < items.bin > items.csv
```

[Return to top](#dataconversioncomponents)

<a id="coverage"></a>
## Coverages
The coverages binary contains the list of coverages and the coverage TIVs. The data format is as follows. It is required by gulcalc and fmcalc and must have the following location and filename format;

* input/coverages.bin

#### File format
The csv file should contain the following fields and include a header row.

| Name              | Type   |  Bytes | Description                                                   | Example     |
|:------------------|--------|--------| :-------------------------------------------------------------|------------:|
| coverage_id       | int    |    4   | Identifier of the coverage                                    |     1       |
| tiv               | float  |    4   | The total insured value of the coverage                       |   200000    |

The data should not contain nulls.

#### coveragetobin
```
$ coveragetobin < coverages.csv > coverages.bin
```

#### coveragetocsv
```
$ coveragestocsv < coverages.bin > coverages.csv
```

[Return to top](#dataconversioncomponents)

<a id="rand"></a>
## Random numbers 
A random number file may be provided for the gulcalc component as an option (using gulcalc -r parameter) The random number binary contains a list of random numbers used for ground up loss sampling in the kernel calculation. It must have the following location and filename format;
* static/random.bin

If the gulcalc -r parameter is not used, the random number binary is not required and random numbers are generated dynamically in the calculation. 

#### File format
The csv file should contain a simple list of random numbers. It should not contain any headers.

| Name              | Type   |  Bytes | Description                    | Example     |
|:------------------|--------|--------| :------------------------------|------------:|
| rand              | float  |    4   | Random number between 0 and 1  |  0.75875    |  


#### randtobin
```
$ randtobin < random.csv > random.bin
```

#### randtocsv
```
$ randtocsv < random.bin > random.csv
```

[Return to top](#dataconversioncomponents)



<a id="fmprogramme"></a>
## fm programme 
The fm programme binary file contains the level heirarchy and defines aggregations of losses required to perform a loss calculation, and is required for fmcalc only. 

This must be in the following location with filename format;
* input/fm_programme.bin

#### File format
The csv file should contain the following fields and include a header row.


| Name                     | Type   |  Bytes | Description                                    | Example     |
|:-------------------------|--------|--------| :----------------------------------------------|------------:|
| from_agg_id              | int    |    4   | Oasis Financial Module from_agg_id             |    1        |
| level_id                 | int    |    4   | Oasis Financial Module level_id                |     1       |
| to_agg_id                | int    |    4   | Oasis Financial Module to_agg_id               |     1       |

* All fields must have integer values and no nulls
* Must have at least one level where level_id = 1, 2, 3 ...
* For level_id = 1, the set of values in from_agg_id must be equal to the set of item_ids in the input ground up loss stream (which has fields event_id, item_id, idx, gul).  Therefore level 1 always defines a group of items.
* For subsequent levels, the from_agg_id must be the distinct values from the previous level to_agg_id field.
* The from_agg_id and to_agg_id values, for each level, should be a contiguous block of integers (a sequence with no gaps).  This is not a strict rule in this version and it will work with non-contiguous integers, but it is recommended as best practice.

#### fmprogrammetobin
```
$ fmprogrammetobin < fm_programme.csv > fm_programme.bin
``` 

#### fmprogrammetocsv
```
$ fmprogrammetocsv < fm_programme.bin > fm_programme.csv
```

<a id="fmprofile"></a>
## fm profile
The fmprofile binary file contains the list of calculation rules with profile values (policytc_ids) that appear in the policytc file. This is required for fmcalc only. 

This must be in the following location with filename format;
* input/fm_profile.bin

#### File format
The csv file should contain the following fields and include a header row.

| Name                     | Type   |  Bytes | Description                                    | Example     |
|:-------------------------|--------|--------| :----------------------------------------------|------------:|
| policytc_id              | int    |    4   | Oasis Financial Module policytc_id             |     34      |
| calcrule_id              | int    |    4   | Oasis Financial Module calcrule_id             |      1      |
| allocrule_id             | int    |    4   | Oasis Financial Module allocrule_id            |      0      |
| ccy_id                   | int    |    4   | Oasis Financial Module ccy_id                  |      0      |
| deductible               | float  |    4   | Deductible                                     |   50        |
| limit                    | float  |    4   | Limit                                          |   100000    |
| share_prop_of_lim        | float  |    4   | Share/participation as a proportion of limit   |   0         |
| deductible_prop_of_loss  | float  |    4   | Deductible as a proportion of loss             |   0         |
| limit_prop_of_loss       | float  |    4   | Limit as a proportion of loss                  |   0         |
| deductible_prop_of_tiv   | float  |    4   | Deductible as a proportion of TIV              |   0         |
| limit_prop_of_tiv        | float  |    4   | Limit as a proportion of TIV                   |   0         |
| deductible_prop_of_limit | float  |    4   | Deductible as a proportion of limit            |   0         |

* All distinct policytc_id values that appear in the policytc table must appear once in the policytc_id field of the profile table. We suggest that policytc_id=1 is included by default using calcrule_id = 12 and DED = 0 as a default 'null' calculation rule whenever no terms and conditions apply to a particular level_id / agg_id in the policytc table.
* All data fields that are required by the relevant profile must be provided, with the correct calcrule_id (see FM Profiles)
* Any fields that are not required for the profile should be set to zero.
* allocrule_id may be set to 0 or 1 for each policytc_id.  Generally, it is recommended to use 0 everywhere except for the final level calculations when back-allocated losses are required, else 0 everywhere.
* The ccy_id field is currently not used.

#### fmprofiletobin
```
$ fmprofiletobin < fm_profile.csv > fm_profile.bin
``` 

#### fmprofiletocsv
```
$ fmprofiletocsv < fm_profile.bin > fm_profile.csv
```

<a id="fmpolicytc"></a>
## fm policytc
The fm policytc binary file contains the cross reference between the aggregations of losses defined in the fm programme file at a particular level and the calculation rule that should be applied as defined in the fm profile file. This is required for fmcalc only. 

This must be in the following location with filename format;
* input/fm_policytc.bin

#### File format
The csv file should contain the following fields and include a header row.


| Name                     | Type   |  Bytes | Description                                    | Example     |
|:-------------------------|--------|--------| :----------------------------------------------|------------:|
| layer_id                 | int    |    4   | Oasis Financial Module layer_id                |    1        |
| level_id                 | int    |    4   | Oasis Financial Module level_id                |     1       |
| agg_id                   | int    |    4   | Oasis Financial Module agg_id                  |     1       |
| policytc_id              | int    |    4   | Oasis Financial Module policytc_id             |     1       |

* All fields must have integer values and no nulls
* Must contain the same levels as the fm programme where level_id = 1, 2, 3 ...
* For every distinct combination of to_agg_id and level_id in the programme table, there must be a corresponding record matching level_id and agg_id values in the policytc table with a valid value in the policytc_id field.  
* layer_id = 1 at all levels except the last where there may be multiple layers, with layer_id = 1, 2, 3 ... This allows for the specification of several policy contracts applied to the same aggregation of losses defined in the programme table.
 
#### fmpolicytctobin
```
$ fmpolicytctobin < fm_policytc.csv > fm_policytc.bin
``` 

#### fmpolicytctocsv
```
$ fmpolicytctocsv < fm_policytc.bin > fm_policytc.csv
```

[Return to top](#inputtools)


<a id="fmxref"></a>
## fm xref 
The fmxref binary file contains cross reference data specifying the output_id in the fmcalc as a combination of agg_id and layer_id, and is required for fmcalc only. 

This must be in the following location with filename format;
* input/fm_xref.bin

#### File format
The csv file should contain the following fields and include a header row.

| Name                        | Type   |  Bytes | Description                                    | Example     |
|:----------------------------|--------|--------| :----------------------------------------------|------------:|
| output_id                   | int    |    4   | Identifier of the output group of losses       |     1       |
| agg_id                      | int    |    4   | Identifier of the agg_id to output             |     1       |
| layer_id                    | int    |    4   | Identifier of the layer_id to output           |     1       |

The data should not contain any nulls.

The output_id represents the summary level at which losses are output from fmcalc, as specified by the user.

There are two cases;
* losses are output at the final level of aggregation (represented by the final level to_agg_id's from the fm programme file ) for each contract or layer (represented by the final level layer_id's in the fm policytc file)
* losses are back-allocated to the item level and output by item (represented by the from_agg_id of level 1 from the fm programme file) for each policy contract / layer (represented by the final level layer_id's in the fm policytc file)

For example, say there are two policy layers (with layer_ids=1 and 2) which applies to the sum of losses from 4 items (the summary level represented by agg_id=1). Without back-allocation, the policy summary level of losses can be represented as two output_id's as follows;

| output_id | agg_id   | layer_id    |
|:----------|----------|------------:|
| 1         | 1        |    1        | 
| 2         | 1        |    2        | 

If the user wants to back-allocate policy losses to the items and output the losses by item and policy, then the item-policy summary level of losses would be represented by 8 output_id's, as follows;

| output_id | agg_id   | layer_id    |
|:----------|----------|------------:|
| 1         | 1        |    1        | 
| 2         | 2        |    1        | 
| 3         | 3        |    1        | 
| 4         | 4        |    1        |
| 5         | 1        |    2        | 
| 6         | 2        |    2        |
| 7         | 3        |    2        | 
| 8         | 4        |    2        |

#### fmxreftobin
```
$ fmxreftobin < fm_xref.csv > fm_xref.bin
``` 

#### fmxreftocsv
```
$ fmxreftocsv < fm_xref.bin > fm_xref.csv
``` 

[Return to top](#dataconversioncomponents)

[Go to Stream conversion components section](StreamConversionComponents.md)

[Back to Contents](Contents.md)
