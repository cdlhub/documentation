# Financial Module

The Oasis Financial Module is a data-driven process design for calculating the losses on insurance policies. It has an abstract design in order to cater for the many variations in source policy structures and conditions. The way Oasis works is to be fed data in order to execute calculations and so for the insurance calculations it needs to know the structure, parameters and calculation rules to be used. This data must be provided in the files used by the Oasis Financial Module:

* fm_programme
* fm_profile
* fm_policytc 

This section explains the design of the Financial Module which has been implemented in the **fmcalc** component. The formats of the input files are covered in Input Data. Runtime parameters and usage instructions for fmcalc are covered in Reference Model section.

Note that other reference tables are referred to below do not appear explicitly in the kernel as they are not directly required for calculation.  It is expected that a front end system will hold all of the exposure and policy data and generate the above three input files required for the kernel calculation.

## Scope

The Financial Module outputs sample by sample losses by (re)insurance policy/contract, or by item, which represents the individual coverage subject to economic loss. In the latter case, it is necessary to ‘back-allocate’ losses when they are calculated at a higher policy level.  The Financial Module does not, at present, output retained loss or ultimate net loss (UNL) perspectives. It does, though, allow the user to output losses at any stage of the calculation.

The output contains anonymous keys representing the (re)insurance programme (PROG_ID) and policy (LAYER_ID) at the chosen output level (OUTPUT_ID) and a LOSS value. Losses by sample number (IDX) and event (EVENT_ID) are produced.  To make sense of the information, this output must be cross-referenced with Oasis dictionaries containing the information to make sense of the keys.

## Profiles

Profiles are used throughout the Oasis framework and are meta-data definitions with their associated data types and rules.  Profiles are used in the Financial Module to perform the elements of financial calculations used to calculate losses to (re)insurance policies.  For anything other than the most simple policy which has only a deductible and limit, say, a Profile do not represent a policy structure on its own, but rather it should be considered as a building block which can be combined with other building blocks to model a particular financial contract. In this way it is possible to model an unlimited range of structures with a limited number of Profiles.

The FM Profiles form an extensible library of calculations defined within the fmcalc code that can be invoked by specifying a particular CALCRULE_ID and providing the required data values such as deductible and limit, as described below.

The Profiles currently supported are as follows;

#### Supported Profiles

| Profile Description                                                | ProfileID| CALCRULE_ID |
|:-------------------------------------------------------------------|----------|------------:|
| Deductible and limit                                               | 1        |   1         |
| Franchise deductible and limit                                     | 2        |   3         |
| Deductible only                                                    | 3        |  12         |
| Deductible as a cap on the retention of input losses               | 4        |  10         |
| Deductible as a floor on the retention of input losses             | 5        |  11         |
| Deductible, limit and share                                        | 6        |   2         |
| Deductible and limit as a proportion of loss                       | 10       |   5         |
| Limit with deductible as a proportion of limit                     | 11       |   9         |
| Limit only                                                         | 12       |   14        |
| Limit as a proportion of loss                                      | 13       |   15        |
| Deductible as a proportion of loss                                 | 14       |   16        |


## Design

The Oasis Financial Module is a data-driven process design for calculating the losses on insurance policies. Like the Exposure Module, it is an abstract design in order to cater for the many variations and has three basic concepts:

1. A **Programme** which defines which ITEMs are grouped together at which levels for the purpose of providing loss amounts to policy terms and conditions. The Programme has a user-definable Profile and Dictionary called **Prog** which holds optional reference details such as a Description and Account identifier. The Prog table is not required for the calculation and therefore does not appear in the kernel input files.
2. A PolicyTC **Profile** which provides the parameters of the policy’s terms and conditions such as limit and deductible and calculation rules.
3. A **PolicyTC** cross-reference file which associates a policy terms and Conditions Profile to each Programme Item level aggregation.

