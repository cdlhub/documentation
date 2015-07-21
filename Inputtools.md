# Input data tools

The following components convert input data in csv format to the binary format required by the calculation components in the reference model;

* **[evetobin](#events)** is a utility to convert a list of event_ids into binary format.
* **[damagetobin](#damagebins)** is a utility to convert the Oasis damage bin dictionary table into binary format. 
* **[exposuretobin](#exposuretobin)** is a utility to convert the Oasis exposure instance table into binary format. 
* **[randtobin](#randtobin)** is a utility to convert a list of random numbers into binary format. 
* **[cdfdatatobin](#cdfdatatobin)** is a utility to convert the Oasis cdf data into binary format.
* **[fmdatatobin](#fmdatatobin)** is a utility to convert the Oasis FM instance data into binary format.
* **[fmxreftobin](#fmxreftobin)** is a utility to convert the Oasis FM xref table into binary format.


These components are intended to allow users to generate the required input binaries from csv independently of the original data store and technical environment. All that needs to be done is first generate the csv files from the data store (SQL Server database, etc).

```
Note that [oatools](https://github.com/OasisLMF/oatools) contains a component 'gendata' which generates all of the input binaries directly from a SQL Server Oasis back-end database. This component is specific to the implementation of the in-memory kernel as a calculation back-end to the Oasis mid-tier which is why it is kept in a separate project.
```

The following components convert the binary input data required by the calculation components in the reference model into csv format;
* **[evetocsv](#events)** is a utility to convert the event binary into csv format.
* **[damagetocsv](#damagebins)** is a utility to convert the Oasis damage bin dictionary binary into csv format.
* **[exposuretocsv](#exposuretocsv)** is a utility to convert the Oasis exposure instance binary into csv format.
* **[randtocsv](#randtocsv)** is a utility to convert the random numbers binary into csv format.
* **[cdfdatatocsv](#cdfdatatocsv)** is a utility to convert the Oasis cdf data binary into csv format.
* **[fmdatatocsv](#fmdatatocsv)** is a utility to convert the Oasis FM instance binary into csv format.
* **[fmxreftocsv](#fmxreftocsv)** is a utility to convert the Oasis FM xref binary into csv format.

These components are provided for convenience of viewing the data and debugging.

## Events <a id="events"></a>
One or more event binaries are required by eve and getmodel. It must have the following filename format, each uniquely identified by a chunk number (integer >=0);
* e_chunk_{chunk}_data.bin
The chunks represent subsets of events.

```
In general more than 1 chunk of events is not necessary for the in-memory kernel as the computation can be parallelized across the processes. However it is theoretically possible to use the event chunk feature as a means of distributing work to multiple calculation back-ends if more computational power is required.
```

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

## Damage bins <a id="damagebins"></a>
The damage bin dictionary is a reference table in Oasis which defines how the effective damageability cdfs are discretized on a relative damage scale (normally between 0 and 1). It is required by getmodel and gulcalc and must have the following filename format;
* damage_bin_dict.bin

#### File format
The csv file should contain the following fields and include a header row.

| Name              | Type   |  Bytes | Description                                                   | Example     |
|:------------------|--------|--------| :-------------------------------------------------------------|------------:|
| bin_index         | int    |    4   | Identifier of the damage bin                                  |     1       |
| bin_from          | float  |    4   | Lower damage threshold for the bin                            |   0.01      |
| bin_to            | float  |    4   | Upper damage threshold for the bin                            |   0.02      |
| interpolation     | float  |    4   | Interpolation damage value for the bin (usually the mid-point)|   0.015     |
| interval_type     | int    |    4   | Identifier of the interval type, e.g. closed, open            |   1201      | 

The data should be ordered by bin_index ascending and bin_index should start from 1.

#### damagetobin
```
$ damagetobin < damage_bin_dict.csv > damage_bin_dict.bin
```

#### damagetocsv
```
$ damagetocsv < damage_bin_dict.bin > damage_bin_dict.csv
```

## Exposures <a id="exposures"></a>
The exposures binary contains the list of exposures for which ground up loss will be sampled in the kernel calculations. The data format is that of the Oasis Exposure instance. It is required by gulcalc and outputcalc and must have the following filename format;
* exposures.bin

#### File format
The csv file should contain the following fields and include a header row.

| Name              | Type   |  Bytes | Description                                                   | Example     |
|:------------------|--------|--------| :-------------------------------------------------------------|------------:|
| item_id           | int    |    4   | Identifier of the exposure item                               |     1       |
| areaperil_id      | int    |    4   | Identifier of the locator and peril of the item               |   4545      |
| vulnerability_id  | int    |    4   | Identifier of the vulnerability distribution of the item      |   345456    |
| tiv               | float  |    4   | The total insured value of the item                           |   200000    |
| group_id          | int    |    4   | Identifier of the correlation group of the item               |    1        |  

#### exposuretobin
```
$ exposuretobin < exposures.csv > exposures.bin
```

#### exposuretocsv
```
$ exposuretocsv < exposures.bin > exposures.csv
```

## Random numbers <a id="random"></a>
One or more random number files may be provided for the gulcalc component as an option (using gulcalc -r parameter) The random number binary contains a list of random numbers used for ground up loss sampling in the kernel calculation. It should be provided for the same number of chunks as events and must have the following filename format;
* random_{chunk}.bin

If the gulcalc -r parameter is not used, the random number binary is not required and random numbers are generated dynamically in the calculation. 

#### File format
The csv file should contain the runtime number of samples as the first value followed by a simple list of random numbers. It should not contain any headers.

First value;

| Name              | Type     |  Bytes | Description                  Example     |
|:------------------|----------|--------| :--------------------------|------------:|
| number of samples | integer  |    4   | Runtime number of samples  |  100        |

Subsequent values;

| Name              | Type   |  Bytes | Description                    | Example     |
|:------------------|--------|--------| :------------------------------|------------:|
| rand              | float  |    4   | Random number between 0 and 1  |  0.75875    |  

The first value in the file is an exception and should contain the runtime number of samples (integer > 0).  This is required for running gulcalc in 'Reconciliation mode' (using gulcalc -R parameter) which uses the same random numbers as Oasis classic and produces identical sampled values.  If not running in reconciliation mode, it is not used and can be set to 1.

#### randtobin
```
$ randtobin < random_1.csv > random_1.bin
```

#### randtocsv
```
$ randtocsv < random_1.bin > random_1.csv
```

## CDF <a id="cdfs"></a>
One or more cdf data files are required for the getmodel component, as well as an index file containing the starting positions of each event block. These should be located in a cdf sub-directory of the main working directory.
* cdf/damage_cdf_chunk_{chunk}.bin
* cdf/damage_cdf_chunk_{chunk}.bin

If chunked, the binary file should contain the cdf data for the same events as the corresponding chunk event binary. 

#### File format
The csv file should contain the following fields and include a header row.

| Name              | Type   |  Bytes | Description                                                   | Example     |
|:------------------|--------|--------| :-------------------------------------------------------------|------------:|
| event_id          | int    |    4   | Oasis event_id                                                |     1       |
| areaperil_id      | int    |    4   | Oasis areaperil_id                                            |   4545      |
| vulnerability_id  | int    |    4   | Oasis vulnerability_id                                        |   345456    |
| bin_index         | int    |    4   | Identifier of the damage bin                                  |     10      |
| prob_to           | float  |    4   | The cumulative probability at the upper damage bin threshold  |    0.765    | 

The data should be ordered by event_id, areaperil_id, vulnerability_id, bin_index ascending and bin_index should start at 1. All bins corresponding to the bin indexes in the damage bin dictionary should be present, except records may be truncated after the last bin where the prob_to = 1.

#### cdfdatatobin
Not yet implemented. 

#### cdfdatatocsv
```
$ cdfdatatocsv < damage_cdf_chunk_1.bin > damage_cdf_chunk_1.csv
```
