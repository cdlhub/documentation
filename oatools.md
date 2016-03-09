# oatools

oatools provides 2 additional components to ktools in order to implement the in-memory kernel as an Oasis calculation back-end:

* **gendata** : pulls required data from SQLServer or MySQL DB into binary files which can be used as input by ktools
* **bcp** : pushes binary GUL or FM files (outputs of gulcalc and fmcalc respectively) into a SQLServer DB, including table creation (will delete if table exists)  

oatools can be compiled in Linux only. ktools must also be compiled in the same environment.

### Release

Please click [here](https://github.com/OasisLMF/oatools/releases) to download the latest release. 

The source code will change on a regular basis but only the releases are supported. Support enquiries should be sent to support@oasislmf.org.

## Linux Installation

The installation involves compiling the code, and installing and configuring an odbc driver for the relevant database engine. The two supported database engines are SQL Server and MySQL (gendata only).

Lastly, it is necessary to configure two connection string text files which should be placed in the working directory (where the components are to be invoked).

### Compilation pre-requisites

The g++ compiler, make and libboost needs to be installed in Linux. This can be installed using the following command;

``` sh
sudo apt-get install g++ make libboost-dev-all
```

### Compilation instructions

Copy oatools-[version].tar.gz onto your machine and untar.
``` sh
$ tar -xvf oatools-[version].tar.gz
```

Go into the oatools folder and configure using the following command;
``` sh
$ cd oatools-[version]
$ ./configure
```

Make using the following command;
``` sh
$ make
```

Finally, install the executables using the following command;
``` sh
$ make install
```

The installation is complete. The executables are located in ~/usr/local/bin. 

If installing the latest code from the git repository, clone the oatools repository onto your machine.

Go into the oatools folder and configure using the following command;
``` sh
$ cd oatools
$ ./kconfigure
```
Follow the rest of the process as described above.

### ODBC driver configuration

To use the components, it is necessary to configure a database odbc driver.  

#### SQL Server odbc configuration

The supported SQL Server odbc driver for Unix is freeTDS which needs to be installed locally. This can be installed using the following command;

``` sh
sudo apt-get install unixodbc unixodbc-dev freetds-dev tdsodbc
```
To complete configuration, do the following.

Append to /etc/odbc.ini:
``` sh
[freetds] 
Description = “freetds” 
Driver = freetds
```
Append to /etc/odbcinst.ini (ensure .so files are in the directories specified – modify as required):

``` sh
[FreeTDS] 
Description   = FreeTDS 
Driver        = /usr/lib/x86_64-linux-gnu/odbc/libtdsodbc.so 
Setup         = /usr/lib/x86_64-linux-gnu/odbc/libtdsS.so
UsageCount    = 1
Threading     = 1
```

#### MySQL configuration

TO DO

### Connection string files

gendata requires **odbcconnect.txt** which contains:

1,DRIVER={FreeTDS};Server=10.0.0.36;Database=master;UID=sa;PWD=myPassword;TDS_Version=8.0;Port=1433;

The first field is 1, for SQLServer or 2, for MySQL.

Enter the IP address of the database server along with the relevant login credentials. 

TDS_Version=8.0;Port=1433 is fixed.

bcp requires **bcpconnectionstring.txt** which contains:

10.0.0.36;sa;myPassword;OASIS_C1_RESULTS;

The fields are: IP;userNameForDB;passwordForDB;DBName

### Questions/problems?

Email support@oasislmf.org
