# Introduction
The in-memory solution for the Oasis Kernel is called the kernel tools or “ktools”. ktools is an independent “specification” of a set of processes which means that it defines the processing architecture and data structures and is then implemented as “reference model” which can then be adapted for particular model or business needs. 

The code can be compiled in Linux, POSIX-compliant Windows and native Windows.

### Background
The Kernel performs the core Oasis calculations of computing effective damageability distributions, Monte-Carlo sampling of ground up loss, the financial module, which applies insurance policy terms and conditions to the sampled losses, and some common catastrophe model outputs.

The first releases of Oasis used SQL-based calculation back-end with all calculations being performed in a database.  Release 1.3  included the first “in-memory” version of the Oasis Kernel written in C++ and C to provide streamed calculation at high computational performance, as an alternative to the SQL-based calculation back-end for the ground up loss sampling and financial module calculations. The in-memory variant was first delivered as a stand-alone toolkit "ktools" with R1.4. 

With R1.5, a Linux-based in-memory calculation back-end was deployed, using the reference model components of ktools. The range of functionality of ktools was limited to ground up loss sampling and the financial module and single output workflows, with effective damage distributions and output calculations still being performed in a SQL-compliant database.

In 2016 the functionality of ktools was extended to include the full range of Kernel calculations, including effective damageability distribution calculations and a wider range of financial module and output calculations.  The data stream structures and input data formats were also substantially revised to handle multi-peril models, user-definable summary levels for outputs, and multiple output workflows.  

### Architecture

The Kernel is provided as a toolkit of components (“ktools”) which can be invoked at the command line.  Each component is a separately compiled executable with a binary data stream of inputs and outputs.  
The principle is to stream data through by event end-to-end, with multiple processes being used either sequentially or concurrently, at the control of the user using a script file appropriate to the operating system.

### Language

The components can be written in any language as long as the data structures of the binary streams are adhered to.  The current set of components have been written in POSIX-compliant C++ and C.  This means that they can be compiled in Linux and Windows using the latest GNU compiler toolchain.

### Components

The components in the Reference Model can be summarized as follows;

* **[Core components](CoreComponents.md)** perform the core kernel calculations and are required in the main analysis workflow.
* **[Output components](OutputComponents.md)** perform specific output calculations at the end of the analysis workflow. They are essentially various summaries of the sampled losses from the core calculation
* **[Data conversion components](InputConversionComponents.md)** are provided to assist users get model and exposure data into the required binary formats
* **[Stream conversion components](StreamConversionComponents.md)** are provided to inspect the data stream flowing between the core components, for debugging or more detailed analysis.
 
### Usage

Standard piping syntax can be used to invoke the components at the command line. For example the following command invokes eve, getmodel, gulcalc, fmcalc, and outputcalc, and exports the results to a csv file.
``` sh
$ eve 1 2 | getmodel | gulcalc –S100 –i - | fmcalc | eltcalc > elt.csv
```

Example python scripts are provided along with a binary data package in the /examples folder to demonstrate usage of the toolkit. For more guidance on how to use the toolkit, see [Workflows](Workflows.md)

[Go to Data streaming architecture overview](Overview.md)

[Back to Contents](Contents.md)
