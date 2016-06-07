# Outputs <a id="outputs"></a>

ktools supports the following output calculation components;

* **[summarycalc](#summarycalc)** returns ground up or insured loss samples by summary_id. 

summarycalc is the input to the following output calculation components.

* **[eltcalc](#eltcalc)** outputs event loss tables 'elt' to a csv. 
* **[leccalc](#leccalc)** outputs loss exceedance curves 'lec' to a csv. 
* **[pltcalc](#pltcalc)** outputs period loss tables 'plt' to a csv.
* **[aalcalc](#aalcalc)** outputs average annual losses 'aal' to a csv.
* **[elhcalc](#elhcalc)** outputs event loss histograms 'elh' to a csv. Future.

##### Stdin stream_id

summarycalc takes either gulcalc standard output or fm standard output as a streamed input, or a binary file of the same format can be piped in.

| Byte 1 | Bytes 2-4 |  Description                  |
|:-------|-----------|:------------------------------|
|    1   |     2     |  gulcalc coverage stdout      |
|    2   |     1     |  fmcalc stdout                |

The other output components take summarycalc standard output as a streamed input, or a binary file of the same format can be piped in.

| Byte 1 | Bytes 2-4 |  Description         |
|:-------|-----------|:---------------------|
|    3   |     1     |  summarycalc stdout  |

A binary file of the same format as summarycalc standard output can be piped into the output components.

## summarycalc <a id="summarycalc"></a>

##### Parameters

The input stream should be identified explicitly as -g for gulcalc stream or -f for fmcalc stream.
summarycalc supports up to 10 concurrent outputs.  This is achieved by explictly directing each output to a named pipe, file, or to standard output.  
For each output stream, the following tuple of parameters must be specified for at least one summary set;

* summaryset_id number. valid values are -0 to -9
* destination pipe or file. Use either - for standard output, or -[name] for a named pipe or file.

For example the following parameter choices are valid;
```
$ summarycalc -g -1 -                                       'outputs summaryset 1 results to standard output
$ summarycalc -g -1 summarycalc1.bin                        'outputs summaryset 1 results to a file (or named pipe)
$ summarycalc -g -1 summarycalc1.bin -2 summarycalc2.bin    'outputs summaryset 1 and 2 results to a file (or named pipe)
```
Note that the summaryset_id relates to a summaryset_id in the required input data file **gulsummaryxref.bin** or **fmsummaryxref.bin** for a gulcalc input stream or a fmcalc input stream, respectively, and represents a user specified summary reporting level, for example site, zipcode or portfolio.

##### Usage
```
$ [stdin component] | summarycalc [parameters] | [stdout component]
$ [stdin component] | summarycalc [parameters] > [stdout].bin
$ summarycalc [parameters] < [stdin].bin > [stdout].bin
```

##### Example

Single output
```
$ eve 1 1 | getmodel | gulcalc -S100 -c - | summarycalc -g -1 - | eltcalc > eltcalc.csv
$ eve 1 1 | getmodel | gulcalc -S100 -c - | summarycalc -g -1 > gulsummarycalc.bin 
$ summarycalc -f -1 fmsummarycalc.bin < fmcalc.bin
```

Multiple outputs
```
eltcalc < pipe1 > eltcalc1.csv
eltcalc < pipe2 > eltcalc2.csv
eltcalc < pipe3 > eltcalc3.csv
eve 1 1 | getmodel | gulcalc -S100 -c - | summarycalc -g -1 pipe1 -2 pipe2 -3 pipe3
```

## eltcalc <a id="eltcalc"></a>

blah blah blah

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

##### Output
Csv file with the following fields;

| Name              | Type   |  Bytes | Description                                                         | Example     |
|:------------------|--------|--------| :-------------------------------------------------------------------|------------:|
| summary_id        | int    |    4   | summary_id representing a grouping of losses                        |   10        |
| event_id          | int    |    4   | Oasis event_id                                                      |  45567      |
| mean              | float  |    4   | sample mean                                                         |   1345.678  |
| standard_deviation| float  |    4   | sample standard deviation                                           |    945.89   | 
| exposure_value    | float  |    4   | sum of TIV for summary_id affected by event_id                      |   70000     |


[Return to top](#outputs)

## leccalc <a id="leccalc"></a>

blah blah blah

##### Parameters
* -K the subdirectory containing the input summarycalc binary files.
The following tuple of parameters must be specified for at least one analysis type;
* Analysis type. Use -F for Full Uncertainty Aggregate, -f for Full Uncertainty Occurrence, -W for Wheatsheaf Aggregate,  -w for Wheatsheaf Occurrence, -S for Sample Mean Aggregate, -s for Sample Mean Occurrence, -M for Mean of Wheatsheaf Aggregate, -m for Mean of Wheatsheaf Occurrence
* Output filename (csv)
* Return period (optional). Use -r if you are providing a file with a specific list of return periods. If this is not present then all calculated return periods will be returned, for losses greater than zero. 

##### Usage
```
$ leccalc [parameters] > lec.csv

```

##### Example
```
$ eve 1 1 1 | getmodel | gulcalc -S100 -C1 | summarycalc -1 - | leccalc -A -F -r > lec.csv
$ leccalc -O -M < summarycalc.bin > lec.csv 
```

##### Output
csv file with the following fields;

| Name              | Type   |  Bytes | Description                                                         | Example     |
|:------------------|--------|--------| :-------------------------------------------------------------------|------------:|
| summary_id        | int    |    4   | summary_id representing a grouping of losses                        |   10        |
| return_period     | float  |    4   | return period interval                                              |    250      |
| loss              | float  |    4   | loss exceedance threshold for return period                         |    546577.8 |

If -W Wheatsheaf output is specified, the sample index is output as an additional column;

| Name              | Type   |  Bytes | Description                                                         | Example     |
|:------------------|--------|--------| :-------------------------------------------------------------------|------------:|
| summary_id        | int    |    4   | summary_id representing a grouping of losses                        |   10        |
| sidx              | int    |    4   | Oasis sample index                                                  |    100      |
| return_period     | float  |    4   | return period interval                                              |    250      |
| loss              | float  |    4   | loss exceedance threshold for return period                         |    546577.8 |

## pltcalc <a id="pltcalc"></a>

blah blah blah

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

##### Output
csv file with the following fields;

| Name              | Type   |  Bytes | Description                                                         | Example     |
|:------------------|--------|--------| :-------------------------------------------------------------------|------------:|
| occurrence_id     | int    |    4   | identifier of a distinct occurrence of an event_id                  |   10        |
| event_id          | int    |    4   | Oasis event_id                                                      |  45567      |
| period_num        | int    |    4   | identifying an abstract period of time, such as a year              |  876        |
| mean              | float  |    4   | sample mean                                                         |   1345.678  |
| standard_deviation| float  |    4   | sample standard deviation                                           |    945.89   | 
| exposure_value    | float  |    4   | sum of TIV for summary_id affected by the event                     |   70000     |

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

## elhcalc <a id="elhcalc"></a>

To be defined.

[Return to top](#outputs)


