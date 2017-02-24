# acclocconv

A Windows utility to convert tab delimited acc and loc files as exported from RMS software into utf8 csv format required by Oasis Flamingo.

### Usage instructions

1. Download acclocconvert folder and save onto a drive where you have administrative privileges to run executables.
2. Copy one loc file and one acc file into acclocconvert folder and rename them as loc.txt and acc.txt
3. Run/double click **go.bat**
4. View output files. These are loc_out.csv and acc_out.csv 

### Contents of folder
* acclocconv.exe - The main program
* NDesk.Options.dll - A required dll

All of the following files are user-configurable. 

* go.bat - A Windows batch file which runs the main programme with a set of input parameters defined
* required_loc_fields.csv - Defines which fields should be extracted from the input loc file
* required_acc_fields.csv - Defines which fields should be extracted from the input acc file
* policy_type.csv - Defines the value of POLICY_TYPE in the acc file to filter records by (related to peril)
* baseccy.csv - The target/base currency that all monetary values will be converted to. (PLACEHOLDER - NOT USED YET)
* fx.csv - The currency exchange rates into the target currency. (PLACEHOLDER - NOT USED YET)
* loc_value_fields.csv - Defines the currency field and monetary value fields in the input loc file to apply currency conversion to. (PLACEHOLDER - NOT USED YET)
* acc_value_fields.csv - Defines the currency field and monetary value fields in the input acc file to apply currency conversion to. (PLACEHOLDER - NOT USED YET)


By configuring these files you can specify;
* Input and output filenames, and batch multiple commands.
* Which loc and acc fields to extract, as required by the canonical format of exposures for the model in Flamingo
* Which policy_type records to extract from acc file
* Which monetary value fields to apply currency conversion to (IN FUTURE VERSIONS)
* Which currency to convert monetary values fields into and at what FX rates (IN FUTURE VERSIONS)
