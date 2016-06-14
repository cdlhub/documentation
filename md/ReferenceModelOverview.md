# Reference Model Overview <a id="referencemodel"></a>

This section provides an overview of the reference model, which is an implementation of each of the components in the framework. 

There are four subsections which cover, in more detail;

* The [core components](CoreComponents.md)
* The [output components](OutputComponents.md)
* The [input data conversion components](InputConversionComponents.md)
* The [stream conversion components](StreamConversionComponents.md)

The set of **core components** provided in this release is as follows;
* **[eve](#eve)** is the event distributing utility. Based on the number of events in the input and the number of processes specified as a parameter, eve outputs subsets of the events as a stream. The output streams into getmodel.
* **[getmodel](#getmodel)** generates a stream of effective damageability cdfs for the input stream of events. The reference example generates these from the model files eventfootprint.bin and vulnerability.bin, and the user's exposures file which is called items.bin. getmodel streams into gulcalc or can be output to a binary file.
* **[gulcalc](#gulcalc)** performs the ground up loss sampling calculations and numerical integration. The output is the Oasis kernel gul sample table. This can be output to a binary file or streamed into  fmcalc or summarycalc.
* **[fmcalc](#fmcalc)** performs the insured loss calculations on the ground up loss samples and mean. The output is the Oasis format loss sample table. The result can be output to a binary file or streamed into summarycalc.
* **[summarycalc](#summarycalc)** performs a summing of sampled losses according to the user's reporting requirements.  For example this might involve summing coverage losses to regional level, or policy losses to portfolio level.  The output is sampled loss by event_id and summary_id, which represents a meaningful grouping of losses to the user. 

The **output components** are different implementations of **outputcalc**. The results are written directly into csv file as there is no downstream processing.

* **[eltcalc](#eltcalc)**  generates an event loss table from the sampled losses from summarycalc. It contains sample mean and standard deviation, and total exposed value for each event at the given summary level. 
* **[leccalc](#leccalc)**  generates loss exceedance curve from the sampled losses from summarycalc. There are 8 variants of curves with small differences in the output format but the common output fields are summary_id, return period, and loss exceedance threshold. This output is only available for models which provide an event **occurrence** file.
* **[pltcalc](#pltcalc)**  generates a period loss table from the sampled losses from summarycalc. It contains sample mean and standard deviation, and total exposed value for each event and each period (for example a year) at the given summary level. It also contains a date field, corresponding to the date of the event occurrence. This output is only available for models which provide an event **occurrence** file.
* **[aalcalc](#aalcalc)**  generates the average annual loss and standard deviation of loss from the sampled losses from summarycalc, for each summary_id. It also contains total exposed value for each summary level, which is the maximum of the total exposed value across all simulated periods. This output is only available for models which provide an event **occurrence** file.

Other components in the Reference Model include;
* **[Input conversion components](Inputtools.md)** to convert input data between csv and binary.
* **[Stream conversion components](Outputtools.md)** to convert the binary data stream output to csv. These are useful for 

Figure 1 shows the core data stream workflow of the reference model with its particular internal data files.

##### Figure 1. Core workflow and required data
![alt text](../img/KtoolsRequiredData1.jpg "Core workflow and required data")

The model / static data for the core workflow, shown as red source files, are the event footprint, vulnerability, damage bin dictionary and random number file.  These are stored in the 'static' sub-directory of the working folder.  

The user / analysis input data for the core workflow, shown as blue source files, are the events, items, coverages, fm programme, fm policytc, fm profile, fm xref, fm summary xref and gul summary xref files. These are stored in the 'input' sub-directory of the working folder. 

These are all Oasis kernel format data objects with fixed formats. Note that the events are a user input rather than a static input because the user could choose to run a subset of the full list of events, or even just one event. Usually, however, the whole event set will be run. 

There are some further files required for the output components as shown in Figure 2.

##### Figure 2. Output workflows and required data
![alt text](../img/OutputsRequiredData.jpg "Output workflows and required data")

The following sections explain the usage and internal processes of each of the reference components. The standard input and output data streams for the components are generic and are covered in the Specification. The input data requirements are covered in [Input data components](Inputtools.md).

<a id="eve"></a>
## eve
eve takes as input a list of event ids in binary format and generates a partition of event ids as a binary data stream, according to the parameters supplied. 

##### Stream_id
None. The output stream is a simple list of event_ids (4 byte integers).

##### Parameters
* process number - determines which partition of events to output to a single process
* number of processes - determines the total number of partitions of events to distribute to processes

##### Usage
```
$ eve [parameters] > [output].bin
$ eve [parameters] | getmodel | gulcalc [parameters] > [stdout].bin
```

##### Example
```
$ eve 1 2 > events1_2.bin 
$ eve 1 2 | getmodel | gulcalc -S100 -i - > gulcalc.bin
```

In this example, the events from the file **events.bin** will be read into memory and the first half (partition 1 of 2) would be streamed out to binary file, or downstream to a single process calculation workflow.

##### Internal data
The program requires an event binary. The file is picked up from the **input** sub-directory relative to where the program is invoked and has the following filename;
* input/events.bin

The data structure of events.bin is a simple list of event ids (4 byte integers).

[Return to top](#referencemodel)

<a id="getmodel"></a>
## getmodel 

getmodel generates a stream of effective damageability distributions (cdfs) from an input list of events. Specifically, it combines the probability distributions from the model files, eventfootprint.bin and vulnerability.bin, to generate effective damageability cdfs for the subset of exposures contained in the items.bin file and converts them into a binary stream. The source input data must have been generated as binary files by a separate program.

This is reference example of the class of programs which generates the damage distributions for an event set and streams them into memory. It is envisaged that model developers who wish to use the toolkit as a back-end calculator of their existing platforms can write their own version of getmodel, reading in their own source data and converting it into the standard output stream. As long as the standard input and output structures are adhered to, the program can be written in any language and read any input data.

##### Stream_id

| Byte 1 | Bytes 2-4 |  Description                                   |
|:-------|-----------|:-----------------------------------------------|
|    0   |     1     |  getmodel reference example                    |

##### Parameters
None

##### Usage
```
$ [stdin component] | getmodel | [stout component]
$ [stdin component] | getmodel > [stdout].bin
$ getmodel < [stdin].bin > [stdout].bin
```

##### Example
```
$ eve 1 1 | getmodel | gulcalc -r -S100 -i gulcalci.bin
$ eve 1 1 | getmodel > getmodel.bin
$ getmodel < events.bin > getmodel.bin 
```

##### Internal data
The program requires the eventfootprint binary and index file for the model, the vulnerability binary model file, the items file representing the user's exposures. The files are picked up from sub-directories relative to where the program is invoked, as follows;

* static/eventfootprint.bin
* static/eventfootprint.idx
* static/vulnerability.bin
* static/damage_bin_dict.bin
* input/items.bin

The getmodel output stream is ordered by event and streamed out in blocks for each event. 

##### Calculation
The program filters the eventfootprint binary file for all 'areaperil_id's which appear in the items file. This selects the event footprints that affect the exposures on the basis on their location, and discards the rest.  Similarly the program filters the vulnerability file for 'vulnerability_id's that appear in the items file. This removes damage distributions which are not relevant for the exposures. Then the intensity distributions from the eventfootprint file and conditional damage distributions from the vulnerability file are convolved for every combination of areaperil_id and vulnerability_id in the items file. The effective damage probabilities are calculated by a sum product of conditional damage probabilities with intensity probabilities for each event, areaperil, vulnerability combination and each damage bin.  The resulting discrete probability distributions are converted into discrete cumulative distribution functions 'cdfs'.  Finally, the damage bin mid-point from the damage bin dictionary ('interpolation' field) is read in as a new field in the cdf stream as 'bin_mean'.  This field is the conditional mean damage for the bin and it is used to facilitate the calculation of mean and standard deviation in the gulcalc component. 

[Return to top](#referencemodel)

<a id="gulcalc"></a>
## gulcalc 
The gulcalc program performs Monte Carlo sampling of ground up loss and calculates mean and standard deviation by numerical integration of the cdfs. The sampling methodology has been extended beyond linear interpolation to include point value sampling and quadratic interpolation. This supports damage bin intervals which represent a single discrete damage value, and damage distributions with cdfs that are described by a piecewise quadratic function. 

##### Stream_id

| Byte 1 | Bytes 2-4 |  Description                                   |
|:-------|-----------|:-----------------------------------------------|
|    1   |     1     |  gulcalc reference example                     |

##### Parameters
Required parameters are;
* -R{number} or -r.  Number of random numbers to generate or read random numbers from file
* -S{number}. Number of samples
* -c or -i {destination}. There are two alternative streams, 'coverage' or 'item'. 
The destination is either a filename or named pipe, or use - for standard output.

Optional parameters are;

* -L{number} loss threshold (optional) excludes losses below the threshold from the output stream
* -d debug mode - output random numbers rather than losses (optional)

##### Usage
```
$ [stdin component] | gulcalc [parameters] | [stout component]
$ [stdin component] | gulcalc [parameters]
$ gulcalc [parameters] < [stdin].bin 
```

##### Example
```
$ eve 1 1 | getmodel | gulcalc -R1000000 -S100 -i - | fmcalc > fmcalc.bin
$ eve 1 1 | getmodel | gulcalc -R1000000 -S100 -c - | summarycalc -g -1 summarycalc1.bin
$ eve 1 1 | getmodel | gulcalc -r -S100 -i gulcalci.bin
$ eve 1 1 | getmodel | gulcalc -r -S100 -c gulcalcc.bin
$ gulcalc -r -S100 -c gulcalcc.bin < getmodel.bin 
```

##### Internal data
The program requires the damage bin dictionary for the static folder and the item and coverage files from the input folder, all binary files. The files are found in the following locations relative to the working directory with the filenames;

* static/damage_bin_dictionary.bin
* input/items.bin
* input/coverages.bin

If the user specifies -r as a parameter, then the program also picks up a random number file from the static directory. The filename is;
* static/random.bin

##### Calculation
The stdin stream is a block of cdfs which are ordered by event_id, areaperil_id, vulnerability_id and bin_index asccending, from getmodel. The gulcalc program constructs a cdf for each item, based on matching the areaperil_id and vulnerability_id from the stdin and the item file.

For each item cdf and for the number of samples specified, the program draws a random number and uses it to sample ground up loss from the cdf using one of three methods, as follows;

For a given damage interval corresponding to a cumulative probability interval that each random number falls within;
* If the conditional mean damage (of the cdf) is the mid-point of the damage bin interval (of the damage bin dictionary) then the gulcalc program performs linear interpolation. 
* If the conditional mean damage is equal to the lower and upper damage threshold of the damage bin interval (i.e the bin represents a damage value, not a range) then that value is sampled.
* Else, the gulcalc program performs quadrative interpolation using the bin_mean to calculate the quadratic equation in the damage interval.

An example of the three cases and methods is given below;
 
| bin_from | bin_to |  bin_mean | Method used             |
|:---------|--------|-----------| :-----------------------|
|    0.1   |  0.2   |    0.15   | Linear interpolation    |
|    0.1   |  0.1   |    0.1    | Sample bin value        |
|    0.1   |  0.2   |    0.14   | Quadratic interpolation |

Each sampled damage is multiplied by the item TIV, looked up from the coverage file.

If the -i parameter is specified, then the ground up losses for the items are output to the stream (stream_id 1).
If the -c parameter is specified, then the ground up losses for the items are grouped by coverage and capped to the coverage TIV, and the ground up losses for the coverages are output to the stream (stream_id 2).

The purpose of the coverage stream is to cap losses to the coverage TIV where items represent damage to the same coverage from different perils.

A final calculation which occurs in the gulcalc program is of the mean and standard deviation of ground up loss. For each cdf, the mean and standard deviation of damage is calculated by numerical integration of the effective damageability probability distribution and the result is multiplied by the TIV. The results are included in the output to the stdout stream as sidx=-1 (mean) and sidx=-2 (standard deviation), for each event and item (or coverage).

If the -R parameter is used along with a specified number of random numbers then random numbers used for sampling are generated on the fly for each event and group of items which have a common group_id using the Mersenne twister psuedo random number generator (the default RNG of the C++ v11 compiler).  if the -r parameter is used, gulcalc reads a random number from the provided random number file. See [Random Numbers](RandomNumbers.md) for more details.

[Return to top](#referencemodel)

<a id="fmcalc"></a>
## fmcalc 
fmcalc is the in-memory implementation of the Oasis Financial Module. It applies policy terms and conditions to the ground up losses and produces insured loss sample output.  It takes in losses from gulcalc at item level only (stream_id = 1). 

##### Stream_id

| Byte 1 | Bytes 2-4 |  Description                                   |
|:-------|-----------|:-----------------------------------------------|
|    2   |     1     |  fmcalc reference example                      |

##### Parameters
There are no parameters as all of the information is taken from the gul stdout stream and fm input data.

##### Usage
```
$ [stdin component] | fmcalc | [stout component]
$ [stdin component] | fmcalc > [stdout].bin
$ fmcalc < [stdin].bin > [stdout].bin
```

##### Example
```
$ eve 1 1 | getmodel | gulcalc -r -S100 -i - | fmcalc | summarycalc -f -2 - | eltcalc > elt.csv
$ eve 1 1 | getmodel | gulcalc -r -S100 -i - | fmcalc > fmcalc.bin
$ fmcalc < gulcalc.bin > fmcalc.bin 
```

##### Internal data
The program requires the item, coverage and fm input data, which are Oasis abstract data objects that describe an insurance programme. This data is picked up from the following files relative to the working directory;

* input/items.bin
* input/coverages.bin
* input/fm_programme.bin
* input/fm_policytc.bin
* input/fm_profile.bin
* input/fm_xref.bin

##### Calculation
See [Financial Module](FinancialModule.md)

[Return to top](#referencemodel)

<a id="summarycalc"></a>
## summarycalc 
The purpose of summarycalc is firstly to aggregate the samples of loss to a level of interest for reporting, thereby reducing the volume of data in the stream. This is a general first step which precedes all of the downstream output calculations.  Secondly, it unifies the formats of the gulcalc and fmcalc streams, so that they are transformed into an identical stream type for downstream outputs. Finally, it can generate up to 10 summary level outputs in one go, creating multiple output streams or files.

The output is similar to the gulcalc or fmcalc input which are losses are by sample index and by event, but the coverage or policy input losses are grouped to an abstract level represented by a summary_id.  The relationship between the input identifier and the summary_id are defined in cross reference files called **gulsummaryxref** and **fmsummaryxref**.

##### Stream_id

| Byte 1 | Bytes 2-4 |  Description                                   |
|:-------|-----------|:-----------------------------------------------|
|    3   |     1     |  summarycalc reference example                 |

##### Parameters

The input stream should be identified explicitly as -g for gulcalc stream or -f for fmcalc stream.

summarycalc supports up to 10 concurrent outputs.  This is achieved by explictly directing each output to a named pipe, file, or to standard output.  

For each output stream, the following tuple of parameters must be specified for at least one summary set;

* summaryset_id number. valid values are -0 to -9
* destination pipe or file. Use either - for standard output, or {name} for a named pipe or file.

For example the following parameter choices are valid;
```
$ summarycalc -g -1 -                                       
'outputs summaryset 1 results to standard output
$ summarycalc -g -1 summarycalc1.bin                        
'outputs summaryset 1 results to a file (or named pipe)
$ summarycalc -g -1 summarycalc1.bin -2 summarycalc2.bin    
'outputs summaryset 1 and 2 results to a file (or named pipe)
```
Note that the summaryset_id relates to a summaryset_id in the required input data file **gulsummaryxref.bin** or **fmsummaryxref.bin** for a gulcalc input stream or a fmcalc input stream, respectively, and represents a user specified summary reporting level. For example summaryset_id = 1 represents portfolio level, summaryset_id = 2 represents zipcode level and summaryset_id 3 represents site level.

##### Usage
```
$ [stdin component] | summarycalc [parameters] | [stdout component]
$ [stdin component] | summarycalc [parameters]
$ summarycalc [parameters] < [stdin].bin
```

##### Example

```
$ eve 1 1 | getmodel | gulcalc -S100 -c - | summarycalc -g -1 - | eltcalc > eltcalc.csv
$ eve 1 1 | getmodel | gulcalc -S100 -c - | summarycalc -g -1 gulsummarycalc.bin 
$ summarycalc -f -1 fmsummarycalc.bin < fmcalc.bin
```

##### Internal data
The program requires the gulsummaryxref file for gulcalc input (-g option), or the fmsummaryxref file for fmcalc input (-f option). This data is picked up from the following files relative to the working directory;

* input/gulsummaryxref.bin
* input/fmsummaryxref.bin

##### Calculation
summarycalc takes either ground up loss (by coverage) or insured loss samples (by policy) as input and aggregates them to a user-defined summary reporting level. The output is similar to the input which are losses are by sample index and by event, but the coverage or policy input losses are grouped to an abstract level represented by a summary_id.  The relationship between the input identifier, coverage_id for gulcalc or output_id for fmcalc, and the summary_id are defined in the internal data files.

summarycalc also calculates the maximum exposure value by summary_id and event_id. This is carried through in the stream header and output by eltcalc, aalcalc and pltcalc. For ground up losses this is the sum of TIV for all coverages which have a non-zero sampled ground up loss for a given event. For insured losses, this is the sum of the losses for sample index -3 from the fmcalc stdout stream.  Sample index -3 is the TIV of the items in the programme which have been passed through the financial module calculations to take account of the deductibles and limits.  Essentially, it is a 100% damage scenario for all items simultaneously.

## eltcalc <a id="eltcalc"></a>

The program outputs sample mean and standard deviation by summary_id and by event_id.

##### Parameters

None

##### Usage
```
$ [stdin component] | eltcalc > elt.csv
$ eltcalc < [stdin].bin > elt.csv
```

##### Example
```
$ eve 1 1 | getmodel | gulcalc -S100 -c - | summarycalc -g -1 - | eltcalc > elt.csv
$ eltcalc < summarycalc.bin > elt.csv 
```

##### Internal data

No additional data is required, all the information is contained within the input stream. 

##### Calculation

For each summary_id and event_id, the sample mean and standard deviation is calculated from the sampled losses in the summarycalc stream and output to file.  The exposure_value, which is carried in the event_id, summary_id header of the stream is also output.

##### Output
csv file with the following fields;

| Name              | Type   |  Bytes | Description                                                         | Example     |
|:------------------|--------|--------| :-------------------------------------------------------------------|------------:|
| summary_id        | int    |    4   | summary_id representing a grouping of losses                        |   10        |
| event_id          | int    |    4   | Oasis event_id                                                      |  45567      |
| mean              | float  |    4   | sample mean                                                         |   1345.678  |
| standard_deviation| float  |    4   | sample standard deviation                                           |    945.89   | 
| exposure_value    | float  |    4   | sum of TIV for summary_id affected by event_id                      |   70000     |


[Return to top](#outputs)

## leccalc <a id="leccalc"></a>

Loss exceedance curves, also known as exceedance probability curves, are computed by a rank ordering a set of losses by period and computing the probability of exceedance for each level of loss in any given period based on relative frequency. Period losses are first computed by reference to the **occurrence** file which contains the event occurrences against each period. Event losses are summed within each period for an Aggregate loss exceedance curve, or the maximum of the event losses in each period is taken for an Occurrence loss exceedance curve.  From this point, there are a few variants available as follows;

* Full uncertainty
* Wheatsheaf
* Sample mean
* Wheatsheaf mean

##### Parameters
* -K{sub-directory}. The subdirectory of /work containing the input summarycalc binary files.
* -r. Use return period file - use this parameter if you are providing a file with a specific list of return periods. If this file is not present then all calculated return periods will be returned, for losses greater than zero.
Then the following tuple of parameters must be specified for at least one analysis type;
* Analysis type. Use -F for Full Uncertainty Aggregate, -f for Full Uncertainty Occurrence, -W for Wheatsheaf Aggregate,  -w for Wheatsheaf Occurrence, -S for Sample Mean Aggregate, -s for Sample Mean Occurrence, -M for Mean of Wheatsheaf Aggregate, -m for Mean of Wheatsheaf Occurrence
* Output filename
 

##### Usage
```
$ leccalc [parameters] > lec.csv

```

##### Example
```
$ eve 1 1 | getmodel | gulcalc -r -S100 -c - | summarycalc -g -1 - > work/summary1/summarycalc1.bin
$ leccalc -Ksummary1 -F lec_full_uncertainty_agg.csv -f lec_full_uncertainty_occ.csv 
```

##### Internal data

leccalc requires the occurrence.bin file 

* static/occurrence.bin

leccalc does not have a standard input that can be streamed in. Instead, it reads in summarycalc binary data from a file in a fixed location.  The format of the binaries must match summarycalc standard output. The location is in the 'work' subdirectory of the present working directory. For example;

* work/summarycalc1.bin
* work/summarycalc2.bin
* work/summarycalc3.bin

The user must ensure the work subdirectory exists.  The user may also specify a subdirectory of work to store these files. e.g.

* work/summaryset1/summarycalc1.bin
* work/summaryset1/summarycalc2.bin
* work/summaryset1/summarycalc3.bin

The reason for leccalc not having an input stream is that the calculation is not valid on a subset of events, i.e. within a single process when distributing events across multiple processes.  It must bring together all event losses before assigning event losses to periods and ranking losses by period.  The summarycalc losses for all events (all processes) must be written to the input folder before running leccalc.

##### Calculation

All files with extension .bin from the specified subdirectory are read into memory, as well as the occurrence.bin. The summarycalc losses are grouped together and sampled losses are assigned to period according to which period the events occur in.

If multiple events occur within a period;
* For **aggregate** loss exceedance curves, the sum of losses is assigned to the period.
* For **occurrence** loss exceedance curves, the maximum loss is assigned to the period.

Then the calculation differs by lec type, as follows;

* Full uncertainty - all sampled losses by period are rank ordered to produce a single loss exceedance curve
* Wheatsheaf - losses by period are rank ordered for each sample, which produces many loss exceedance curves - one for each sample
* Sample mean - the losses by period are first averaged across the samples, and then a single loss exceedance curve is created from the period sample mean losses.
* Wheatsheaf mean - the loss exceedance curves from the Wheatsheaf are averaged across each return period, which produces a single loss exceedance curve.

* For the Sample mean and Wheatsheaf mean curves, the analytical mean loss (sidx = -1) is kept separate and output as a separate exceedance probability curve.  If the calculation is run with 0 samples, then leccalc will still return the analytical mean loss exceedance curve.  The 'type' field in the output identifies the type of loss exceedance curve, which is 1 for analytical mean, and 2 for the mean calculated from the samples.

The total number of periods is carried in the header of the occurrence file.  The ranked losses represent the first, second, third (etc..) largest loss periods within the total number of periods of say 10,000 years.  The relative frequency of these periods of loss is interpreted as the probability of loss exceedance, that is to say that the top ranked loss has an exceedance probability of 1 in 10000, or 0.01%, the second largest loss has an exceedance probability of 0.02%, and so on.  In the output file, the exceedance probability is expressed as a return period, which is the reciprocal of the exceedance probability multiplied by the total number of periods.  Only non-zero loss periods are returned.

##### Output
csv file with the following fields;

###### Full uncertainty loss exceedance curve

| Name              | Type   |  Bytes | Description                                                         | Example     |
|:------------------|--------|--------| :-------------------------------------------------------------------|------------:|
| summary_id        | int    |    4   | summary_id representing a grouping of losses                        |   10        |
| return_period     | float  |    4   | return period interval                                              |    250      |
| loss              | float  |    4   | loss exceedance threshold for return period                         |    546577.8 |

###### Wheatsheaf exceedance curve;

| Name              | Type   |  Bytes | Description                                                         | Example     |
|:------------------|--------|--------| :-------------------------------------------------------------------|------------:|
| summary_id        | int    |    4   | summary_id representing a grouping of losses                        |   10        |
| sidx              | int    |    4   | Oasis sample index                                                  |    50       |
| return_period     | float  |    4   | return period interval                                              |    250      |
| loss              | float  |    4   | loss exceedance threshold for return period                         |    546577.8 |

###### Sample mean and Wheatsheaf mean exceedance curve;

| Name              | Type   |  Bytes | Description                                                         | Example     |
|:------------------|--------|--------| :-------------------------------------------------------------------|------------:|
| summary_id        | int    |    4   | summary_id representing a grouping of losses                        |   10        |
| type              | int    |    4   | 1 for analytical mean, 2 for sample mean                            |    2        |
| return_period     | float  |    4   | return period interval                                              |    250      |
| loss              | float  |    4   | loss exceedance threshold for return period                         |    546577.8 |

## pltcalc <a id="pltcalc"></a>

The program outputs sample mean and standard deviation by summary_id,  event_id and period_num.  It also outputs an event occurrence date.

##### Parameters

None

##### Usage
```
$ [stdin component] | pltcalc > plt.csv
$ pltcalc < [stdin].bin > plt.csv
```

##### Example
```
$ eve 1 1 1 | getmodel | gulcalc -S100 -C1 | summarycalc -1 - | pltcalc > plt.csv
$ pltcalc < summarycalc.bin > plt.csv 
```
##### Calculation

The occurrence.bin file is read into memory.  For each summary_id,  event_id and period_num, the sample mean and standard deviation is calculated from the sampled losses in the summarycalc stream and output to file.  The exposure_value, which is carried in the event_id, summary_id header of the stream is also output, as well as the date field(s) from the occurrence file.

##### Output
There are two output formats, depending on whether an event occurrence date is an integer offset to some base date that most external programs can interpret as a real date (to the nearest day), or a day in a numbered scenario year. The output format will depend on the format of the date fields in the occurrence.bin file.  

In the former case, the output format is; 

| Name              | Type   |  Bytes | Description                                                         | Example     |
|:------------------|--------|--------| :-------------------------------------------------------------------|------------:|
| event_id          | int    |    4   | Oasis event_id                                                      |  45567      |
| period_num        | int    |    4   | identifying an abstract period of time, such as a year              |  56876      |
| mean              | float  |    4   | sample mean                                                         |   1345.678  |
| standard_deviation| float  |    4   | sample standard deviation                                           |    945.89   | 
| exposure_value    | float  |    4   | exposure value for summary_id affected by the event                 |   70000     |
| date_id           | int    |    4   | the date_id of the event occurrence                                 |   28616     |

Using a base date of 1/1/1900 the integer 28616 is interpreted as 16/5/1978.

In the latter case, the output format is; 

| Name              | Type   |  Bytes | Description                                                         | Example     |
|:------------------|--------|--------| :-------------------------------------------------------------------|------------:|
| event_id          | int    |    4   | Oasis event_id                                                      |  45567      |
| period_num        | int    |    4   | identifying an abstract period of time, such as a year              |  56876      |
| mean              | float  |    4   | sample mean                                                         |   1345.678  |
| standard_deviation| float  |    4   | sample standard deviation                                           |    945.89   | 
| exposure_value    | float  |    4   | exposure value for summary_id affected by the event                 |   70000     |
| occ_year          | int    |    4   | the year number of the event occurrence                             |   56876     |
| occ_month         | int    |    4   | the month of the event occurrence                                   |   5         |
| occ_year          | int    |    4   | the day of the event occurrence                                     |   16        |

## aalcalc <a id="aalcalc"></a>

blah blah blah

##### Parameters

None

##### Usage
```
$ [stdin component] | aalcalc > aal.csv
$ aalcalc < [stdin].bin > aal.csv
```

##### Example
```
$ eve 1 1 1 | getmodel | gulcalc -S100 -C1 | summarycalc -g - | aalcalc > aal.csv
$ aalcalc < summarycalc.bin > aal.csv 
```

##### Output
Csv file with the following fields;

| Name              | Type   |  Bytes | Description                                                         | Example     |
|:------------------|--------|--------| :-------------------------------------------------------------------|------------:|
| summary_id        | int    |    4   | summary_id representing a grouping of losses                        |   10        |
| mean              | float  |    4   | average annual loss                                                 |    67856.9  |
| standard_deviation| float  |    4   | standard deviation of annual loss                                   |    546577.8 |

##### Parameters
There are no parameters as all of the information is taken from the gul stdout stream and fm input data.

##### Usage
```
$ [stdin component] | fmcalc | [stout component]
$ [stdin component] | fmcalc > [stdout].bin
$ fmcalc < [stdin].bin > [stdout].bin
```

##### Example
```
$ eve 1 1 | getmodel | gulcalc -r -S100 -i - | fmcalc | eltcalc > elt.csv
$ eve 1 1 | getmodel | gulcalc -r -S100 -i - | fmcalc > fmcalc.bin
$ fmcalc < gulcalc.bin > fmcalc.bin 
```

##### Internal data
The program requires the item, coverage and fm input data, which are Oasis abstract data formats that describe an insurance programme. This data is picked up from the following files relative to the working directory;

* input/items.bin
* input/coverages.bin
* input/fm_programme.bin
* input/fm_policytc.bin
* input/fm_profile.bin
* input/fm_xref.bin

##### Calculation
See [Financial Module](FinancialModule.md)

[Return to top](#referencemodel)
<a id="eltcalc"></a>
## outputcalc 
The reference example of an output produces an event loss table 'elt' for either ground up loss or insured losses.

##### Stream_id

There is no output stream_id, the results table is written directly to csv.

##### Parameters
There are no parameters as all of the information is taken from the input stream and internal data.

##### Usage
Either gulcalc or fmcalc stdout can be input streams to outputcalc
```
$ [stdin component] | outputcalc | [output].csv
$ outputcalc < [stdin].bin > [output].csv
```

##### Example
Either gulcalc or fmcalc stdout streams can be input streams to outputcalc. For example;
```
$ eve 1 1 1 | getmodel 1 | gulcalc -C1 -S100 | outputcalc > output.csv
$ eve 1 1 1 | getmodel 1 | gulcalc -C1 -S100 | fmcalc | outputcalc > output.csv
$ outputcalc < gul.bin > output.csv
$ outputcalc < fm.bin > output.csv
```

##### Internal data
The program requires the exposures.bin file, and for fmcalc input it requires an additional cross reference file to relate the output_id from the fm stdout stream (which represents an abstract grouping of exposure items) back to the original item_id. This file is picked up from the fm subdirectory;

fm/fmxref.bin

##### Calculation
The program sums the sampled losses from either gulcalc stream or fmstream across the portfolio/programme by event and sample, and then computes sample mean and standard deviation. It reads the TIVs from the exposure instance table and sums them for the group of items affected by each event.

[Return to top](#referencemodel)

[Go to Input data tools](Inputtools.md)

[Back to Contents](Contents.md)
