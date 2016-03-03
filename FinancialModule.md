# Financial Module

The Oasis Financial Module is a data-driven process design for calculating the losses on insurance policies. It has an abstract design in order to cater for the many variations in source policy structures and conditions. The way Oasis works is to be fed data in order to execute calculations and so for the insurance calculations it needs to know the structure, parameters and calculation rules to be used. It is these data which are uploaded in the files used by the Oasis Financial Module.

## Design

The Oasis Financial Module is a data-driven process design for calculating the losses on insurance policies. Like the Exposure Module, it is an abstract design in order to cater for the many variations and has three basic concepts:

1. A **Programme** which defines which ITEMs are grouped together at which levels for the purpose of providing loss amounts to policy terms and conditions. The Programme has a user-definable Profile and Dictionary called **Prog** which holds optional reference details such as a Description and Account identifier.
2. A PolicyTC **Profile** which provides the parameters of the policy’s terms and conditions such as limit and deductible and calculation rules.
3. A **PolicyTC** cross-reference file which associates a policy terms and Conditions Profile to each Programme Item level aggregation.

A PolicyTC Profile not only provides the fields to be used in calculating losses (such as limit and deductible) but also which mathematical calculation (CALCRULE_ID) and which allocation rule (ALLOCRULE_ID) to apply.

## Data requirements

The Financial Module brings together three elements in order to undertake a calculation:
* Structural information, notably which Items are protected by a (set of) policies.
* Loss values of items.
* Policy Profiles and Profile values.
 
There are many ways an insurance loss can be calculated with many different terms and conditions. For instance, there may be deductibles applied to each element of coverage (e.g. a buildings damage deductible), some site-specific deductibles or limits, and some overall policy deductibles and limits and line share. To undertake the calculation in the correct order and using the correct items (and their values) requires a file defining the structure and sequence of calculations. 

This is the Programme file which defines a heirarchy of groups across a number of LEVELs.  LEVELs drive the sequence of calculation. A financial calculation is performed at successive LEVELs, depending on the structure of policy terms and conditions. For example there might be a coverage LEVEL, site LEVEL and policy LEVEL. Groups are defined within LEVELs and they represent aggregations of losses on which to perform the financial calculations.  The FROM_AGG_ID represents a grouping of the previous level of the heirarchy, and the TO_AGG_ID represents the grouping at the present level.

### Programme table rules
1. All fields have integer values
2. Must have at least one level. LEVEL_ID = 1, 2, 3 ...
2. For LEVEL_ID = 1, the FROM_AGG_ID must be equal to the ITEM_ID which cross references to the input ground up loss stream (which has fields EVENT_ID, ITEM_ID, IDX, GUL).  Therefore Level 1 always represents a group of items.
3. For subsequent levels, the FROM_AGG_ID must be the distinct values from the previous level TO_AGG_ID field.
4. Each programme table may only have a single integer value in PROG_ID. Note that this field is a cross-reference to the PROG dictionary and business meaningful information such as account number, and is not currently used in calculations.  This field may be deprecated in future versions.
5. The FROM_AGG_ID and TO_AGG_ID values, for each level, should be a contiguous block of integers (a sequence with no gaps).  This is not a strict rule in this version and it will work with non-contiguous integers, but it is recommended as best practice.

In R1.1 of Oasis we took the view that it was simpler throughout to refer back to the base ITEMs rather than use a hierarchy of aggregated loss calculations. So, for example, we could have calculated a loss at the site level and then used this calculated loss directly at the policy conditions level but instead we allocateed back to ITEM level and then re-aggregated to the next LEVEL. The reason for this was that intermediate conditions may only apply to certain ITEMs so if we didn’t go back to the base ITEM “ground-up” level then any higher LEVEL could have a complicated grouping of a LEVEL. 

In the implementation this required back-allocating losses to item at every level in a multi-level calculation even the next level calculaton did not require it.  In fact, the insurance policy structures provided as worked examples by Oasis members were simple heirarchies.   

