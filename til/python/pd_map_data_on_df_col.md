---
layout: page
title: "TIL pandas: mapping data"
---


# `pandas`: mapping data from one DataFrame to another based on column


### Context
Doing ICPMS data reduction, I had 2 dataframes -- 1 for analysed samples, and 1 
for external standard reference compositions. I wanted to get the corresponding 
compositions for every standard analysed. However, there was only 1 row per 
standard in the external reference composition dataframe, while the dataframe 
with analysed data had one row per sample, including unknowns; this prevented 
the use of `merge`. (Merging was hindered also by the fact that the column 
headers for the two dataframes didn't match: analysed data reported the isotope 
analysed, while the external standards reference compositions were given by 
chemical element; extracting the elemental symbol from the isotope column 
header is not shown here.)


### Solution
Use `pandas` `map` functionality to get the external standards elemental 
compositions as `dict` and append these reference compositions as new column to 
the analysed dataframe by matching on the sample names.


Untested: whether `map` can take a dataframe directly, and whether there is any 
way to map without `for` looping over every analyte measured (e.g., first bulk 
extract elemental symbols from the analysed column headers to find 
corresponding records in the reference compositions dataframe, then use `map`.)


### Code
```python
# convert Series of elemental compositions of stds to dict
# map the dict to a new col in the measured data according to each std's ID
# in effect: in measured data, get sample IDs and use these as keys on 
# which to map std composition values
analysed_df[isotope_reference_data] = analysed_df[sample_ID_column].map(
    std_compositions_df[corresponding_element_column].to_dict()
)
```


### Result


### Citation
[kanoki.org : map dict values with df col](https://kanoki.org/2019/04/06/pandas-map-dictionary-values-with-dataframe-columns/)


<div align="center">![go back](..)</div>
