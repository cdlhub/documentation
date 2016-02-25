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
* Loss Values of Items (and TIVs if needed).
* Policy Profiles and Profile values.
 
There are many ways an insurance loss can be calculated with many different terms and conditions. For instance, there may be deductibles applied to each element of coverage (e.g. a buildings damage deductible), some site-specific deductibles or limits, and some overall policy deductibles and limits and line share. To undertake the calculation in the correct order and using the correct items (and their values) requires a file defining the structure and sequence of calculations. 

This is the Programme file which groups together ITEMs into LEVELs and for each LEVEL defines a way in which they are aggregated for the purpose of calculation.  We took the view that it was simpler throughout to refer back to the base ITEMs rather than use a hierarchy of aggregated loss calculations. So, for example, we could have calculated a loss at the site level and then used this calculated loss directly at the policy conditions level but instead what we have done is to allocate back to ITEM level and then re-aggregate to the next LEVEL. The reason for this is that intermediate conditions may only apply to certain ITEMs so if we didn’t go back to the base ITEM “ground-up” level then any higher LEVEL could have a complicated grouping of a LEVEL. Note that the design does not require this, it is just how we are currently implementing it.

Loss values
The initial input is the Ground-up Loss (GUL) table coming from the main Oasis calculation of ground-up losses. Here is an example, for a single event:

