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

#### 2. Franchise deductible and limit

| Attributes                        | Example |
|:----------------------------------|--------:|
| policytc_id                       | 1       |
| ccy_id                            | 1       |
| deductible                        | 100000  |
| limit                             | 1000000 |

| Rules                             | Value   |
|:----------------------------------|---------|
| calcrule_id                       | 3       | 

##### Logic

if (loss < ded) loss = 0;

else loss = loss;
  
if (loss > lim) loss = lim;

#### 3. Deductible only

| Attributes                        | Example |
|:----------------------------------|--------:|
| policytc_id                       | 1       |
| ccy_id                            | 1       |
| deductible                        | 100000  |

| Rules                             | Value   |
|:----------------------------------|---------|
| calcrule_id                       | 12      | 

##### Logic

loss = x.loss - deductible;

if (loss < 0) loss = 0;

#### 4. Deductible as a cap on the retention of input losses

| Attributes                        | Example |
|:----------------------------------|--------:|
| policytc_id                       | 1       |
| ccy_id                            | 1       |
| deductible                        | 40000   |

| Rules                             | Value   |
|:----------------------------------|---------|
| calcrule_id                       | 10      | 

##### Logic

if (x.retained_loss > ded) { 

	loss = x.loss + x.retained_loss - ded;
	
	if (loss < 0) loss = 0;
	
	x.retained_loss = x.retained_loss + ( x.loss - loss);
} else { 

	loss = x.loss;
	
}

#### 5. Deductible as a floor on the retention of input losses

| Attributes                        | Example |
|:----------------------------------|--------:|
| policytc_id                       | 1       |
| ccy_id                            | 1       |
| deductible                        | 70000   |

| Rules                             | Value   |
|:----------------------------------|---------|
| calcrule_id                       | 11      | 

##### Logic

if (x.retained_loss < ded) {

	loss = x.loss + x.retained_loss - ded;
	
	if (loss < 0) loss = 0;
	
	x.retained_loss = x.retained_loss + ( x.loss - loss);
 
} else { 

	loss = x.loss;
	
}

#### 6. Deductible, limit and share

| Attributes                        | Example |
|:----------------------------------|--------:|
| policytc_id                       | 1       |
| ccy_id                            | 1       |
| deductible                        | 70000   |
| limit                             | 1000000 |
| share_prop_of_lim                 | 1000000 |

| Rules                             | Value   |
|:----------------------------------|---------|
| calcrule_id                       | 2       | 

##### Logic

if (x.loss > (ded + lim)) loss = lim;

	else loss = x.loss - ded;
				
if (loss < 0) loss = 0;
				
	x.loss = loss * share;
				
#### 10. Deductible and limit as a proportion of loss

| Attributes                        | Example |
|:----------------------------------|--------:|
| policytc_id                       | 1       |
| deductible_prop_of_loss           | 0.05    |
| limit_prop_of_loss                | 0.3     |

| Rules                             | Value   |
|:----------------------------------|---------|
| calcrule_id                       | 5       | 

##### Logic

loss = x.loss * (lim - ded);

#### 11. Limit with deductible as a proportion of limit

| Attributes                        | Example |
|:----------------------------------|--------:|
| policytc_id                       | 1       |
| ccy_id                            | 1       |
| deductible_prop_of_loss           | 0.05    |
| limit_prop_of_loss                | 100000  |

| Rules                             | Value   |
|:----------------------------------|---------|
| calcrule_id                       | 9       | 

##### Logic

loss = x.loss - (ded * lim);

if (loss < 0) loss = 0;

if (loss > lim) loss = lim;
				
#### 12. Limit only

| Attributes                        | Example |
|:----------------------------------|--------:|
| policytc_id                       | 1       |
| ccy_id                            | 1       |
| limit                             | 100000  |

| Rules                             | Value   |
|:----------------------------------|---------|
| calcrule_id                       | 14     | 

##### Logic

loss = x.loss;

if (loss > lim) loss = lim;

#### 13. Limit as a proportion of loss

| Attributes                        | Example |
|:----------------------------------|--------:|
| policytc_id                       | 1       |
| ccy_id                            | 1       |
| limit_prop_of_loss                | 0.3     |

| Rules                             | Value   |
|:----------------------------------|---------|
| calcrule_id                       | 15      | 

##### Logic

loss = x.loss;

loss = loss * lim;

#### 14. Deductible as a proportion of loss

| Attributes                        | Example |
|:----------------------------------|--------:|
| policytc_id                       | 1       |
| ccy_id                            | 1       |
| deductible_prop_of_loss           | 0.05    |

| Rules                             | Value   |
|:----------------------------------|---------|
| calcrule_id                       | 16      | 

##### Logic

loss = x.loss;

loss = loss - (loss * ded);

if (loss < 0) loss = 0;
