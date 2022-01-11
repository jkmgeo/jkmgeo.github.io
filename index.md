---
layout : page
title : jkmgeo
---

# About
I am -- in order of decreasing skillfulness -- a dog dad, geologist, and coder.


I plan to explore building a website in the future, but for now, follow along as I record some useful code snippets for myself in plaintext. (All the better to facilitate "FCP" -- find, copy, paste!) Credit where credit's due ("CWCD") is indicated where appropriate.


## 2022-01-11

- task: lookup data in a pandas DataFrame and add it to another DataFrame

    - context: I had 2 dataframes -- 1 for analysed samples, 1 for external standard reference compositions -- and wanted to get the corresponding compositions for every standard analysed. However, there was only 1 row per standard in the external reference composition dataframe, so I couldn't use `merge`. (I also had to extract the elemental symbol from the isotope column header in the measured dataframe, not shown here.)

    - CWCD: https://kanoki.org/2019/04/06/pandas-map-dictionary-values-with-dataframe-columns/

    - code:
    ```python
    analysed_df[isotope_reference_data] = analysed_df[sample_ID_column].map(
        std_library_df[corresponding_element_column].to_dict()
    )
    ```

.