The default execution path now supports only simple heirarchies but we may extend the functionality to handle both execution paths, if required by the Oasis members.

Another difference versus R1.1 is that in R1.1 not all ITEMs needed to be defined at every LEVEL so it was quite possible, for example, to include one ITEM with a coverage deductible out of a hundred thousand ITEMs in a Programme.  In the present version, all losses are explicitly included at every level in the heirarchy, and if no terms and conditions apply, then a null rule (zero deductible, for example) must be applied to all groupings.  The principle behind this change is 'no special rules' which makes the computation simpler and faster even though the input data may be larger.

Loss values
The initial input is the Ground-up Loss (GUL) table coming from the main Oasis calculation of ground-up losses. Here is an example, for a single event and sample (IDX=1):

| EVENT_ID | ITEM_ID  | IDX    | GUL    |
|:---------|----------|--------| ------:|
|       1  | 1        |    1   | 100,000|
|       1  | 2        |    1   | 10,000 |
|       1  | 3        |    1   | 2,500  |
|       1  | 4        |    1   | 400    |

The values represent a single ground-up loss sample for items belonging to an Account. We use “Programme” rather than Account as it is more general characteristic of a client’s exposure protection needs and allows a client to have multiple Programmes active for a given period.
The linkage between Account and Programme can be provided by a user defined **Prog** Dictionary, for example;

| PROG_ID  | ACCOUNT_ID  | PROG_NAME                |
|:---------|-------------|-------------------------:|
|       1  | 1           | Oasis hotels renewal 2011|
|       2  | 1           | Oasis hotels renewal 2012|
|       3  | 1           | Oasis hotels renewal 2013|
|       4  | 1           | Oasis hotels renewal 2014|

Items 1-4 represent Structure, Other Structure, Contents and Time Element coverage ground up losses for a single property, respectively, and this example is a simple residential policy with combined property coverage terms. For this policy type, the Structure, Other Structure and Contents losses are aggregated, and a deductible and limit is applied to the total. A separate set of terms, again a simple deductible and limit, is applied to the “time element” coverage which, for residential policies, generally means costs for temporary accommodation. The total insured loss is the sum of the output from the combined property terms and the time element terms.

The actual Items falling into the Programme are specified in the **Programme** table together with the aggregation groupings that go into a given LEVEL calculation:

| PROG _ID | ITEM _ID | LEVEL_ID| AGG_ID |
|:---------|----------|---------| ------:|
|       1  | 1        |     1   | 1      |
|       1  | 2        |     1   | 1      |
|       1  | 3        |     1   | 1      |
|       1  | 4        |     1   | 2      |
|       1  | 1        |     2   | 1      |
|       1  | 2        |     2   | 1      |
|       1  | 3        |     2   | 1      |
|       1  | 4        |     2   | 1      |

For example in LEVEL 1, Items 1, 2 and 3 all have AGG_ID =1 so losses will be summed together before applying the combined deductible and limit, but Item 4 (time) will be treated separately (not aggregated). For LEVEL 2 we have all 4 Items losses aggregated together as they have the same AGG_ID (value 1 in this case).

Next we have the Profile Description table, which list the profiles representing general policy types. Our example is represented by two general profiles which specify the input fields and mathematical operations to perform. In this example, the Profile for the combined coverages and time is the same (albeit with different values) and requires a limit, a deductible, and an associated calculation rule, whereas the Profile for the policy requires a limit, deductible, and share, and an associated calculation rule.

| ProfileID | Profile Description                          |
|:----------|:---------------------------------------------|
|       1   | Deductible and Limit (Function 1)            |
|       6   | Deductible, Limit and Share (Function 2)     |


There is a “Profile Value” table for each Profile containing the applicable policy terms, each identified by a POLICYTC_ID. The table below shows the list of policy terms for Profile 1.

| POLICYTC_ID | CCY_ID | LIM       | DED   | CALCRULE_ID |
|:------------|--------|-----------| ------|------------:|
|       1     | 1      | 1,000,000 | 1,000 |     1       |
|       2     | 1      |    18,000 | 2,000 |     1       |

