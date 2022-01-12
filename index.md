---
layout : page
title : jkmgeo
---


[//]: # (Jekyll header info is optional; can also comment with `<!-- -->`)


# About
I am -- in order of decreasing skillfulness -- a dog dad, geologist, and coder.


I plan to explore building a website in the future, but for now, follow along as I record some useful code snippets for myself in plaintext. (All the better to facilitate "FCP" -- find, copy, paste!) Credit where credit's due ("CWCD") is indicated where appropriate.


## 2022-01-11

### lookup data in a pandas DataFrame and add it to another DataFrame

- context: I had 2 dataframes -- 1 for analysed samples, 1 for external standard reference compositions -- and wanted to get the corresponding compositions for every standard analysed. However, there was only 1 row per standard in the external reference composition dataframe, so I couldn't use `merge`. (I also had to extract the elemental symbol from the isotope column header in the measured dataframe, not shown here.)

- CWCD: [map dict values with df col | kanoki.org](https://kanoki.org/2019/04/06/pandas-map-dictionary-values-with-dataframe-columns/)

- code:
```python
# convert Series of elemental compositions of stds to dict
# map the dict to a new col in the measured data according to each std's ID
# in effect: in measured data, get sample IDs and use these as keys on 
# which to map std composition values
analysed_df[isotope_reference_data] = analysed_df[sample_ID_column].map(
    std_compositions_df[corresponding_element_column].to_dict()
)
```

### pandas: encode categorical data | matplotlib: color data by category

- context: plotting icpms external standard calibration data, I had some measured values that clusted at similar standard compositions, but "noisy" measured intensities. The lack of style differentiation between standards used to define the calibration curve made it hard to tell if these were 2 standards with coincidentally similar compositions, or bad data for a single standard. I didn't need the full power of `seaborn`'s `hue`, so figured I'd just add a little styling from `pandas` & `matplotlib`.

- CWCD: [encoding categorical data | pbpython.com](https://pbpython.com/categorical-encoding.html)

- code:
```python
# get array of numerical codes for categorical data by converting dtype
# N.B. doesn't permit category-wise labeling in legend
colors = df[categorical_col].astype('category').cat.codes

# example:
df = pd.DataFrame({
    "x": np.arange(10),
    "y": np.arange(10)**2,
    "class": 5*["a"] + 5*["b"]
})
plt.scatter(df.x, df.y, c=df["class"].astype('category').cat.codes)
plt.show()
```
.
