# FM Profiles

| Profile description                                                | profile_id| calcrule_id |
|:-------------------------------------------------------------------|-----------|------------:|
| Deductible and limit                                               | 1         |   1         |
| Franchise deductible and limit                                     | 2         |   3         |
| Deductible only                                                    | 3         |  12         |
| Deductible as a cap on the retention of input losses               | 4         |  10         |
| Deductible as a floor on the retention of input losses             | 5         |  11         |
| Deductible, limit and share                                        | 6         |   2         |
| Deductible and limit as a proportion of loss                       | 10        |   5         |
| Limit with deductible as a proportion of limit                     | 11        |   9         |
| Limit only                                                         | 12        |   14        |
| Limit as a proportion of loss                                      | 13        |   15        |
| Deductible as a proportion of loss                                 | 14        |   16        |

#### 1. Deductible and limit

| Attributes                        | Example |
|:----------------------------------|--------:|
| policytc_id                       | 1       |
| ccy_id                            | 1       |
| deductible                        | 50000   |
| limit                             | 900000  |

| Rules                             | Value   |
|:----------------------------------|---------|
| calcrule_id                       | 1       | 

##### Logic

loss = x.loss - ded;

if (loss < 0) loss = 0;

if (loss > lim) loss = lim;

#### 1. Deductible and limit

| Attributes                        | Example |
|:----------------------------------|--------:|
| policytc_id                       | 1       |
| ccy_id                            | 1       |
| deductible                        | 50000   |
| limit                             | 900000  |

| Rules                             | Value   |
|:----------------------------------|---------|
| calcrule_id                       | 1       | 

##### Logic

loss = x.loss - ded;

if (loss < 0) loss = 0;

if (loss > lim) loss = lim;
