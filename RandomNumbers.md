# Random numbers <a id="randomnumbers"></a>

Simple uniform random numbers are assigned to each event, group and sample number to sample ground up loss in the gulcalc process. Groups are the method of supporting spatial correlation in ground up loss sampling in Oasis and ktools.

```
#### Correlation
A group is a collection of items which share the same group_id. This information is provided by the user in the exposure input file (exposures.bin).

Items (typically representing, in insurance terms, the underlying risk coverages) that are assigned the same group_id will use the same random number to sample damage for a given event and sample number. Items with different group_ids will be assigned independent random numbers.  Therefore sampled damage is fully correlated within groups and fully independent between groups, where group is an abstract collection of items and defined by the user.
```

The method of assigning random numbers in gulcalc uses an random number index (RIDX) which is used as a position reference into a list of random numbers.  N random numbers corresponding to the runtime number of samples are drawn from the list starting at the RIDX position.

### Options in ktools

There are three options in ktools for choosing random numbers to apply in the sampling process;

#### 1. Generate dynamically during the calculation

##### Usage
No random number parameters are required, this is the default.

##### Method

Random numbers are sampled dynamically using the Mersenne twister psuedo random number generator (the default RNG of the C++ v11 compiler). 
A buffer of 1 million random numbers is allocated to each event. The RIDX is generated from the group_id and sidx using the following modulus function;

RIDX=MOD(GROUP_ID*500,009 + SIDX*500,029, 1,000,000)

500,009 and 500,029 are the first two prime numbers which are greater than 500,000.  This formula pseudo-randomly assigns RIDX indexes to each GROUP_ID and SIDX combo between 0 and 999999. 

As the RIDX's are sampled, this position in the buffer is populated with random numbers unless they have already been populated, in which case the existing random numbers are reused.

The buffer is cleared for the next event and a new set of random numbers is generated.  

#### 2. Use numbers from random number file (gulcalc parameter -r)

##### Usage
gulcalc -r

##### Method
The random number file(s) is read into memory at the start of the gulcalc process. The same set of numbers are used for all events in the corresponding event chunk.

The RIDX is generated from the event_id and group_id using the following modulus function;

RIDX=MOD(EVENT_ID*P1+GROUP_ID*P2, D)

D is the divisor of the modulus, equal to the total number of random numbers in the list.
P1 and P2 are the first two prime numbers which are greater than half of D.

This formula pseudo-randomly assigns RIDX indexes to each EVENT_ID and GROUP_ID combo between 0 and D-1.

#### 3. Use numbers from random number file in 'reconciliation mode'

##### Usage
gulcalc -r -R

##### Method

Reconciliation mode uses the same method as 2. but calculates the RIDX in the same way as Oasis classic, thereby producing exactly the same losses for comparison.  

In Oasis classic random numbers are held in a tabular format, with the sample index in columns and the RIDX being a reference into a row of random numbers for a sample set of size N.  In gulcalc the divisor D is set to the total number of rows, given the number of columns N.  The number of columns N is the first 4 byte integer value in the random number file.

[Back to Contents](Contents.md)