And next, for Profile 6, the values for the overall policy deductible, limit and share

| POLICYTC_ID | CCY_ID | LIM       | DED   | SHARE  | CALCRULE_ID |
|:------------|--------|-----------| ------|--------|------------:|
|       3     | 1      | 1,000,000 | 1,000 |    0.1 |     2       |

In the present implementation, all profile values are stored in a single flattened format which contains all supported profile fields. The actual **profile** table would like this;

| POLICYTC_ID | CALCRULE_ID | ALLOCRULE_ID | SOURCERULE_ID | LEVELRULE_ID | CCY_ID | DED  | LIM     |  SHARE  | SHARE_PROP_OF_LIM | DEDUCTIBLE_PROP_OF_LOSS | LIMIT_PROP_OF_LOSS | DEDUCTIBLE_PROP_OF_TIV | LIMIT_PROP_OF_TIV | DEDUCTIBLE_PROP_OF_TIV | LIMIT_PROP_OF_TIV | DEDUCTIBLE_PROP_OF_LIMIT |
|:------------|-------------|--------------|---------------|--------------|--------|------|---------|---------|-------------------|---------------------------|--------------------|------------------------|-------------------|------------------------|------------------|-------------------------:|
|      1      | 1           |       -1     |        0      |       0      |    1   | 1,000|1,000,000|   0     |        0          |          0                |         0          |           0            |        0          |     0                  |   0              |         0                |

The **PolicyTC** table specifies the insurance policies (a policy in Oasis FM is a combination of PROG_ID and LAYER_ID, there is no separate Policy dictionary required in the calculation) and the separate terms and conditions which will be applied to each LAYER_ID/AGG_ID for a given LEVEL. In our example, we have a limit and deductible with the same value applicable to the combination of the first three Items a limit and deductible for the fourth Item (time) in LEVEL 1, and then a limit, deductible, and line applicable at LEVEL 2 covering all Items. We’d represent this in terms of the distinct AGG_IDs as follows:

| PROG_ID | LAYER_ID | LEVEL_ID | AGG_ID   | POLICYTC_ID |
|:--------|----------|----------| ---------|------------:|
|    1    |    1     |     1    |    1     |    1        |
|    1    |    1     |     1    |    2     |    2        |
|    1    |    1     |     2    |    1     |    3        |

The data in the table means; 
In Level 1;
Apply POLICYTC_ID (calculation rule) 1 to (the sum of losses grouped by) AGG_ID 1
Apply POLICYTC_ID 2 to AGG_ID 2
Then in Level 2;
Apply POLICYTC_ID 3 to AGG_ID 1

Levels are processed in ascending order and the calculated losses from a previous level are the input losses to the next level.
LEVEL_ID is included in the table to avoid duplicates if AGG_ID is not unique across Levels.

For any given Profile we have four standard fields:
 CALCRULE_ID, being the Function used to calculate the losses from the given profile’s fields. There are various families of such functions as shown in the list below
 ALLOCRULE_ID, being the rule for allocating back to ITEM level. There are really only two meaningful values here – don’t allocate (0) used typically for the final level to avoid maintaining lots of detailed losses, or allocate back to ITEMs (1) used in all other cases which is in proportion to the latest values input to that level for the items being processed. When at a level where each ITEM is treated separately (e.g. coverage deductibles) then there is another value (-1) used to tell Oasis not to waste time allocating back to a level it is already at. (Allocation does not need to be on this basis, by the way, there could be other rules such as allocate back always on TIV or ground-up loss, but the simplest solution is to allocate back in proportion to the losses contributing at that level.)
Oasis Data Interfaces Specification R1.1 Page 37
 SOURCERULE_ID (not currently in R1.1), which is used for conditional logic if TIV (for example) needs to be used in a calculation.
 LEVELRULE_ID (not currently in R1.1) used for processing level-specific rules such as “special conditions”.
