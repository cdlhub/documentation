# Input data tools

The following components convert input data in csv format to the binary format required by the calculation components in the reference model;

* **[evetobin](#evetobin)** is a utility to convert a list of event_ids into binary format.
* **[damagetobin](#damagetobin)** is a utility to convert the Oasis damage bin dictionary table into binary format. 
* **[exposuretobin](#exposuretobin)** is a utility to convert the Oasis exposure instance table into binary format. 
* **[randtobin](#randtobin)** is a utility to convert a list of random numbers into binary format. 
* **[fmdatatobin](#fmdatatobin)** is a utility to convert the Oasis FM instance data into binary format.
* **[fmxreftobin](#fmxreftobin)** is a utility to convert the Oasis FM xref table into binary format.
* **[cdfdatatobin](#cdfdatatobin)** is a utility to convert the Oasis cdf data into binary format.

These components are intended to allow users to generate the required input binaries from csv independently of the original data store and technical environment. All that needs to be done is first generate the csv files from the data store (SQL Server database, etc).

```
Note that [oatools](https://github.com/OasisLMF/oatools) contains a component 'gendata' which generates all of the input binaries directly from a SQL Server Oasis back-end database. This component is specific to the implementation of the in-memory kernel as a calculation back-end to the Oasis mid-tier which is why it is kept in a separate project.
```

The following components convert the binary input data required by the calculation components in the reference model into csv format;
* **[evetocsv](#evetocsv)** is a utility to convert the event binary into csv format.
* **[damagetocsv](#damagetocsv)** is a utility to convert the Oasis damage bin dictionary binary into csv format.
* **[exposuretocsv](#exposuretocsv)** is a utility to convert the Oasis exposure instance binary into csv format.
* **[randtocsv](#randtocsv)** is a utility to convert the random numbers binary into csv format.
* **[fmdatatocsv](#fmdatatocsv)** is a utility to convert the Oasis FM instance binary into csv format.
* **[fmxreftocsv](#fmxreftocsv)** is a utility to convert the Oasis FM xref binary into csv format.
* **[cdfdatatocsv](#cdfdatatocsv)** is a utility to convert the Oasis cdf data binary into csv format.

## Events binary
One or more event binaries are required by all calculation components and must have the following filename format, which each uniquely identified by a chunk number (integer >=0);
* e_chunk_{chunk}_data.bin
The chunks represent partitions of the event set.

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
