# Output data tools

In addition, some components which convert the binary output of each calculation step to csv format are provided.
* **cdftocsv** is a utility to convert binary format CDFs to a csv. getmodel standard output can be streamed directly into cdftocsv, or a binary file of the same format can be input.
* **gultocsv** is a utility to convert binary format GULs to a csv. gulcalc standard output can be streamed directly into gultocsv, or a binary file of the same format can be input.
* **fmtocsv** is a utility to convert binary format losses to a csv. fmcalc standard output can be streamed directly into fmtocsv, or a binary file of the same format can be input.

Figure 1 shows the workflows for the data conversion components.

##### Figure 1. Data Conversion Workflows
![alt text](https://github.com/OasisLMF/ktools/blob/master/docs/img/Dbtools2.jpg "Data Conversion Workflows")

```
Note that no examples of the component which generates the binary files have been provided in the tool set 
as yet. This component is, in general, specific to the technical environment and may also be calling 
the data from a remote server as an API. However an example of 'gendata' is provided with Oasis R1.4 
in a github project called 'oatools' which reads the input data from a SQL Server database.
```
