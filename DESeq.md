# DESeq2 Errors

## Duplicated Row Names

```
Fatal error: An undefined error occurred, please check your input carefully and contact your administrator.
Error in `row.names<-.data.frame`(`*tmp*`, value = value) :
  duplicate 'row.names' are not allowed
Calls: rownames<- ... rownames<- -> row.names<- -> row.names<-.data.frame
Warning message:
non-unique values when setting 'row.names': 'X0', 'X1'
```

DESeq2 complains that the column names of the input data (e.g., htseq-count data) has duplicated names. DESeq2 create a table based on the count data where the rows correspond to each sample.

Solution:
- Check if the count data has distinctive column names for each sample.
- Set `Files have header` to `No`

## Every Gene Contains at Least One Zero

```
Fatal error: An undefined error occurred, please check your input carefully and contact your administrator.
estimating size factors
Error in estimateSizeFactorsForMatrix(counts(object), locfunc = locfunc,  :
  every gene contains at least one zero, cannot compute log geometric means
Calls: DESeq ... estimateSizeFactors -> .local -> estimateSizeFactorsForMatrix
```

Every gene has at least one zero for at least one sample of your four samples.
The thing is that DESeq2 calculates a geometric mean for every gene over every sample.
The geometric mean will be zero if at least one entry is zero, i.e., if one of the four samples
has a zero count it will be zero and cannot be used. The geometric means are later important for the
DESeq2 normalisation.

Example:

| ID     | S1 |	S2 | S3 | S4 |
|--------|:----:|:-----:|:----:|:----:|
| Gene_1 |  1 |	22  | 3  |	0 |
| Gene_2 |  30|	0   | 12 |	5 |
| Gene_3 |	0 |	0   | 1  |	6 |

All Genes above are invalid and cannot be used by DESeq2 because at least one entry is zero.

What you need is something like:

| ID     | S1 |	S2 | S3 | S4 |
|--------|:----:|:-----:|:----:|:----:|
| Gene_1 |  1 |	22  | 3  |	1 |
| Gene_2 |  30|	1   | 12 |	5 |
| Gene_3 |	0 |	0   | 1  |	6 |

That would be fine since Gene_1 and Gene_2 can be used later for the normalisation.
Gene_3 will be filtered out by DESeq2.

Solution:
- You include more data
- You add a pseudo-count of 1 to all counts. Just add 1 to every entry. Thats technically
not cheating because you treat every entry and DESeq takes the log of the data anyway.
If you do that make sure you remove beforehand genes that have zeros for all samples, e.g., Remove Gene_1 in this example:

| ID     | S1 |	S2 | S3 | S4 |
|--------|:----:|:-----:|:----:|:----:|
| Gene_1 |  0 |	0  | 0  |	0 |
| Gene_2 |  30|	0   | 12 |	5 |
| Gene_3 |	0 |	0   | 1  |	6 |