A PolicyTC Profile not only provides the fields to be used in calculating losses (such as limit and deductible) but also which mathematical calculation (CALCRULE_ID) and which allocation rule (ALLOCRULE_ID) to apply.

## Data requirements

The Financial Module brings together three elements in order to undertake a calculation:
* Structural information, notably which Items are protected by a (set of) policies.
* Loss values of items.
* Policy Profiles and Profile values.

There are many ways an insurance loss can be calculated with many different terms and conditions. For instance, there may be deductibles applied to each element of coverage (e.g. a buildings damage deductible), some site-specific deductibles or limits, and some overall policy deductibles and limits and line share. To undertake the calculation in the correct order and using the correct items (and their values) requires a file defining the structure and sequence of calculations. 

This is the Programme file which defines a heirarchy of groups across a number of LEVELs.  LEVELs drive the sequence of calculation. A financial calculation is performed at successive LEVELs, depending on the structure of policy terms and conditions. For example there might be 3 LEVELs representing coverage, site and policy terms and conditions. 

Groups are defined within LEVELs and they represent aggregations of losses on which to perform the financial calculations.  The grouping fields are called FROM_AGG_ID and TO_AGG_ID which represent a grouping of losses at the previous level and the present level of the heirarchy, respectively.  

### Loss values
The initial input is the Ground-up Loss (GUL) table coming from the main Oasis calculation of ground-up losses. Here is an example, for a single event and sample (IDX=1):

| EVENT_ID | ITEM_ID  | IDX    | GUL    |
|:---------|----------|--------| ------:|
|       1  | 1        |    1   | 100,000|
|       1  | 2        |    1   | 10,000 |
|       1  | 3        |    1   | 2,500  |
|       1  | 4        |    1   | 400    |

The values represent a single ground-up loss sample for items belonging to an Account. We use “Programme” rather than Account as it is more general characteristic of a client’s exposure protection needs and allows a client to have multiple Programmes active for a given period.
The linkage between Account and Programme can be provided by a user defined **Prog** Dictionary, for example;

| PROG_ID  | ACCOUNT_ID  | PROG_NAME                     |
|:---------|-------------|------------------------------:|
|       1  | 1           | ABC Insurance Co. 2016 renewal|

Items 1-4 represent Structure, Other Structure, Contents and Time Element coverage ground up losses for a single property, respectively, and this example is a simple residential policy with combined property coverage terms. For this policy type, the Structure, Other Structure and Contents losses are aggregated, and a deductible and limit is applied to the total. A separate set of terms, again a simple deductible and limit, is applied to the “time element” coverage which, for residential policies, generally means costs for temporary accommodation. The total insured loss is the sum of the output from the combined property terms and the time element terms.

### Programme

The actual Items falling into the Programme are specified in the **Programme** table together with the aggregation groupings that go into a given LEVEL calculation:

| PROG _ID | FROM_AGG_ID | LEVEL_ID| TO_AGG_ID |
|:---------|----------|---------| ------:|
|       1  | 1        |     1   | 1      |
|       1  | 2        |     1   | 1      |
|       1  | 3        |     1   | 1      |
|       1  | 4        |     1   | 2      |
|       1  | 1        |     2   | 1      |
|       1  | 2        |     2   | 1      |

Note that FROM_AGG_ID for LEVEL_ID=1 is equal to the ITEM_ID in the input loss table (but in theory FROM_AGG_ID could represent a higher level of grouping, if required). 

In LEVEL 1, Items 1, 2 and 3 all have TO_AGG_ID =1 so losses will be summed together before applying the combined deductible and limit, but Item 4 (time) will be treated separately (not aggregated) as it has TO_AGG_ID = 2. For LEVEL 2 we have all 4 Items losses (now represented by two groups FROM_AGG_ID =1 and 2 from the previous level) aggregated together as they have the same TO_AGG_ID (value 1 in this case).

### Profile

