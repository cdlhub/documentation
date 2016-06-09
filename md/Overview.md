# Data Streaming Framework Overview

This is the general data streaming framework showing the main components of the toolkit.

##### Figure 1. Data streaming framework
![alt text](../img/DataStreamingFramework.jpg "Data Streaming Framework")

The architecture consists of;

* A **model system** which provides the model data in the correct binary format.  This is a one time set-up job.
* A **user front end**, which provides the users data for an analysis against the model.
* A **compute server**, where the computations are performed and the required model data is held in a 'static' folder, and the required user data is held in an 'input' folder.

The binary conversion of input data can be performed either inside or outside the compute server. ktools provides a full set of binary conversion tools from csv input files.

The in-memory data streams are initiated by the process 'eve' (meaning 'event emitter') and shown by solid arrows. The read/write data flows are shown as dashed arrows. Multiple arrows mean multiple processes. 

The calculation components are *getmodel*, *gulcalc*, *fmcalc*, *summarycalc* and *outputcalc*. The analysis data passes through the components in memory one event at a time and written out to an output file on the compute server.  The user can then retrieve the results (csvs) and consume them in their BI system.

The reference model demonstrates an implementation of the principal calculation components, along with the data conversion components which convert binary files to csv files. 

The analysis workflows are controlled by the user, not the toolkit, and they can be as simple or as complex as required.

The simplest workflow is straight through in-memory processing to produce a single output file.  This minimises the amount of disk I/O at each stage and results in the best performance. This workflow is shown in Figure 2.

##### Figure 2. Straight-through processing
![alt text](../img/SingleOutput.jpg "Straight-through processing")

However it is possible to stream data from one process into to several processes, allowing the calculation of multiple outputs simultaneously, as shown in Figure 3.

##### Figure 3. Multiple output file processing
![alt text](../img/MultipleOutput1.jpg "Multiple output file processing")

For multi-output, multi-process workflows, the Linux operating system provides functionality such as 'named pipes' which in-memory data streams can be diverted to and manipulated as if they were files, and 'tee' which sends a stream from one process into multiple processes.  Some examples scripts for multi-process workflows are provided in the examples folder. Also, see **Multiplexing in Linux**.


[Go to Specification](Specification.md)

[Back to Contents](Contents.md)
