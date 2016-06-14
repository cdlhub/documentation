# Output Components <a id="outputcomponents"></a>

## eltcalc <a id="eltcalc"></a>

The program calculats sample mean and standard deviation of loss by summary_id and by event_id.

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

Loss exceedance curves, also known as exceedance probability curves, are computed by a rank ordering a set of losses by period and computing the probability of exceedance for each level of loss in any given period based on relative frequency. Period losses are first computed by reference to the **occurrence** file which contains the event occurrences against each period. Event losses are summed within each period for an aggregate loss exceedance curve, or the maximum of the event losses in each period is taken for an occurrence loss exceedance curve.  From this point, there are a few variants available as follows;

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

The user must ensure the work subdirectory exists.  The user may also specify a subdirectory of /work to store these files. e.g.

* work/summaryset1/summarycalc1.bin
* work/summaryset1/summarycalc2.bin
* work/summaryset1/summarycalc3.bin

The reason for leccalc not having an input stream is that the calculation is not valid on a subset of events, i.e. within a single process when distributing events across multiple processes.  It must bring together all event losses before assigning event losses to periods and ranking losses by period.  The summarycalc losses for all events (all processes) must be written to the /work folder before running leccalc.

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

* For the Sample mean and Wheatsheaf mean curves, the analytical mean loss (sidx = -1) is output as a separate exceedance probability curve.  If the calculation is run with 0 samples, then leccalc will still return the analytical mean loss exceedance curve.  The 'type' field in the output identifies the type of loss exceedance curve, which is 1 for analytical mean, and 2 for the mean calculated from the samples.

The total number of periods is carried in the header of the occurrence file.  The ranked losses represent the first, second, third,  etc.. largest loss periods within the total number of periods of say 10,000 years.  The relative frequency of these periods of loss is interpreted as the probability of loss exceedance, that is to say that the top ranked loss has an exceedance probability of 1 in 10000, or 0.01%, the second largest loss has an exceedance probability of 0.02%, and so on.  In the output file, the exceedance probability is expressed as a return period, which is the reciprocal of the exceedance probability multiplied by the total number of periods.  Only non-zero loss periods are returned.

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
There are two output formats, depending on whether an event occurrence date is an integer offset to some base date that most external programs can interpret as a real date, or a day in a numbered scenario year. The output format will depend on the format of the date fields in the occurrence.bin file.  

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

aalcalc computes the average annual loss and standard deviation of loss for each summary_id. Technically, this is calculated on a period by period basis because aalcalc, like leccalc and pltcalc, uses the occurrence file in which the periods of event occurrence are defined, and these are not necessarily annual periods.

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
| type              | int    |    4   | 1 for analytical mean, 2 for mean calculated from samples           |    1        |
| mean              | float  |    4   | average annual loss                                                 |    67856.9  |
| standard_deviation| float  |    4   | standard deviation of annual loss                                   |    546577.8 |
| exposure_value    | float  |    4   | maximum exposure value across all periods                           |    10098730 |

##### Parameters

None

##### Usage
```
$ [stdin component] | aalcalc | [stout component]
$ [stdin component] | aalcalc > [stdout].bin
$ fmcalc < [stdin].bin > [stdout].bin
```

##### Example
```
$ eve 1 1 | getmodel | gulcalc -r -S100 -c - | summarycalc -g | aalcalc > elt.csv
$ eve 1 1 | getmodel | gulcalc -r -S100 -i - | fmcalc > fmcalc.bin
$ fmcalc < gulcalc.bin > fmcalc.bin 
```

##### Internal data
The program requires the occurrence file;

* static/occurrence.bin

##### Calculation
The occurrence file is read into memory.

[Return to top](#outputcomponents)

[Go to Data conversion components section](DataConversionComponents.md)

[Back to Contents](Contents.md)