Next we have the Profile Description table, which list the profiles representing general policy types. Our example is represented by two general Profiles which specify the input fields and mathematical operations to perform. In this example, the Profile for the combined coverages and time is the same (albeit with different values) and requires a limit, a deductible, and an associated calculation rule, whereas the Profile for the policy requires a limit, deductible, and share, and an associated calculation rule.

| Profile Description                                                | ProfileID| CALCRULE_ID |
|:-------------------------------------------------------------------|----------|------------:|
| Deductible and limit                                               | 1        |   1         |
| Deductible, limit and share                                        | 6        |   2         |


There is a “Profile Value” table for each Profile containing the applicable policy terms, each identified by a POLICYTC_ID. The table below shows the list of policy terms for ProfileID 1.

| POLICYTC_ID | CCY_ID | LIM       | DED   | CALCRULE_ID |
|:------------|--------|-----------| ------|------------:|
|       1     | 1      | 1,000,000 | 1,000 |     1       |
|       2     | 1      |    18,000 | 2,000 |     1       |

And next, for Profile 6, the values for the overall policy deductible, limit and share

| POLICYTC_ID | CCY_ID | LIM       | DED   | SHARE  | CALCRULE_ID |
|:------------|--------|-----------| ------|--------|------------:|
|       3     | 1      | 1,000,000 | 1,000 |    0.1 |     2       |

In practice, all Profile values are stored in a single flattened format which contains all supported profile fields, but conceptually they belong in separate Profile Value tables. The actual **Profile** table looks like this;

| POLICYTC_ID | CALCRULE_ID | ALLOCRULE_ID | SOURCERULE_ID | LEVELRULE_ID | CCY_ID | DED  | LIM     |  SHARE  | SHARE_PROP_OF_LIM | DEDUCTIBLE_PROP_OF_LOSS | LIMIT_PROP_OF_LOSS | DEDUCTIBLE_PROP_OF_TIV | LIMIT_PROP_OF_TIV | DEDUCTIBLE_PROP_OF_TIV | LIMIT_PROP_OF_TIV | DEDUCTIBLE_PROP_OF_LIMIT |
|:------------|-------------|--------------|---------------|--------------|--------|------|---------|---------|-------------------|---------------------------|--------------------|------------------------|-------------------|------------------------|------------------|-------------------------:|
|      1      | 1           |       -1     |        0      |       0      |    1   | 1,000|1,000,000|   0     |        0          |          0                |         0          |           0            |        0          |     0                  |   0              |         0                |

### Policytc
The **Policytc** table specifies the insurance policies (a policy in Oasis FM is a combination of PROG_ID and LAYER_ID, there is no separate Policy dictionary required in the calculation) and the separate terms and conditions which will be applied to each LAYER_ID/AGG_ID for a given LEVEL. In our example, we have a limit and deductible with the same value applicable to the combination of the first three Items a limit and deductible for the fourth Item (time) in LEVEL 1, and then a limit, deductible, and line applicable at LEVEL 2 covering all Items. We’d represent this in terms of the distinct AGG_IDs (correspas follows:

| PROG_ID | LAYER_ID | LEVEL_ID | AGG_ID   | POLICYTC_ID |
|:--------|----------|----------| ---------|------------:|
|    1    |    1     |     1    |    1     |    1        |
|    1    |    1     |     1    |    2     |    2        |
|    1    |    1     |     2    |    1     |    3        |

In words, the data in the table means;

At Level 1;

Apply POLICYTC_ID (calculation rule) 1 to (the sum of losses grouped by) AGG_ID 1

Apply POLICYTC_ID 2 to AGG_ID 2

Then at Level 2;

Apply POLICYTC_ID 3 to AGG_ID 1

LEVELs are processed in ascending order and the calculated losses from a previous level are summed according to the groupings defined in the Programme table which become the input losses to the next LEVEL.

For any given Profile we have four standard fields:
* CALCRULE_ID, being the Function used to calculate the losses from the given Profile’s fields. There list of functions are shown below.
* ALLOCRULE_ID, being the rule for allocating back to ITEM level. There are really only two meaningful values here – don’t allocate (0) used typically for the final level to avoid maintaining lots of detailed losses, or allocate back to ITEMs (1) used in all other cases which is in proportion to the input ground up losses.
(Allocation does not need to be on this basis, by the way, there could be other rules such as allocate back always on TIV or in proportion to the input losses from the previous level, but we have implemented a ground up loss back-allocation rule.
* SOURCERULE_ID (not currently used), which is used for conditional logic if TIV (for example) needs to be used in a calculation.
* LEVELRULE_ID (not currently used) used for processing level-specific rules such as “special conditions”.

### Programme table rules
1. All fields have integer values
2. Must have at least one level. LEVEL_ID = 1, 2, 3 ...
2. For LEVEL_ID = 1, the FROM_AGG_ID must be equal to the ITEM_ID which cross references to the input ground up loss stream (which has fields EVENT_ID, ITEM_ID, IDX, GUL).  Therefore Level 1 always represents a group of items.
3. For subsequent levels, the FROM_AGG_ID must be the distinct values from the previous level TO_AGG_ID field.
4. Each programme table may only have a single integer value in PROG_ID. Note that this field is a cross-reference to the PROG dictionary and business meaningful information such as account number, and is not currently used in calculations.  This field may be deprecated in future versions.
5. The FROM_AGG_ID and TO_AGG_ID values, for each level, should be a contiguous block of integers (a sequence with no gaps).  This is not a strict rule in this version and it will work with non-contiguous integers, but it is recommended as best practice.

### Policytc table rules
1. All fields have integer values
2. Must have at least one level. LEVEL_ID = 1, 2, 3 ...
3. For every distinct combination of TO_AGG_ID and LEVEL_ID in the Programme table, there must be a corresponding record in the policytc table with a valid value in the POLICYTC_ID field.  If no calculation rule applies (eg a site without any terms) then a default 'null' rule must be used and specified in the profile table.
4. LAYER_ID = 1 at all LEVELs except the last where there may be more layers. LAYED_ID = 1, 2, 3 ... This allows for the specification of several policy contracts applied to the same aggregation of losses defined by the programme table.

### Profile table rules
1. All distinct POLICYTC_ID values that appear in the Policytc table must appear once in the POLICYTC_ID field in the profile table, with the relevant fields for the profile populated. We suggest that POLICYTC_ID=1 is included by default using CALCRULE_ID = 12 and DED = 0 (ProfileID 3 Deductible Only) as a default 'null' calculation rule whenever no terms and conditions apply to a particular LEVEL_ID/AGG_ID.
2. All data fields that are required by the relevant Profile must be provided.  These are specified in the Appendix.
3. Any fields that are not used should be set to zero.
4. The correct CALCRULE_ID should be specified for each POLICYTC_ID
5. ALLOCRULE_ID may be set to 0 or 1 for each POLICYTC_ID.  Generally, it is recommended to use 0 everywhere except for the final level calculations where back-allocated losses are required.

Note that the fields not currently used at all are CCY_ID, LEVELRULE_ID, CALCRULE_ID

## Back-allocation

In R1.1 of Oasis we took the view that it was simpler throughout to refer back to the base ITEMs rather than use a hierarchy of aggregated loss calculations. So, for example, we could have calculated a loss at the site level and then used this calculated loss directly at the policy conditions level but instead we allocated back to ITEM level and then re-aggregated to the next LEVEL. The reason for this was that intermediate conditions may only apply to certain ITEMs so if we didn’t go back to the base ITEM “ground-up” level then any higher LEVEL could have a complicated grouping of a LEVEL. 

In the implementation this required back-allocating losses to item at every level in a multi-level calculation even the next level calculaton did not require it.  In fact, the insurance policy structures provided as worked examples by Oasis members were simple heirarchies.   

The default execution path now supports only simple heirarchies but we may extend the functionality to handle both execution paths, if required.

