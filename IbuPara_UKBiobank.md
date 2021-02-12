# Investigation of the effect of several markers on COVID-19 risk and deaths
## Author: Yizhou Yu
### Affiliation: MRC Toxicology, University of Cambridge
#### Contract detail: yzy21 [at] mrc-tox.cam.ac.uk

Here, I will first extract the UK Biobank data, from all >500,000 people. I will then subset the data from all people who have been tested for COVID-19.

I will first test people who were infected (without information on death). This will allow me to determine what drives infection. <br>
In this analysis, I will try to include: UK Biobank phenotype and genotype markers (e.g. BMI, smoking status, dementia, etc.), their air pollution exposure (in 2018), the crowd mobility that they were exposed to, etc. 

I will then look at the adversive outcomes, and how these markers affect lethality.

Access to the UK Biobank data is granted under the application 60124.

Note that I had to downgrade R to run it with rpy2<br>
update.packages(ask=FALSE,
                checkBuilt=TRUE,
                repos="https://cloud.r-project.org")

<h1>Table of Contents<span class="tocSkip"></span></h1>
<div class="toc"><ul class="toc-item"><li><span><a href="#Investigation-of-the-effect-of-several-markers-on-COVID-19-risk-and-deaths" data-toc-modified-id="Investigation-of-the-effect-of-several-markers-on-COVID-19-risk-and-deaths-1"><span class="toc-item-num">1&nbsp;&nbsp;</span>Investigation of the effect of several markers on COVID-19 risk and deaths</a></span><ul class="toc-item"><li><span><a href="#Author:-Yizhou-Yu" data-toc-modified-id="Author:-Yizhou-Yu-1.1"><span class="toc-item-num">1.1&nbsp;&nbsp;</span>Author: Yizhou Yu</a></span><ul class="toc-item"><li><span><a href="#Affiliation:-MRC-Toxicology,-University-of-Cambridge" data-toc-modified-id="Affiliation:-MRC-Toxicology,-University-of-Cambridge-1.1.1"><span class="toc-item-num">1.1.1&nbsp;&nbsp;</span>Affiliation: MRC Toxicology, University of Cambridge</a></span><ul class="toc-item"><li><span><a href="#Contract-detail:-yzy21-[at]-mrc-tox.cam.ac.uk" data-toc-modified-id="Contract-detail:-yzy21-[at]-mrc-tox.cam.ac.uk-1.1.1.1"><span class="toc-item-num">1.1.1.1&nbsp;&nbsp;</span>Contract detail: yzy21 [at] mrc-tox.cam.ac.uk</a></span></li></ul></li></ul></li></ul></li><li><span><a href="#Data-Curation" data-toc-modified-id="Data-Curation-2"><span class="toc-item-num">2&nbsp;&nbsp;</span>Data Curation</a></span><ul class="toc-item"><li><span><a href="#Set-R-environment-&amp;-Load-python-libraries" data-toc-modified-id="Set-R-environment-&amp;-Load-python-libraries-2.1"><span class="toc-item-num">2.1&nbsp;&nbsp;</span>Set R environment &amp; Load python libraries</a></span></li><li><span><a href="#Obtain-the-UK-Biobank-phenotype-data-subset,-with-the-variables-of-interest." data-toc-modified-id="Obtain-the-UK-Biobank-phenotype-data-subset,-with-the-variables-of-interest.-2.2"><span class="toc-item-num">2.2&nbsp;&nbsp;</span>Obtain the UK Biobank phenotype data subset, with the variables of interest.</a></span><ul class="toc-item"><li><span><a href="#Create-a-selector-function" data-toc-modified-id="Create-a-selector-function-2.2.1"><span class="toc-item-num">2.2.1&nbsp;&nbsp;</span>Create a selector function</a></span></li></ul></li><li><span><a href="#Subset-the-participants-with-COVID-19-results-(as-of-August-2020)" data-toc-modified-id="Subset-the-participants-with-COVID-19-results-(as-of-August-2020)-2.3"><span class="toc-item-num">2.3&nbsp;&nbsp;</span>Subset the participants with COVID-19 results (as of August 2020)</a></span><ul class="toc-item"><li><span><a href="#Load-UK-Biobank-COVID-19-data" data-toc-modified-id="Load-UK-Biobank-COVID-19-data-2.3.1"><span class="toc-item-num">2.3.1&nbsp;&nbsp;</span>Load UK Biobank COVID-19 data</a></span><ul class="toc-item"><li><span><a href="#Get-the-date-range-of-the-test-results" data-toc-modified-id="Get-the-date-range-of-the-test-results-2.3.1.1"><span class="toc-item-num">2.3.1.1&nbsp;&nbsp;</span>Get the date range of the test results</a></span></li></ul></li><li><span><a href="#Save-curated-data" data-toc-modified-id="Save-curated-data-2.3.2"><span class="toc-item-num">2.3.2&nbsp;&nbsp;</span>Save curated data</a></span></li></ul></li><li><span><a href="#Mergeing-the-2-parts-of-this-dataset" data-toc-modified-id="Mergeing-the-2-parts-of-this-dataset-2.4"><span class="toc-item-num">2.4&nbsp;&nbsp;</span>Mergeing the 2 parts of this dataset</a></span><ul class="toc-item"><li><span><a href="#Here,-I-will-get-the-most-up-to-date-result" data-toc-modified-id="Here,-I-will-get-the-most-up-to-date-result-2.4.1"><span class="toc-item-num">2.4.1&nbsp;&nbsp;</span>Here, I will get the most up-to-date result</a></span></li><li><span><a href="#Subset-the-data-with-only-the-COVID-19-tested-participants" data-toc-modified-id="Subset-the-data-with-only-the-COVID-19-tested-participants-2.4.2"><span class="toc-item-num">2.4.2&nbsp;&nbsp;</span>Subset the data with only the COVID-19-tested participants</a></span><ul class="toc-item"><li><span><a href="#Select-rows-by-chunk" data-toc-modified-id="Select-rows-by-chunk-2.4.2.1"><span class="toc-item-num">2.4.2.1&nbsp;&nbsp;</span>Select rows by chunk</a></span></li><li><span><a href="#Select-rows-by-chunk" data-toc-modified-id="Select-rows-by-chunk-2.4.2.2"><span class="toc-item-num">2.4.2.2&nbsp;&nbsp;</span>Select rows by chunk</a></span></li></ul></li><li><span><a href="#Get-the-most-up-to-date-data-from-these-participants" data-toc-modified-id="Get-the-most-up-to-date-data-from-these-participants-2.4.3"><span class="toc-item-num">2.4.3&nbsp;&nbsp;</span>Get the most up-to-date data from these participants</a></span></li><li><span><a href="#Decode-with-the-UK-Biobank-values" data-toc-modified-id="Decode-with-the-UK-Biobank-values-2.4.4"><span class="toc-item-num">2.4.4&nbsp;&nbsp;</span>Decode with the UK Biobank values</a></span><ul class="toc-item"><li><span><a href="#Prepare-encodings" data-toc-modified-id="Prepare-encodings-2.4.4.1"><span class="toc-item-num">2.4.4.1&nbsp;&nbsp;</span>Prepare encodings</a></span></li><li><span><a href="#Decode-UKB-phenotype-data" data-toc-modified-id="Decode-UKB-phenotype-data-2.4.4.2"><span class="toc-item-num">2.4.4.2&nbsp;&nbsp;</span>Decode UKB phenotype data</a></span></li><li><span><a href="#Assign-custom-column-names" data-toc-modified-id="Assign-custom-column-names-2.4.4.3"><span class="toc-item-num">2.4.4.3&nbsp;&nbsp;</span>Assign custom column names</a></span></li></ul></li><li><span><a href="#Merge-the-UK-Biobank-phenotype-data-with-the-UK-Biobank-COVID-19-data" data-toc-modified-id="Merge-the-UK-Biobank-phenotype-data-with-the-UK-Biobank-COVID-19-data-2.4.5"><span class="toc-item-num">2.4.5&nbsp;&nbsp;</span>Merge the UK Biobank phenotype data with the UK Biobank COVID-19 data</a></span><ul class="toc-item"><li><span><a href="#Curate-the-dementia-diagnosis-data" data-toc-modified-id="Curate-the-dementia-diagnosis-data-2.4.5.1"><span class="toc-item-num">2.4.5.1&nbsp;&nbsp;</span>Curate the dementia diagnosis data</a></span></li><li><span><a href="#Hypertension-/-high-blood-pressure-calculations" data-toc-modified-id="Hypertension-/-high-blood-pressure-calculations-2.4.5.2"><span class="toc-item-num">2.4.5.2&nbsp;&nbsp;</span>Hypertension / high blood pressure calculations</a></span></li><li><span><a href="#Calculate-Age:-2021---birth-year" data-toc-modified-id="Calculate-Age:-2021---birth-year-2.4.5.3"><span class="toc-item-num">2.4.5.3&nbsp;&nbsp;</span>Calculate Age: 2021 - birth year</a></span></li><li><span><a href="#Drop-the-unused-columns" data-toc-modified-id="Drop-the-unused-columns-2.4.5.4"><span class="toc-item-num">2.4.5.4&nbsp;&nbsp;</span>Drop the unused columns</a></span></li></ul></li><li><span><a href="#Assign-population-density" data-toc-modified-id="Assign-population-density-2.4.6"><span class="toc-item-num">2.4.6&nbsp;&nbsp;</span>Assign population density</a></span></li></ul></li><li><span><a href="#Merge-generated-data-with-data-from-the-UK-Biobank" data-toc-modified-id="Merge-generated-data-with-data-from-the-UK-Biobank-2.5"><span class="toc-item-num">2.5&nbsp;&nbsp;</span>Merge generated data with data from the UK Biobank</a></span><ul class="toc-item"><li><span><a href="#Drop-unused-columns" data-toc-modified-id="Drop-unused-columns-2.5.1"><span class="toc-item-num">2.5.1&nbsp;&nbsp;</span>Drop unused columns</a></span></li><li><span><a href="#Add-2-more-variables-to-the-list:-education-and-care-home" data-toc-modified-id="Add-2-more-variables-to-the-list:-education-and-care-home-2.5.2"><span class="toc-item-num">2.5.2&nbsp;&nbsp;</span>Add 2 more variables to the list: education and care home</a></span><ul class="toc-item"><li><span><a href="#Get-the-most-up-to-date-data-from-these-participants" data-toc-modified-id="Get-the-most-up-to-date-data-from-these-participants-2.5.2.1"><span class="toc-item-num">2.5.2.1&nbsp;&nbsp;</span>Get the most up-to-date data from these participants</a></span></li><li><span><a href="#Decode" data-toc-modified-id="Decode-2.5.2.2"><span class="toc-item-num">2.5.2.2&nbsp;&nbsp;</span>Decode</a></span></li></ul></li><li><span><a href="#Replace-&quot;No-Answer&quot;-as-NA" data-toc-modified-id="Replace-&quot;No-Answer&quot;-as-NA-2.5.3"><span class="toc-item-num">2.5.3&nbsp;&nbsp;</span>Replace "No Answer" as NA</a></span></li><li><span><a href="#Curate-death-data" data-toc-modified-id="Curate-death-data-2.5.4"><span class="toc-item-num">2.5.4&nbsp;&nbsp;</span>Curate death data</a></span></li><li><span><a href="#Calculate-the-percentage-of-deaths-related-to-COVID-19-&amp;-odds-ratio" data-toc-modified-id="Calculate-the-percentage-of-deaths-related-to-COVID-19-&amp;-odds-ratio-2.5.5"><span class="toc-item-num">2.5.5&nbsp;&nbsp;</span>Calculate the percentage of deaths related to COVID-19 &amp; odds ratio</a></span></li><li><span><a href="#Add-drug-intake-data" data-toc-modified-id="Add-drug-intake-data-2.5.6"><span class="toc-item-num">2.5.6&nbsp;&nbsp;</span>Add drug-intake data</a></span></li><li><span><a href="#Subset-the-data-to-decrease-processing-time" data-toc-modified-id="Subset-the-data-to-decrease-processing-time-2.5.7"><span class="toc-item-num">2.5.7&nbsp;&nbsp;</span>Subset the data to decrease processing time</a></span></li><li><span><a href="#Create-separate-columns-for-each-of-the-8-drugs-investigated" data-toc-modified-id="Create-separate-columns-for-each-of-the-8-drugs-investigated-2.5.8"><span class="toc-item-num">2.5.8&nbsp;&nbsp;</span>Create separate columns for each of the 8 drugs investigated</a></span></li><li><span><a href="#Check-how-many-people-take-each-drug-in-the-drug-dataset" data-toc-modified-id="Check-how-many-people-take-each-drug-in-the-drug-dataset-2.5.9"><span class="toc-item-num">2.5.9&nbsp;&nbsp;</span>Check how many people take each drug in the drug dataset</a></span></li><li><span><a href="#Merge-and-save" data-toc-modified-id="Merge-and-save-2.5.10"><span class="toc-item-num">2.5.10&nbsp;&nbsp;</span>Merge and save</a></span></li></ul></li></ul></li><li><span><a href="#Relationship-between-ibuprofen-&amp;-paracetamol-and-COVID-19" data-toc-modified-id="Relationship-between-ibuprofen-&amp;-paracetamol-and-COVID-19-3"><span class="toc-item-num">3&nbsp;&nbsp;</span>Relationship between ibuprofen &amp; paracetamol and COVID-19</a></span><ul class="toc-item"><li><span><a href="#Descriptive-stats-for-the-drugs" data-toc-modified-id="Descriptive-stats-for-the-drugs-3.1"><span class="toc-item-num">3.1&nbsp;&nbsp;</span>Descriptive stats for the drugs</a></span><ul class="toc-item"><li><span><a href="#Susbet-amantidine" data-toc-modified-id="Susbet-amantidine-3.1.1"><span class="toc-item-num">3.1.1&nbsp;&nbsp;</span>Susbet amantidine</a></span></li></ul></li></ul></li><li><span><a href="#AD-/-PD-drug-analysis" data-toc-modified-id="AD-/-PD-drug-analysis-4"><span class="toc-item-num">4&nbsp;&nbsp;</span>AD / PD drug analysis</a></span><ul class="toc-item"><li><span><a href="#Data-curation" data-toc-modified-id="Data-curation-4.1"><span class="toc-item-num">4.1&nbsp;&nbsp;</span>Data curation</a></span><ul class="toc-item"><li><span><a href="#Subset-AD-&amp;-PD-eids" data-toc-modified-id="Subset-AD-&amp;-PD-eids-4.1.1"><span class="toc-item-num">4.1.1&nbsp;&nbsp;</span>Subset AD &amp; PD eids</a></span></li><li><span><a href="#Overview-of-drugs-taken-by-these-participants" data-toc-modified-id="Overview-of-drugs-taken-by-these-participants-4.1.2"><span class="toc-item-num">4.1.2&nbsp;&nbsp;</span>Overview of drugs taken by these participants</a></span></li><li><span><a href="#Curate-drugs-with-sufficient-data" data-toc-modified-id="Curate-drugs-with-sufficient-data-4.1.3"><span class="toc-item-num">4.1.3&nbsp;&nbsp;</span>Curate drugs with sufficient data</a></span></li></ul></li><li><span><a href="#Analysis---death" data-toc-modified-id="Analysis---death-4.2"><span class="toc-item-num">4.2&nbsp;&nbsp;</span>Analysis - death</a></span><ul class="toc-item"><li><span><a href="#Data-visualisation" data-toc-modified-id="Data-visualisation-4.2.1"><span class="toc-item-num">4.2.1&nbsp;&nbsp;</span>Data visualisation</a></span></li><li><span><a href="#AD-drug-model" data-toc-modified-id="AD-drug-model-4.2.2"><span class="toc-item-num">4.2.2&nbsp;&nbsp;</span>AD drug model</a></span><ul class="toc-item"><li><span><a href="#Complex-model" data-toc-modified-id="Complex-model-4.2.2.1"><span class="toc-item-num">4.2.2.1&nbsp;&nbsp;</span>Complex model</a></span></li><li><span><a href="#Models-with-only-age,-sex-and-whr-as-covariates" data-toc-modified-id="Models-with-only-age,-sex-and-whr-as-covariates-4.2.2.2"><span class="toc-item-num">4.2.2.2&nbsp;&nbsp;</span>Models with only age, sex and whr as covariates</a></span></li><li><span><a href="#raw-odds-of-drugs-in-the-AD-models" data-toc-modified-id="raw-odds-of-drugs-in-the-AD-models-4.2.2.3"><span class="toc-item-num">4.2.2.3&nbsp;&nbsp;</span>raw odds of drugs in the AD models</a></span></li></ul></li><li><span><a href="#PD-drug-models" data-toc-modified-id="PD-drug-models-4.2.3"><span class="toc-item-num">4.2.3&nbsp;&nbsp;</span>PD drug models</a></span><ul class="toc-item"><li><span><a href="#Complex-model" data-toc-modified-id="Complex-model-4.2.3.1"><span class="toc-item-num">4.2.3.1&nbsp;&nbsp;</span>Complex model</a></span></li><li><span><a href="#Simplified-model" data-toc-modified-id="Simplified-model-4.2.3.2"><span class="toc-item-num">4.2.3.2&nbsp;&nbsp;</span>Simplified model</a></span></li><li><span><a href="#Model-with-only-drug---raw-odds" data-toc-modified-id="Model-with-only-drug---raw-odds-4.2.3.3"><span class="toc-item-num">4.2.3.3&nbsp;&nbsp;</span>Model with only drug - raw odds</a></span></li></ul></li></ul></li><li><span><a href="#Analysis---infection" data-toc-modified-id="Analysis---infection-4.3"><span class="toc-item-num">4.3&nbsp;&nbsp;</span>Analysis - infection</a></span><ul class="toc-item"><li><span><a href="#AD-infection-models" data-toc-modified-id="AD-infection-models-4.3.1"><span class="toc-item-num">4.3.1&nbsp;&nbsp;</span>AD infection models</a></span><ul class="toc-item"><li><span><a href="#Visualisation" data-toc-modified-id="Visualisation-4.3.1.1"><span class="toc-item-num">4.3.1.1&nbsp;&nbsp;</span>Visualisation</a></span></li><li><span><a href="#Complex-model" data-toc-modified-id="Complex-model-4.3.1.2"><span class="toc-item-num">4.3.1.2&nbsp;&nbsp;</span>Complex model</a></span></li><li><span><a href="#Simplified-model" data-toc-modified-id="Simplified-model-4.3.1.3"><span class="toc-item-num">4.3.1.3&nbsp;&nbsp;</span>Simplified model</a></span></li></ul></li><li><span><a href="#PD-infection-models" data-toc-modified-id="PD-infection-models-4.3.2"><span class="toc-item-num">4.3.2&nbsp;&nbsp;</span>PD infection models</a></span><ul class="toc-item"><li><span><a href="#Complex-models" data-toc-modified-id="Complex-models-4.3.2.1"><span class="toc-item-num">4.3.2.1&nbsp;&nbsp;</span>Complex models</a></span></li><li><span><a href="#Simplified-model" data-toc-modified-id="Simplified-model-4.3.2.2"><span class="toc-item-num">4.3.2.2&nbsp;&nbsp;</span>Simplified model</a></span></li></ul></li></ul></li></ul></li></ul></div>

# Data Curation

## Set R environment & Load python libraries 


```python
import pandas as pd
import tensorflow as tf
#load R 
%load_ext rpy2.ipython
```

    /Users/yizhouyu/miniconda3/envs/yy_37_env2/lib/python3.7/site-packages/rpy2/robjects/pandas2ri.py:17: FutureWarning: pandas.core.index is deprecated and will be removed in a future version.  The public classes are available in the top-level namespace.
      from pandas.core.index import Index as PandasIndex


## Obtain the UK Biobank phenotype data subset, with the variables of interest.

### Create a selector function


This function selects the relevant variables for this analysis


```python
def UKB_select_rows(input_dt,col_list,output_dt):
    #input_dt = full UKB dataset
    #col_list = metadata 
    #output_dt = name of output data

    input_name = str(''.join(input_dt))
    output_name = str(''.join(output_dt))
    col_list_name = str(''.join(col_list))

    #create the list of columns to select 
    col_df = pd.read_csv(col_list_name, sep = ',', usecols = ["FieldID","Instances",'Array'])
    
    #this is the array of columns to select, dont forget to include eid...
    col_list_selected = ['eid']

    #here I make a loop to collect all instances + arrays of a field
    for index, row in col_df.iterrows():
        # range automatically goes from 0 to n-1
        instance_list = list(range(0,row['Instances'].astype(int)))
        list_single_field = []
        for instance_item in instance_list:
            field_instance = [row['FieldID'].astype(str) + "-" + str(instance_item)]
            list_single_field = list_single_field + field_instance
            array_list = list(range(0,row['Array'].astype(int)))
            for fi_item in list_single_field:
                for array_item in array_list:
                    field_instance_array = [str(fi_item) + "." + str(array_item)]
                    col_list_selected = col_list_selected + field_instance_array
                
    #print(len(col_list_selected))
    #col_df['UID'] = col_df["FieldID"].astype(str) + "-" + col_df["Instances"].astype(str) + "." + col_df['Array'].astype(str)
    #selected_UID_list = col_df['UID'].tolist()
    
    # read the large csv file with specified chunksize 
    #I tried with many chunksizes... 
    df_chunk = pd.read_csv(input_name, sep = ',', chunksize=50000, dtype={'eid': int},encoding= 'unicode_escape', usecols = col_list_selected)

    # Each chunk is in df format -> save them 
    # only write header once...
    write_header = True
    for chunk in df_chunk:
        print("50K done, still running...")
        chunk.to_csv(output_name, mode='a', index=False, header=write_header)
        write_header = False
```


```python
UKB_select_rows(input_dt="~/camDrive/ukb_dt_22_01_2021/ukb43784.csv", 
                col_list = "data/metadt.csv", 
                output_dt = "data/phenodt.csv")

```

    50K done, still running...
    50K done, still running...
    50K done, still running...
    50K done, still running...
    50K done, still running...
    50K done, still running...
    50K done, still running...
    50K done, still running...
    50K done, still running...
    50K done, still running...
    50K done, still running...


## Subset the participants with COVID-19 results (as of August 2020)

### Load UK Biobank COVID-19 data



```r
%%R 
raw_ukb_covid = read.csv("data/covid19_result_10022021.txt", sep = "\t")
nrow(raw_ukb_covid)
```


    [1] 113882



#### Get the date range of the test results 


```r
%%R
raw_ukb_covid$specdate = as.Date(raw_ukb_covid$specdate,tryFormats = "%d/%m/%Y")
max(raw_ukb_covid$specdate)
```


    [1] "2021-02-01"



Number of unique eids: 60446


```r
%%R
length(unique(raw_ukb_covid$eid))
```


    [1] 60446



For the the positive cases, I would like to know when they are first tested positive, i.e. the earliest date only. 


```r
%%R 
raw_ukb_covid.pos = subset(raw_ukb_covid, result == 1)
raw_ukb_covid.pos_cur = aggregate(raw_ukb_covid.pos, list(raw_ukb_covid.pos$eid), FUN=head, 1)
```

For the negative test results, I should keep the last date because that is the most up-to-date test. 


```r
%%R 
raw_ukb_covid.neg = subset(raw_ukb_covid, result == 0)
raw_ukb_covid.neg_cur = aggregate(raw_ukb_covid.neg, list(raw_ukb_covid.neg$eid), FUN=tail, 1)
```


```r
%%R
ukb_covid_bind = rbind(raw_ukb_covid.pos_cur, raw_ukb_covid.neg_cur)
```


```r
%%R
print(paste(length((ukb_covid_bind$eid)),length(unique(ukb_covid_bind$eid)),sep = " "))
```


    [1] "63487 60446"



There are  ~3000 eids that are duplicates. I will sum them to keep the positive tests only, with the latest date.


```r
%%R
ukb_covid_bind = ukb_covid_bind[c("eid","specdate","result")]
ukb_covid_bind$specdate = as.Date(ukb_covid_bind$specdate,tryFormats = "%d/%m/%Y")
ukb_covid_bind_test = aggregate(ukb_covid_bind, list(ukb_covid_bind$eid), FUN=max)[c("eid","specdate","result")]
nrow(ukb_covid_bind_test)
```


    [1] 60446




```r
%%R
sum(ukb_covid_bind_test$result)
```


    [1] 14877



### Save curated data

The curated data now is: if a participant is positive, then save this result & date, if not, save the latest (most up-to-date) negative test result.


```r
%%R
write.csv(ukb_covid_bind_test,"data_output/curated_ukb_covid_test_results.csv", row.names=FALSE)
```

## Mergeing the 2 parts of this dataset

### Here, I will get the most up-to-date result

### Subset the data with only the COVID-19-tested participants

#### Select rows by chunk


```python
def UKB_select_rows(input_dt,row_list,output_dt):
    input_name = str(''.join(input_dt))
    row_list_name = str(''.join(row_list))
    output_name = str(''.join(output_dt))

    #get the list of IDs
    ID_list_df = pd.read_csv(row_list_name, sep = ',', usecols = ['eid'], dtype={'eid': int})
    selected_ID_list = ID_list_df['eid'].tolist()
    #print(selected_ID_list)

    df_chunk = pd.read_csv(input_name, sep = ',', chunksize=10000, 
                           encoding= 'unicode_escape')

    #make a chunks list 
    chunk_list = []
    # Each chunk is in df format
    for chunk in df_chunk:  
        # perform data filtering 
        chunk_filter = chunk[chunk['eid'].isin(selected_ID_list)]
        print("10K done, still running...")
        #get the rows that are in the ID list
        #print(chunk_filter)
        # Once the data filtering is done, append the chunk to list
        chunk_list.append(chunk_filter)
        
    # concat the list into dataframe 
    #header_chunk.append(chunk_list)
    df_concat = pd.concat(chunk_list)
    #print(df_concat)
    df_concat.to_csv(output_name)
```

#### Select rows by chunk


```python
UKB_select_rows(input_dt="data/phenodt.csv",
                row_list="data_output/curated_ukb_covid_test_results.csv",
               output_dt="data_output/full_ukb_covidTested_results.csv")
```

    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...
    10K done, still running...


### Get the most up-to-date data from these participants



```r
%%R
# load non-covid UKB phenotype data
ukb_pheno_dt = read.csv("data_output/full_ukb_covidTested_results.csv")[-1]
#-1: delete first column of index from the previous function

#aggregate the columns by selecting the last results, for each 
library(reshape)
ukb_pheno_dt_t = melt(ukb_pheno_dt, id='eid')
# head(ukb_pheno_dt_t)

#order by variable to make sure I am will be replacing the latest value 
ukb_pheno_dt_t <- ukb_pheno_dt_t[order(ukb_pheno_dt_t$variable),] 

#delete everything after period 
ukb_pheno_dt_t$variable = gsub("\\..*","",ukb_pheno_dt_t$variable)

#aggregate by last 
ukb_covid_t_na = na.omit(ukb_pheno_dt_t)
ukb_covid_t_na = aggregate(ukb_covid_t_na, by=list(ukb_covid_t_na$eid,ukb_covid_t_na$variable), FUN=tail, n = 1)

# Curate
ukb_covid_t_na = ukb_covid_t_na[c("eid", "variable", "value")]
ukb_covid_t_na$variable = as.factor(ukb_covid_t_na$variable)

# put it back straight 
ukb_covid_cur = cast(ukb_covid_t_na, eid~variable)

```

### Decode with the UK Biobank values

#### Prepare encodings


```r
%%R
#get levels and labels 
lvl.0493 <- c(-141,-131,-121,0)
lbl.0493 <- c("Often","Sometimes","Do not know","Rarely/never")

lvl.0007 <- c(0,1)
lbl.0007 <- c("FALSE","TRUE")

lvl.100349 <- c(-3,-1,0,1)
lbl.100349 <- c("Prefer not to answer","Do not know","FALSE","TRUE")

lvl.0090 <- c(-3,0,1,2)
lbl.0090 <- c("Prefer not to answer","Never","Previous","Current")

lvl.0009 <- c(0,1)
lbl.0009 <- c("Female","Male")

lvl.0272 <- c(1)
lbl.0272 <- c("Date is unknown")

lvl.0300 <- c(0,1,2)
lbl.0300 <- c("Self-reported only","Hospital admission","Death only")

lvl.100290 <- c(-10,-3,-1)
lbl.100290 <- c("Less than a year","Prefer not to answer","Do not know")

lvl.100291 <- c(-3,-1)
lbl.100291 <- c("Prefer not to answer","Do not know")

lvl.100294 <- c(-3,-1,1,2,3,4,5)
lbl.100294 <- c("Prefer not to answer","Do not know",
                "Less than 18,000","18,000 to 30,999","31,000 to 51,999",
                "52,000 to 100,000","Greater than 100,000")

lvl.100298 <- c(-10,-3,-1)
lbl.100298 <- c("Less than once a week","Prefer not to answer","Do not know")

lvl.1001 <- c(-3,-1,1001,1002,1003,2001,2002,2003,2004,3001,3002,3003,3004,4001,4002,4003,1,2,3,4,5,6)
lbl.1001 <- c("Prefer not to answer","Do not know",
              "British","Irish","Any other white background","White and Black Caribbean",
              "White and Black African","White and Asian","Any other mixed background","Indian",
              "Pakistani","Bangladeshi","Any other Asian background","Caribbean","African","Any other Black background",
              "White","Mixed","Asian or Asian British","Black or Black British","Chinese","Other ethnic group")

### eid replacement, saved here ----
yy_replace <- function(string, patterns, replacements) {
  for (i in seq_along(patterns))
    string <- gsub(patterns[i], replacements[i], string, perl=TRUE)
  string
}
#label = replacement, lvl = to replace

```

#### Decode UKB phenotype data


```r
%%R
ukb_covid_decode = ukb_covid_cur
```


```r
%%R
colnames(ukb_covid_decode)
```


     [1] "eid"    "X134"   "X189"   "X20074" "X20075" "X20116" "X20153" "X21000"
     [9] "X22130" "X22609" "X2316"  "X2443"  "X31"    "X34"    "X4079"  "X4080" 
    [17] "X42019" "X48"    "X49"    "X50"    "X709"   "X738"   "X777"  




```r
%%R
ukb_covid_decode$X22609 <- yy_replace(ukb_covid_decode$X22609, lvl.0493, lbl.0493)

ukb_covid_decode$X2316 <- yy_replace(ukb_covid_decode$X2316, lvl.100349, lbl.100349)
ukb_covid_decode$X31 <- yy_replace(ukb_covid_decode$X31, lvl.0009, lbl.0009)
ukb_covid_decode$X22130 <- yy_replace(ukb_covid_decode$X22130, lvl.0007, lbl.0007)

ukb_covid_decode$X20116 <- yy_replace(ukb_covid_decode$X20116, lvl.0090, lbl.0090)

ukb_covid_decode$X42019 <- yy_replace(ukb_covid_decode$X42019, lvl.0300, lbl.0300)

ukb_covid_decode$X709 <- yy_replace(ukb_covid_decode$X709, lvl.100291, lbl.100291)

ukb_covid_decode$X777 <- yy_replace(ukb_covid_decode$X777, lvl.100298, lbl.100298)

#household income

ukb_covid_decode$X738 <- ordered(ukb_covid_decode$X738, levels = lvl.100294, 
                                 labels = lbl.100294)

#ethnicity
ukb_covid_decode$X21000 <- ordered(ukb_covid_decode$X21000, levels = lvl.1001, 
                                  labels = lbl.1001)

```

#### Assign custom column names


```r
%%R
#get the ID that I designed
UID_names = read.csv("data/metadt.csv")[c("FieldID","my_colname")]
UID_names$FieldID = paste("X",UID_names$FieldID, sep = "")
UID_names$my_colname = lapply(UID_names$my_colname,toString)
UID_names[nrow(UID_names) + 1,] = c("eid","eid")

# sort it according to the dataset

UID_names_match = colnames(ukb_covid_cur)

UID_sort_df = data.frame(UID_names_match)
colnames(UID_sort_df) <- "FieldID"

#merge to sort
UID_sort_df = merge(UID_sort_df, UID_names, by = "FieldID")
# this is now sorted according to the correct df

#replace column names
ukb_covid_pheno_df = ukb_covid_decode

colnames(ukb_covid_pheno_df) <- UID_sort_df$my_colname
colnames(ukb_covid_pheno_df)
```


     [1] "eid"                    "n_cancers"              "townsend"              
     [4] "x_coord"                "y_coord"                "smoking"               
     [7] "fev"                    "ethnicity"              "copd"                  
    [10] "WP_dusty"               "whistling"              "diabetes"              
    [13] "sex"                    "birthYear"              "diaBP"                 
    [16] "sysBP"                  "dementia"               "waist"                 
    [19] "hip"                    "height"                 "peopleInHousehold"     
    [22] "AverageHouseholdIncome" "travelToWork"          



### Merge the UK Biobank phenotype data with the UK Biobank COVID-19 data


```r
%%R 
ukb_covid_pheno_merged = merge(ukb_covid_pheno_df, ukb_covid_bind_test, by = "eid")
write.csv(ukb_covid_pheno_merged,"data_output/ukb_covid19_pheno_merged.csv", row.names=FALSE)
```

#### Curate the dementia diagnosis data


```r
%%R 
ukb_covid_pheno_merged = read.csv("data_output/ukb_covid19_pheno_merged.csv")
# PD diagnosis
library(dplyr)
# Ad diagnosis
ukb_covid_pheno_merged = ukb_covid_pheno_merged %>%
  mutate(Dem_diag = c("FALSE", "TRUE")[(!is.na(dementia)
                                     )+1] )
```

#### Hypertension / high blood pressure calculations


```r
%%R
ukb_covid_pheno_merged = ukb_covid_pheno_merged %>%
  mutate(highBP = c("FALSE", "TRUE")[(diaBP >= 90 | sysBP > 140
                                     )+1] )
```

#### Calculate Age: 2021 - birth year


```r
%%R 
ukb_covid_pheno_merged$age = 2020 - ukb_covid_pheno_merged$birthYear
```

#### Drop the unused columns


```r
%%R 
vars_drop = c("specdate","diaBP","sysBP")
ukb_covid_pheno_subset = ukb_covid_pheno_merged[ , !(names(ukb_covid_pheno_merged) %in% vars_drop)]
```


```r
%%R
write.csv(ukb_covid_pheno_subset,"data_output/merged_ukb_covid_test_results.csv", row.names=FALSE)
```

### Assign population density

Use the modelled N of people per 30m2
For the population density data, I downloaded the most precise and up-to-date models. This data has longitude and latitude values for every 30m2 as well as the modelled population value. In other words, these are estimations of "the number of people living within 30-meter grid tiles in nearly every country around the world." - as stated on their website.

Citation:
Facebook Connectivity Lab and Center for International Earth Science Information Network - CIESIN - Columbia University. 2016. High Resolution Settlement Layer (HRSL). Source imagery for HRSL © 2016 DigitalGlobe. Accessed 25 JUNE 2020.

Note: This dataset is NOT included in this repository because it is too large. The raw data can be downloaded here: https://data.humdata.org/dataset/united-kingdom-high-resolution-population-density-maps-demographic-estimates


```r
%%R
ukb_covid_pheno_subset = read.csv("data_output/merged_ukb_covid_test_results.csv")
ukgrid <- "+init=epsg:27700"
latlong <- "+init=epsg:4326"

ukb_coords = na.omit(ukb_covid_pheno_subset[c("eid","x_coord","y_coord")])
ukb_coords_NE <- data.frame(cbind(Easting = as.numeric(as.character(ukb_coords$x_coord)),
                     Northing = as.numeric(as.character(ukb_coords$y_coord))))
### Create the SpatialPointsDataFrame
library(sp)
ukb_covid_SP <- SpatialPointsDataFrame(ukb_coords_NE,
                                  data = ukb_coords,
                                  proj4string = CRS("+init=epsg:27700"))
library(rgdal)
ukb_covid_ll <- spTransform(ukb_covid_SP, CRS(latlong))

ukb_covid_ll_df = data.frame('ukb_lon' = coordinates(ukb_covid_ll)[, 1], 
                             'ukb_lat' = coordinates(ukb_covid_ll)[, 2], 
                             'eid' = ukb_covid_ll$eid)
write.csv(ukb_covid_ll_df, ("data_output/ukb_covid_lonlat_df.csv"))
```

    /Users/yizhouyu/miniconda3/envs/yy_37_env2/lib/python3.7/site-packages/rpy2/rinterface/__init__.py:146: RRuntimeWarning: rgdal: version: 1.4-4, (SVN revision 833)
     Geospatial Data Abstraction Library extensions to R successfully loaded
     Loaded GDAL runtime: GDAL 2.4.2, released 2019/06/28
     Path to GDAL shared files: /Users/yizhouyu/miniconda3/share/gdal
     GDAL binary built with GEOS: TRUE 
     Loaded PROJ.4 runtime: Rel. 6.1.0, May 15th, 2019, [PJ_VERSION: 610]
     Path to PROJ.4 shared files: /Users/yizhouyu/miniconda3/share/proj
     Linking to sp version: 1.3-1 
    
      warnings.warn(x, RRuntimeWarning)



```python
pop_dens = pd.read_csv('data/population_gbr_2019-07-01.csv')
```


```python
#build functions
from sklearn.neighbors import BallTree
import numpy as np

def get_nearest(src_points, candidates, k_neighbors=1):
    """Find nearest neighbors for all source points from a set of candidate points"""

    # Create tree from the candidate points
    tree = BallTree(candidates, leaf_size=15, metric='haversine')

    # Find closest points and distances
    distances, indices = tree.query(src_points, k=k_neighbors)

    # Transpose to get distances and indices into arrays
    distances = distances.transpose()
    indices = indices.transpose()

    # Get closest indices and distances (i.e. array at index 0)
    # note: for the second closest points, you would take index 1, etc.
    closest = indices[0]
    closest_dist = distances[0]

    # Return indices and distances
    return (closest, closest_dist)


def nearest_neighbor(left_gdf, right_gdf, return_dist=False):
    """
    For each point in left_gdf, find closest point in right GeoDataFrame and return them.

    NOTICE: Assumes that the input Points are in WGS84 projection (lat/lon).
    """

    left_geom_col = left_gdf.geometry.name
    right_geom_col = right_gdf.geometry.name

    # Ensure that index in right gdf is formed of sequential numbers
    right = right_gdf.copy().reset_index(drop=True)

    # Parse coordinates from points and insert them into a numpy array as RADIANS
    left_radians = np.array(left_gdf[left_geom_col].apply(lambda geom: (geom.x * np.pi / 180, geom.y * np.pi / 180)).to_list())
    right_radians = np.array(right[right_geom_col].apply(lambda geom: (geom.x * np.pi / 180, geom.y * np.pi / 180)).to_list())

    # Find the nearest points
    # -----------------------
    # closest ==> index in right_gdf that corresponds to the closest point
    # dist ==> distance between the nearest neighbors (in meters)

    closest, dist = get_nearest(src_points=left_radians, candidates=right_radians)

    # Return points from right GeoDataFrame that are closest to points in left GeoDataFrame
    closest_points = right.loc[closest]

    # Ensure that the index corresponds the one in left_gdf
    closest_points = closest_points.reset_index(drop=True)

    # Add distance if requested
    if return_dist:
        # Convert to meters from radians
        earth_radius = 6371000  # meters
        closest_points['distance'] = dist * earth_radius

    return closest_points
```


```python
import geopandas as gpd
ukb_covid_df = pd.read_csv("data_output/ukb_covid_lonlat_df.csv", usecols=['ukb_lon','ukb_lat','eid'])
ukb_covid = pd.read_csv("data_output/merged_ukb_covid_test_results.csv")

ukb_covid_gdf = gpd.GeoDataFrame(
    ukb_covid_df, geometry=gpd.points_from_xy(ukb_covid_df.ukb_lon, ukb_covid_df.ukb_lat))


pop_dens_gdf = gpd.GeoDataFrame(
    pop_dens, geometry=gpd.points_from_xy(pop_dens.Lon, pop_dens.Lat))
pop_dens_matched_df = nearest_neighbor(ukb_covid_gdf, pop_dens_gdf, return_dist=True).rename(columns={'geometry': 'pop_dens_geom'})
#join
pop_dens_joined_df = ukb_covid_gdf.join(pop_dens_matched_df)[["eid", "Population"]]
ukb_covid_AP_popDens_merged = pd.merge(ukb_covid, pop_dens_joined_df, on='eid',  how = 'left')
# ukb_covid_AP_popDens_merged.head()
```


```python
ukb_covid_AP_popDens_merged.to_csv("data_output/ukbCov_ukb_covid_AP_popDens_merged.csv", index = False)
```

## Merge generated data with data from the UK Biobank


```python
ukb_covid_AP_popDens_merged['whr'] = ukb_covid_AP_popDens_merged['waist']/ukb_covid_AP_popDens_merged['hip']
ukb_covid_AP_popDens_merged.to_csv("data_output/ukb_covid_merged_full_curatedData.csv")
```


```python
ukb_covid_AP_popDens_merged.shape
```




    (60446, 27)



Note that some participants did not provide location data.

### Drop unused columns


```python
for col in ukb_covid_AP_popDens_merged.columns: 
    print(col) 
```

    eid
    n_cancers
    townsend
    x_coord
    y_coord
    smoking
    fev
    ethnicity
    copd
    WP_dusty
    whistling
    diabetes
    sex
    birthYear
    dementia
    waist
    hip
    height
    peopleInHousehold
    AverageHouseholdIncome
    travelToWork
    result
    Dem_diag
    highBP
    age
    Population
    whr



```python
ukb_covidAP = ukb_covid_AP_popDens_merged.drop([
                       'x_coord','y_coord',
                        'fev','dementia',
                        'birthYear'], axis=1)
ukb_covidAP.to_csv("data_output/ukb_covid_merged_full_curatedData.csv", index = False)
```

### Add 2 more variables to the list: education and care home

I extracted these variables and stored them in a separate folder so that they are not uploaded on github. Access to those data should be done via UK Biobank. 

#### Get the most up-to-date data from these participants


```r
%%R

care_home_edu_dt = read.csv("data/care_home_edu_dt.csv")

#aggregate the columns by selecting the last results, for each 
library(reshape)
care_home_edu_dt_t = melt(care_home_edu_dt, id='eid')

#order by variable to make sure I am will be replacing the latest value 
care_home_edu_dt_t <- care_home_edu_dt_t[order(care_home_edu_dt_t$variable),] 
#delete everything after period 
care_home_edu_dt_t$variable = gsub("\\..*","",care_home_edu_dt_t$variable)

#aggregate by last 
care_home_edu_dt_t_na = na.omit(care_home_edu_dt_t)
care_home_edu_dt_t_na = aggregate(care_home_edu_dt_t_na, by=list(care_home_edu_dt_t_na$eid,care_home_edu_dt_t_na$variable), FUN=tail, n = 1)
# Curate
care_home_edu_dt_t_na = care_home_edu_dt_t_na[c("eid", "variable", "value")]
care_home_edu_dt_t_na$variable = as.factor(care_home_edu_dt_t_na$variable)

# put it back straight 
care_home_edu_cur = cast(care_home_edu_dt_t_na, eid~variable)
head(care_home_edu_cur)
```


          eid X6138 X670
    1 1000013     2    1
    2 1000024    -7    1
    3 1000036     3    1
    4 1000048     6    1
    5 1000055     2    1
    6 1000067     6    1



#### Decode 

Prefer not to answer is considered as NA


```r
%%R
# I keep the ordered function because all of the variables need to be decoded
care_home_edu_dec = care_home_edu_cur
#housing
lvl.100286 <- c(-7,-3,1,2,3,4,5)
lbl.100286 <- c("None of the above","NA","A house or bungalow","A flat, maisonette or apartment","Mobile or temporary structure (i.e. caravan)","Sheltered accommodation","Care home")

care_home_edu_dec$X670 <- ordered(care_home_edu_dec$X670, levels=lvl.100286, labels=lbl.100286)


#education
lvl.100305 <- c(-7,-3,1,2,3,4,5,6)
lbl.100305 <- c("None of the above","NA","College or University degree","A levels/AS levels or equivalent","O levels/GCSEs or equivalent","CSEs or equivalent","NVQ or HND or HNC or equivalent","Other professional qualifications eg: nursing, teaching")
care_home_edu_dec$X6138 <- ordered(care_home_edu_dec$X6138, levels=lvl.100305, labels=lbl.100305)
colnames(care_home_edu_dec) <- c("eid","edu_level","house_type")
write.csv(care_home_edu_dec,"data_output/care_home_edu_decoded.csv", row.names = FALSE)
```


```r
%%R
#Merge to data
temp_merged = merge(read.csv("data_output/ukb_covid_merged_full_curatedData.csv"), 
                    read.csv("data_output/care_home_edu_decoded.csv"), by = "eid", all.x = TRUE)
write.csv(temp_merged,"data_output/ukb_covid_phenodt.csv")
```

### Replace "No Answer" as NA


```r
%%R
ukb_covidAP = read.csv("data_output/ukb_covid_phenodt.csv")
ukb_covidAP[] <- lapply(ukb_covidAP, function(x) (gsub("Prefer not to answer", "NA", x)))
ukb_covidAP[] <- lapply(ukb_covidAP, function(x) (gsub("Do not know", "NA", x)))

ukb_covidAP$ethnicity_sim <- ifelse(ukb_covidAP$ethnicity == "White" | 
                                ukb_covidAP$ethnicity == "British", "white", "minority")
write.csv(ukb_covidAP,"data_output/ukb_covid_phenodt.csv", row.names=FALSE)
```

### Curate death data


```r
%%R

ukb_death = read.csv("data/death_cause_10022021.txt", sep = "\t")[c('eid', 'level', 'cause_icd10')]

ukb_dt = read.csv('data_output/ukb_covid_phenodt.csv', sep = ",")
     
ukb_dt_death = merge(ukb_dt, ukb_death, by = "eid")

ukb_deaths_eids = unique(ukb_dt_death$eid)


ukb_dt$death = ukb_dt$eid %in% ukb_deaths_eids

print(sum(ukb_dt$death))

write.csv(ukb_dt,"data_output/ukb_covid_addDeath.csv", row.names=FALSE)
```


    [1] 2031



### Calculate the percentage of deaths related to COVID-19 & odds ratio

29% of deaths were associated with COVID-19


```r
%%R

ukb_dt = read.csv("data_output/ukb_covid_addDeath.csv")
ukb_dt_died = subset(ukb_dt, death == TRUE)

print(sum(ukb_dt_died$result)/sum(ukb_dt$death))
```


    [1] 0.2880516



### Add drug-intake data

### Subset the data to decrease processing time


```r
%%R

med_raw = read.csv("data/medications_20003_dt.csv")
med_coding = read.csv("data/medications_coding4.tsv", 
                     sep = "\t")
```

2038460150 - paracetamol <br>
1140871310 - ibuprofen  <br>


```r
%%R

drug_list = c("paracetamol","ibuprofen")
coding_subset = med_coding[med_coding$meaning %in% drug_list, ]
# length(drug_list)
nrow(coding_subset)
```


    [1] 2




```r
%%R
#subset 
eid_covid = read.csv("data_output/ukb_covid_addDeath.csv")[,c("eid")]

med_covid = med_raw[med_raw$eid %in% eid_covid, ]

nrow(med_covid)
```


    [1] 60446



### Create separate columns for each of the 8 drugs investigated


```r
%%R
coding_subset
```


             coding     meaning
    2154 1140871310   ibuprofen
    6744 2038460150 paracetamol




```r
%%R

med_covid$ibuprofen = apply(med_covid, 1, function(x) any(x %in% c("1140871310")))
med_covid$paracetamol = apply(med_covid, 1, function(x) any(x %in% c("2038460150")))                           

```


```r
%%R
med_covid_out = med_covid[,c("eid","ibuprofen","paracetamol")]

write.csv(med_covid_out,"data_output/covid_drug_subset.csv",row.names=FALSE)

```

### Check how many people take each drug in the drug dataset


```r
%%R

colSums(med_covid_out[,-1])
```


      ibuprofen paracetamol 
           8277       13189 



### Merge and save


```r
%%R
write.csv(merge(
    read.csv(
        "data_output/ukb_covid_addDeath.csv"),
    read.csv("data_output/covid_drug_subset.csv"), by = "eid",all.x = TRUE),
          "data_output/ukb_covid_addDrugs.csv", row.names = F)

```

# Relationship between ibuprofen & paracetamol and COVID-19 

## Descriptive stats for the drugs


```r
%%R
ukb_dt = read.csv("data_output/ukb_covid_addDrugs.csv")
ukb_dt$covid_death = ifelse(ukb_dt$result == 1 & ukb_dt$death == TRUE, 
1, 0)

library(arsenal)

ukb_descript_stats <- tableby(covid_death ~ ., data = ukb_dt[,2:ncol(ukb_dt)])

write2word(ukb_descript_stats, "ukb_descriptStats_drugs_death.doc",
  keep.md = TRUE,
  quiet = TRUE, # passed to rmarkdown::render
  title = "Descriptive statistics of the UK Biobank data") # passed to summary.tableby
```


```r
%%R
ukb_descript_stats <- tableby(result ~ ., data = ukb_dt[,2:ncol(ukb_dt)])

write2word(ukb_descript_stats, "ukb_descriptStats_drugs_infection.doc",
  keep.md = TRUE,
  quiet = TRUE, # passed to rmarkdown::render
  title = "Descriptive statistics of the UK Biobank data") # passed to summary.tableby
```


```r
%%R
colnames(ukb_dt)
```


     [1] "eid"                    "X"                      "n_cancers"             
     [4] "townsend"               "smoking"                "ethnicity"             
     [7] "copd"                   "WP_dusty"               "whistling"             
    [10] "diabetes"               "sex"                    "waist"                 
    [13] "hip"                    "height"                 "peopleInHousehold"     
    [16] "AverageHouseholdIncome" "travelToWork"           "result"                
    [19] "Dem_diag"               "highBP"                 "age"                   
    [22] "Population"             "whr"                    "edu_level"             
    [25] "house_type"             "ethnicity_sim"          "death"                 
    [28] "ibuprofen"              "paracetamol"            "covid_death"           




```r
%%R

ukb_dt = read.csv("data_output/ukb_covid_addDrugs.csv")
ukb_dt$covid_death = ifelse(ukb_dt$result == 1 & ukb_dt$death == TRUE, 
1, 0)

ukb_dt_subset = na.omit(subset(ukb_dt, select = c(covid_death,result,sex, age, townsend, n_cancers, diabetes, whr, edu_level,
                peopleInHousehold, Population, ethnicity_sim, house_type,highBP,Dem_diag,copd,whistling)))

death_general = glm(formula = covid_death ~ sex + age +townsend + n_cancers + diabetes + whr + edu_level+
                peopleInHousehold + Population + ethnicity_sim + house_type+highBP+Dem_diag+copd+whistling,                   
    family = "binomial", data = ukb_dt_subset)
infection_general = glm(formula = result ~ sex + age +townsend + n_cancers + diabetes + whr + edu_level+
                peopleInHousehold + Population + ethnicity_sim + house_type+highBP+Dem_diag+copd+whistling,                   
    family = "binomial", data = ukb_dt_subset)
```


```r
%%R

library(MASS)
# Stepwise regression model
death_general_step <- stepAIC(death_general, direction = "both")
summary(death_general_step)
```


    Start:  AIC=5421.58
    covid_death ~ sex + age + townsend + n_cancers + diabetes + whr + 
        edu_level + peopleInHousehold + Population + ethnicity_sim + 
        house_type + highBP + Dem_diag + copd + whistling
    
                        Df Deviance    AIC
    - house_type         5   5372.9 5416.9
    - peopleInHousehold  1   5368.1 5420.1
    - Population         1   5368.6 5420.6
    <none>                   5367.6 5421.6
    - ethnicity_sim      1   5369.9 5421.9
    - n_cancers          1   5370.3 5422.3
    - edu_level          6   5380.9 5422.9
    - whistling          1   5371.9 5423.9
    - highBP             2   5374.3 5424.3
    - sex                1   5374.2 5426.2
    - copd               2   5379.9 5429.9
    - diabetes           1   5379.1 5431.1
    - townsend           1   5382.5 5434.5
    - whr                1   5394.8 5446.8
    - Dem_diag           1   5400.0 5452.0
    - age                1   5546.7 5598.7
    
    Step:  AIC=5416.89
    covid_death ~ sex + age + townsend + n_cancers + diabetes + whr + 
        edu_level + peopleInHousehold + Population + ethnicity_sim + 
        highBP + Dem_diag + copd + whistling
    
                        Df Deviance    AIC
    - peopleInHousehold  1   5374.0 5416.0
    - Population         1   5374.0 5416.0
    <none>                   5372.9 5416.9
    - edu_level          6   5385.5 5417.5
    - n_cancers          1   5375.5 5417.5
    - ethnicity_sim      1   5375.6 5417.6
    - whistling          1   5377.1 5419.1
    - highBP             2   5379.5 5419.5
    + house_type         5   5367.6 5421.6
    - sex                1   5379.7 5421.7
    - copd               2   5385.2 5425.2
    - diabetes           1   5384.6 5426.6
    - townsend           1   5396.1 5438.1
    - whr                1   5400.3 5442.3
    - Dem_diag           1   5405.1 5447.1
    - age                1   5550.2 5592.2
    
    Step:  AIC=5415.98
    covid_death ~ sex + age + townsend + n_cancers + diabetes + whr + 
        edu_level + Population + ethnicity_sim + highBP + Dem_diag + 
        copd + whistling
    
                        Df Deviance    AIC
    - Population         1   5375.1 5415.1
    <none>                   5374.0 5416.0
    - ethnicity_sim      1   5376.3 5416.3
    - n_cancers          1   5376.6 5416.6
    - edu_level          6   5386.6 5416.6
    + peopleInHousehold  1   5372.9 5416.9
    - whistling          1   5378.2 5418.2
    - highBP             2   5380.7 5418.7
    + house_type         5   5368.1 5420.1
    - sex                1   5380.4 5420.4
    - copd               2   5386.2 5424.2
    - diabetes           1   5385.7 5425.7
    - townsend           1   5398.8 5438.8
    - whr                1   5401.6 5441.6
    - Dem_diag           1   5406.3 5446.3
    - age                1   5579.5 5619.5
    
    Step:  AIC=5415.07
    covid_death ~ sex + age + townsend + n_cancers + diabetes + whr + 
        edu_level + ethnicity_sim + highBP + Dem_diag + copd + whistling
    
                        Df Deviance    AIC
    <none>                   5375.1 5415.1
    - ethnicity_sim      1   5377.5 5415.5
    - n_cancers          1   5377.7 5415.7
    - edu_level          6   5387.8 5415.8
    + Population         1   5374.0 5416.0
    + peopleInHousehold  1   5374.0 5416.0
    - whistling          1   5379.2 5417.2
    - highBP             2   5381.7 5417.7
    + house_type         5   5369.1 5419.1
    - sex                1   5381.6 5419.6
    - copd               2   5387.2 5423.2
    - diabetes           1   5386.7 5424.7
    - townsend           1   5401.0 5439.0
    - whr                1   5402.7 5440.7
    - Dem_diag           1   5407.6 5445.6
    - age                1   5580.4 5618.4
    
    Call:
    glm(formula = covid_death ~ sex + age + townsend + n_cancers + 
        diabetes + whr + edu_level + ethnicity_sim + highBP + Dem_diag + 
        copd + whistling, family = "binomial", data = ukb_dt_subset)
    
    Deviance Residuals: 
        Min       1Q   Median       3Q      Max  
    -0.8214  -0.1514  -0.1010  -0.0640   3.8630  
    
    Coefficients:
                                                                       Estimate
    (Intercept)                                                      -14.956897
    sexMale                                                            0.301899
    age                                                                0.102872
    townsend                                                           0.070675
    n_cancers                                                         -0.213333
    diabetes                                                           0.422905
    whr                                                                3.510299
    edu_levelCollege or University degree                              0.052544
    edu_levelCSEs or equivalent                                       -0.029902
    edu_levelNone of the above                                         0.338304
    edu_levelNVQ or HND or HNC or equivalent                           0.059375
    edu_levelO levels/GCSEs or equivalent                             -0.123028
    edu_levelOther professional qualifications eg: nursing, teaching   0.091946
    ethnicity_simwhite                                                -0.219149
    highBPFalse                                                       -0.487349
    highBPTrue                                                        -0.289264
    Dem_diagTrue                                                       1.391140
    copdFalse                                                         -0.469493
    copdTrue                                                          -0.264584
    whistlingTRUE                                                      0.198210
                                                                     Std. Error
    (Intercept)                                                        0.882013
    sexMale                                                            0.119374
    age                                                                0.007932
    townsend                                                           0.013623
    n_cancers                                                          0.137605
    diabetes                                                           0.119932
    whr                                                                0.663514
    edu_levelCollege or University degree                              0.317413
    edu_levelCSEs or equivalent                                        0.379938
    edu_levelNone of the above                                         0.291447
    edu_levelNVQ or HND or HNC or equivalent                           0.307680
    edu_levelO levels/GCSEs or equivalent                              0.307300
    edu_levelOther professional qualifications eg: nursing, teaching   0.296439
    ethnicity_simwhite                                                 0.137105
    highBPFalse                                                        0.264388
    highBPTrue                                                         0.262085
    Dem_diagTrue                                                       0.208517
    copdFalse                                                          0.142175
    copdTrue                                                           0.587469
    whistlingTRUE                                                      0.096653
                                                                     z value
    (Intercept)                                                      -16.958
    sexMale                                                            2.529
    age                                                               12.969
    townsend                                                           5.188
    n_cancers                                                         -1.550
    diabetes                                                           3.526
    whr                                                                5.290
    edu_levelCollege or University degree                              0.166
    edu_levelCSEs or equivalent                                       -0.079
    edu_levelNone of the above                                         1.161
    edu_levelNVQ or HND or HNC or equivalent                           0.193
    edu_levelO levels/GCSEs or equivalent                             -0.400
    edu_levelOther professional qualifications eg: nursing, teaching   0.310
    ethnicity_simwhite                                                -1.598
    highBPFalse                                                       -1.843
    highBPTrue                                                        -1.104
    Dem_diagTrue                                                       6.672
    copdFalse                                                         -3.302
    copdTrue                                                          -0.450
    whistlingTRUE                                                      2.051
                                                                     Pr(>|z|)    
    (Intercept)                                                       < 2e-16 ***
    sexMale                                                          0.011439 *  
    age                                                               < 2e-16 ***
    townsend                                                         2.12e-07 ***
    n_cancers                                                        0.121061    
    diabetes                                                         0.000422 ***
    whr                                                              1.22e-07 ***
    edu_levelCollege or University degree                            0.868521    
    edu_levelCSEs or equivalent                                      0.937269    
    edu_levelNone of the above                                       0.245734    
    edu_levelNVQ or HND or HNC or equivalent                         0.846978    
    edu_levelO levels/GCSEs or equivalent                            0.688899    
    edu_levelOther professional qualifications eg: nursing, teaching 0.756433    
    ethnicity_simwhite                                               0.109954    
    highBPFalse                                                      0.065284 .  
    highBPTrue                                                       0.269722    
    Dem_diagTrue                                                     2.53e-11 ***
    copdFalse                                                        0.000959 ***
    copdTrue                                                         0.652437    
    whistlingTRUE                                                    0.040292 *  
    ---
    Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
    
    (Dispersion parameter for binomial family taken to be 1)
    
        Null deviance: 5996.6  on 56589  degrees of freedom
    Residual deviance: 5375.1  on 56570  degrees of freedom
    AIC: 5415.1
    
    Number of Fisher Scoring iterations: 8
    




```r
%%R

library(MASS)
# Stepwise regression model
infection_general_step <- stepAIC(infection_general, direction = "both")
summary(infection_general_step)
```


    Start:  AIC=59287.16
    result ~ sex + age + townsend + n_cancers + diabetes + whr + 
        edu_level + peopleInHousehold + Population + ethnicity_sim + 
        house_type + highBP + Dem_diag + copd + whistling
    
                        Df Deviance   AIC
    - highBP             2    59234 59284
    - sex                1    59233 59285
    - Population         1    59233 59285
    - diabetes           1    59234 59286
    - whistling          1    59234 59286
    <none>                    59233 59287
    - ethnicity_sim      1    59248 59300
    - n_cancers          1    59254 59306
    - whr                1    59260 59312
    - house_type         5    59269 59313
    - Dem_diag           1    59278 59330
    - copd               2    59312 59362
    - townsend           1    59328 59380
    - peopleInHousehold  1    59362 59414
    - edu_level          6    59486 59528
    - age                1    60933 60985
    
    Step:  AIC=59283.71
    result ~ sex + age + townsend + n_cancers + diabetes + whr + 
        edu_level + peopleInHousehold + Population + ethnicity_sim + 
        house_type + Dem_diag + copd + whistling
    
                        Df Deviance   AIC
    - sex                1    59234 59282
    - Population         1    59234 59282
    - diabetes           1    59234 59282
    - whistling          1    59234 59282
    <none>                    59234 59284
    + highBP             2    59233 59287
    - ethnicity_sim      1    59249 59297
    - n_cancers          1    59254 59302
    - whr                1    59261 59309
    - house_type         5    59269 59309
    - Dem_diag           1    59279 59327
    - copd               2    59313 59359
    - townsend           1    59329 59377
    - peopleInHousehold  1    59363 59411
    - edu_level          6    59486 59524
    - age                1    60984 61032
    
    Step:  AIC=59281.92
    result ~ age + townsend + n_cancers + diabetes + whr + edu_level + 
        peopleInHousehold + Population + ethnicity_sim + house_type + 
        Dem_diag + copd + whistling
    
                        Df Deviance   AIC
    - Population         1    59234 59280
    - whistling          1    59234 59280
    - diabetes           1    59234 59280
    <none>                    59234 59282
    + sex                1    59234 59284
    + highBP             2    59233 59285
    - ethnicity_sim      1    59249 59295
    - n_cancers          1    59254 59300
    - house_type         5    59270 59308
    - whr                1    59275 59321
    - Dem_diag           1    59279 59325
    - copd               2    59313 59357
    - townsend           1    59329 59375
    - peopleInHousehold  1    59363 59409
    - edu_level          6    59487 59523
    - age                1    60984 61030
    
    Step:  AIC=59280.23
    result ~ age + townsend + n_cancers + diabetes + whr + edu_level + 
        peopleInHousehold + ethnicity_sim + house_type + Dem_diag + 
        copd + whistling
    
                        Df Deviance   AIC
    - whistling          1    59235 59279
    - diabetes           1    59235 59279
    <none>                    59234 59280
    + Population         1    59234 59282
    + sex                1    59234 59282
    + highBP             2    59234 59284
    - ethnicity_sim      1    59249 59293
    - n_cancers          1    59255 59299
    - house_type         5    59270 59306
    - whr                1    59276 59320
    - Dem_diag           1    59279 59323
    - copd               2    59314 59356
    - townsend           1    59330 59374
    - peopleInHousehold  1    59363 59407
    - edu_level          6    59488 59522
    - age                1    60984 61028
    
    Step:  AIC=59278.74
    result ~ age + townsend + n_cancers + diabetes + whr + edu_level + 
        peopleInHousehold + ethnicity_sim + house_type + Dem_diag + 
        copd
    
                        Df Deviance   AIC
    - diabetes           1    59235 59277
    <none>                    59235 59279
    + whistling          1    59234 59280
    + Population         1    59234 59280
    + sex                1    59235 59281
    + highBP             2    59234 59282
    - ethnicity_sim      1    59250 59292
    - n_cancers          1    59255 59297
    - house_type         5    59271 59305
    - whr                1    59276 59318
    - Dem_diag           1    59280 59322
    - copd               2    59314 59354
    - townsend           1    59330 59372
    - peopleInHousehold  1    59364 59406
    - edu_level          6    59488 59520
    - age                1    60984 61026
    
    Step:  AIC=59277.26
    result ~ age + townsend + n_cancers + whr + edu_level + peopleInHousehold + 
        ethnicity_sim + house_type + Dem_diag + copd
    
                        Df Deviance   AIC
    <none>                    59235 59277
    + diabetes           1    59235 59279
    + whistling          1    59235 59279
    + Population         1    59235 59279
    + sex                1    59235 59279
    + highBP             2    59235 59281
    - ethnicity_sim      1    59251 59291
    - n_cancers          1    59256 59296
    - house_type         5    59271 59303
    - whr                1    59279 59319
    - Dem_diag           1    59280 59320
    - copd               2    59314 59352
    - townsend           1    59331 59371
    - peopleInHousehold  1    59365 59405
    - edu_level          6    59489 59519
    - age                1    60988 61028
    
    Call:
    glm(formula = result ~ age + townsend + n_cancers + whr + edu_level + 
        peopleInHousehold + ethnicity_sim + house_type + Dem_diag + 
        copd, family = "binomial", data = ukb_dt_subset)
    
    Deviance Residuals: 
        Min       1Q   Median       3Q      Max  
    -3.9201  -0.7631  -0.6030  -0.4154   2.3770  
    
    Coefficients:
                                                                       Estimate
    (Intercept)                                                        1.835517
    age                                                               -0.058971
    townsend                                                           0.035048
    n_cancers                                                         -0.150602
    whr                                                                0.756318
    edu_levelCollege or University degree                             -0.100066
    edu_levelCSEs or equivalent                                        0.377208
    edu_levelNone of the above                                         0.415453
    edu_levelNVQ or HND or HNC or equivalent                           0.292291
    edu_levelO levels/GCSEs or equivalent                              0.107563
    edu_levelOther professional qualifications eg: nursing, teaching   0.076880
    peopleInHousehold                                                  0.087956
    ethnicity_simwhite                                                -0.122535
    house_typeA house or bungalow                                      0.170710
    house_typeCare home                                               -9.898458
    house_typeMobile or temporary structure (i.e. caravan)            -0.614450
    house_typeNone of the above                                       -0.231113
    house_typeSheltered accommodation                                 -9.952621
    Dem_diagTrue                                                       0.778460
    copdFalse                                                         -0.223718
    copdTrue                                                          -0.302351
                                                                     Std. Error
    (Intercept)                                                        0.147972
    age                                                                0.001414
    townsend                                                           0.003576
    n_cancers                                                          0.033768
    whr                                                                0.114238
    edu_levelCollege or University degree                              0.070419
    edu_levelCSEs or equivalent                                        0.071426
    edu_levelNone of the above                                         0.067193
    edu_levelNVQ or HND or HNC or equivalent                           0.067167
    edu_levelO levels/GCSEs or equivalent                              0.067154
    edu_levelOther professional qualifications eg: nursing, teaching   0.065462
    peopleInHousehold                                                  0.008105
    ethnicity_simwhite                                                 0.031060
    house_typeA house or bungalow                                      0.038758
    house_typeCare home                                              196.967686
    house_typeMobile or temporary structure (i.e. caravan)             0.331432
    house_typeNone of the above                                        0.204428
    house_typeSheltered accommodation                                 50.828750
    Dem_diagTrue                                                       0.110515
    copdFalse                                                          0.025793
    copdTrue                                                           0.171656
                                                                     z value
    (Intercept)                                                       12.405
    age                                                              -41.717
    townsend                                                           9.800
    n_cancers                                                         -4.460
    whr                                                                6.621
    edu_levelCollege or University degree                             -1.421
    edu_levelCSEs or equivalent                                        5.281
    edu_levelNone of the above                                         6.183
    edu_levelNVQ or HND or HNC or equivalent                           4.352
    edu_levelO levels/GCSEs or equivalent                              1.602
    edu_levelOther professional qualifications eg: nursing, teaching   1.174
    peopleInHousehold                                                 10.851
    ethnicity_simwhite                                                -3.945
    house_typeA house or bungalow                                      4.405
    house_typeCare home                                               -0.050
    house_typeMobile or temporary structure (i.e. caravan)            -1.854
    house_typeNone of the above                                       -1.131
    house_typeSheltered accommodation                                 -0.196
    Dem_diagTrue                                                       7.044
    copdFalse                                                         -8.674
    copdTrue                                                          -1.761
                                                                     Pr(>|z|)    
    (Intercept)                                                       < 2e-16 ***
    age                                                               < 2e-16 ***
    townsend                                                          < 2e-16 ***
    n_cancers                                                        8.20e-06 ***
    whr                                                              3.58e-11 ***
    edu_levelCollege or University degree                              0.1553    
    edu_levelCSEs or equivalent                                      1.28e-07 ***
    edu_levelNone of the above                                       6.29e-10 ***
    edu_levelNVQ or HND or HNC or equivalent                         1.35e-05 ***
    edu_levelO levels/GCSEs or equivalent                              0.1092    
    edu_levelOther professional qualifications eg: nursing, teaching   0.2402    
    peopleInHousehold                                                 < 2e-16 ***
    ethnicity_simwhite                                               7.98e-05 ***
    house_typeA house or bungalow                                    1.06e-05 ***
    house_typeCare home                                                0.9599    
    house_typeMobile or temporary structure (i.e. caravan)             0.0637 .  
    house_typeNone of the above                                        0.2582    
    house_typeSheltered accommodation                                  0.8448    
    Dem_diagTrue                                                     1.87e-12 ***
    copdFalse                                                         < 2e-16 ***
    copdTrue                                                           0.0782 .  
    ---
    Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
    
    (Dispersion parameter for binomial family taken to be 1)
    
        Null deviance: 62920  on 56589  degrees of freedom
    Residual deviance: 59235  on 56569  degrees of freedom
    AIC: 59277
    
    Number of Fisher Scoring iterations: 10
    



Paracetamol infection & death, using optimised parameters


```r
%%R
infection_para.b = glm(formula = result ~ age + townsend + n_cancers + whr + edu_level + 
    peopleInHousehold + ethnicity_sim + house_type + Dem_diag + 
    copd+paracetamol, family = "binomial", data = ukb_dt)

death_para.b = glm(formula = covid_death ~ sex + age + townsend + n_cancers + 
    diabetes + whr + edu_level + ethnicity_sim + highBP + Dem_diag + 
    copd + whistling+paracetamol, family = "binomial", data = subset(ukb_dt, result == 1))
```


```r
%%R
summary(infection_para.b)

```


    
    Call:
    glm(formula = result ~ age + townsend + n_cancers + whr + edu_level + 
        peopleInHousehold + ethnicity_sim + house_type + Dem_diag + 
        copd + paracetamol, family = "binomial", data = ukb_dt)
    
    Deviance Residuals: 
        Min       1Q   Median       3Q      Max  
    -3.9224  -0.7658  -0.6049  -0.4078   2.3983  
    
    Coefficients:
                                                                       Estimate
    (Intercept)                                                        1.858413
    age                                                               -0.058775
    townsend                                                           0.034743
    n_cancers                                                         -0.147541
    whr                                                                0.780223
    edu_levelCollege or University degree                             -0.140199
    edu_levelCSEs or equivalent                                        0.353640
    edu_levelNone of the above                                         0.388212
    edu_levelNVQ or HND or HNC or equivalent                           0.266646
    edu_levelO levels/GCSEs or equivalent                              0.076547
    edu_levelOther professional qualifications eg: nursing, teaching   0.041169
    peopleInHousehold                                                  0.088031
    ethnicity_simwhite                                                -0.123824
    house_typeA house or bungalow                                      0.161684
    house_typeCare home                                               -9.912839
    house_typeMobile or temporary structure (i.e. caravan)            -0.674938
    house_typeNone of the above                                       -0.236542
    house_typeSheltered accommodation                                 -9.970525
    Dem_diagTrue                                                       0.789763
    copdFalse                                                         -0.227376
    copdTrue                                                          -0.241590
    paracetamolTRUE                                                   -0.068630
                                                                     Std. Error
    (Intercept)                                                        0.145187
    age                                                                0.001387
    townsend                                                           0.003510
    n_cancers                                                          0.033291
    whr                                                                0.112176
    edu_levelCollege or University degree                              0.068841
    edu_levelCSEs or equivalent                                        0.069813
    edu_levelNone of the above                                         0.065660
    edu_levelNVQ or HND or HNC or equivalent                           0.065651
    edu_levelO levels/GCSEs or equivalent                              0.065649
    edu_levelOther professional qualifications eg: nursing, teaching   0.063990
    peopleInHousehold                                                  0.007916
    ethnicity_simwhite                                                 0.030279
    house_typeA house or bungalow                                      0.037628
    house_typeCare home                                              196.967686
    house_typeMobile or temporary structure (i.e. caravan)             0.330541
    house_typeNone of the above                                        0.194043
    house_typeSheltered accommodation                                 50.762025
    Dem_diagTrue                                                       0.108405
    copdFalse                                                          0.025495
    copdTrue                                                           0.166678
    paracetamolTRUE                                                    0.024111
                                                                     z value
    (Intercept)                                                       12.800
    age                                                              -42.389
    townsend                                                           9.899
    n_cancers                                                         -4.432
    whr                                                                6.955
    edu_levelCollege or University degree                             -2.037
    edu_levelCSEs or equivalent                                        5.066
    edu_levelNone of the above                                         5.912
    edu_levelNVQ or HND or HNC or equivalent                           4.062
    edu_levelO levels/GCSEs or equivalent                              1.166
    edu_levelOther professional qualifications eg: nursing, teaching   0.643
    peopleInHousehold                                                 11.121
    ethnicity_simwhite                                                -4.089
    house_typeA house or bungalow                                      4.297
    house_typeCare home                                               -0.050
    house_typeMobile or temporary structure (i.e. caravan)            -2.042
    house_typeNone of the above                                       -1.219
    house_typeSheltered accommodation                                 -0.196
    Dem_diagTrue                                                       7.285
    copdFalse                                                         -8.918
    copdTrue                                                          -1.449
    paracetamolTRUE                                                   -2.846
                                                                     Pr(>|z|)    
    (Intercept)                                                       < 2e-16 ***
    age                                                               < 2e-16 ***
    townsend                                                          < 2e-16 ***
    n_cancers                                                        9.34e-06 ***
    whr                                                              3.52e-12 ***
    edu_levelCollege or University degree                             0.04170 *  
    edu_levelCSEs or equivalent                                      4.07e-07 ***
    edu_levelNone of the above                                       3.37e-09 ***
    edu_levelNVQ or HND or HNC or equivalent                         4.87e-05 ***
    edu_levelO levels/GCSEs or equivalent                             0.24361    
    edu_levelOther professional qualifications eg: nursing, teaching  0.51998    
    peopleInHousehold                                                 < 2e-16 ***
    ethnicity_simwhite                                               4.32e-05 ***
    house_typeA house or bungalow                                    1.73e-05 ***
    house_typeCare home                                               0.95986    
    house_typeMobile or temporary structure (i.e. caravan)            0.04116 *  
    house_typeNone of the above                                       0.22284    
    house_typeSheltered accommodation                                 0.84428    
    Dem_diagTrue                                                     3.21e-13 ***
    copdFalse                                                         < 2e-16 ***
    copdTrue                                                          0.14722    
    paracetamolTRUE                                                   0.00442 ** 
    ---
    Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
    
    (Dispersion parameter for binomial family taken to be 1)
    
        Null deviance: 65094  on 58351  degrees of freedom
    Residual deviance: 61274  on 58330  degrees of freedom
      (2094 observations deleted due to missingness)
    AIC: 61318
    
    Number of Fisher Scoring iterations: 10
    




```r
%%R
nrow(subset(ukb_dt, 
    result == 1))
```


    [1] 14877




```r
%%R
infection_ibu.b = glm(formula = result ~ age + townsend + n_cancers + whr + edu_level + 
    peopleInHousehold + ethnicity_sim + house_type + Dem_diag + 
    copd+ibuprofen, family = "binomial", data = ukb_dt)

death_ibu.b = glm(formula = covid_death ~ sex + age + townsend + n_cancers + 
    diabetes + whr + edu_level + ethnicity_sim + highBP + Dem_diag + 
    copd + whistling+ibuprofen, family = "binomial", data = ukb_dt)
```


```r
%%R
summary(infection_ibu.b)

```


    
    Call:
    glm(formula = result ~ age + townsend + n_cancers + whr + edu_level + 
        peopleInHousehold + ethnicity_sim + house_type + Dem_diag + 
        copd + ibuprofen, family = "binomial", data = ukb_dt)
    
    Deviance Residuals: 
        Min       1Q   Median       3Q      Max  
    -3.9181  -0.7656  -0.6051  -0.4081   2.3733  
    
    Coefficients:
                                                                       Estimate
    (Intercept)                                                        1.846259
    age                                                               -0.058779
    townsend                                                           0.034277
    n_cancers                                                         -0.149627
    whr                                                                0.785548
    edu_levelCollege or University degree                             -0.140232
    edu_levelCSEs or equivalent                                        0.349223
    edu_levelNone of the above                                         0.381539
    edu_levelNVQ or HND or HNC or equivalent                           0.262871
    edu_levelO levels/GCSEs or equivalent                              0.075372
    edu_levelOther professional qualifications eg: nursing, teaching   0.039300
    peopleInHousehold                                                  0.087981
    ethnicity_simwhite                                                -0.122746
    house_typeA house or bungalow                                      0.160036
    house_typeCare home                                               -9.905102
    house_typeMobile or temporary structure (i.e. caravan)            -0.679644
    house_typeNone of the above                                       -0.235934
    house_typeSheltered accommodation                                 -9.969906
    Dem_diagTrue                                                       0.786887
    copdFalse                                                         -0.225795
    copdTrue                                                          -0.242391
    ibuprofenTRUE                                                     -0.030299
                                                                     Std. Error
    (Intercept)                                                        0.145668
    age                                                                0.001392
    townsend                                                           0.003506
    n_cancers                                                          0.033285
    whr                                                                0.112167
    edu_levelCollege or University degree                              0.068841
    edu_levelCSEs or equivalent                                        0.069789
    edu_levelNone of the above                                         0.065618
    edu_levelNVQ or HND or HNC or equivalent                           0.065634
    edu_levelO levels/GCSEs or equivalent                              0.065644
    edu_levelOther professional qualifications eg: nursing, teaching   0.063983
    peopleInHousehold                                                  0.007916
    ethnicity_simwhite                                                 0.030282
    house_typeA house or bungalow                                      0.037619
    house_typeCare home                                              196.967686
    house_typeMobile or temporary structure (i.e. caravan)             0.330583
    house_typeNone of the above                                        0.193995
    house_typeSheltered accommodation                                 50.825739
    Dem_diagTrue                                                       0.108404
    copdFalse                                                          0.025490
    copdTrue                                                           0.166731
    ibuprofenTRUE                                                      0.028372
                                                                     z value
    (Intercept)                                                       12.674
    age                                                              -42.221
    townsend                                                           9.778
    n_cancers                                                         -4.495
    whr                                                                7.003
    edu_levelCollege or University degree                             -2.037
    edu_levelCSEs or equivalent                                        5.004
    edu_levelNone of the above                                         5.814
    edu_levelNVQ or HND or HNC or equivalent                           4.005
    edu_levelO levels/GCSEs or equivalent                              1.148
    edu_levelOther professional qualifications eg: nursing, teaching   0.614
    peopleInHousehold                                                 11.114
    ethnicity_simwhite                                                -4.053
    house_typeA house or bungalow                                      4.254
    house_typeCare home                                               -0.050
    house_typeMobile or temporary structure (i.e. caravan)            -2.056
    house_typeNone of the above                                       -1.216
    house_typeSheltered accommodation                                 -0.196
    Dem_diagTrue                                                       7.259
    copdFalse                                                         -8.858
    copdTrue                                                          -1.454
    ibuprofenTRUE                                                     -1.068
                                                                     Pr(>|z|)    
    (Intercept)                                                       < 2e-16 ***
    age                                                               < 2e-16 ***
    townsend                                                          < 2e-16 ***
    n_cancers                                                        6.95e-06 ***
    whr                                                              2.50e-12 ***
    edu_levelCollege or University degree                              0.0416 *  
    edu_levelCSEs or equivalent                                      5.62e-07 ***
    edu_levelNone of the above                                       6.08e-09 ***
    edu_levelNVQ or HND or HNC or equivalent                         6.20e-05 ***
    edu_levelO levels/GCSEs or equivalent                              0.2509    
    edu_levelOther professional qualifications eg: nursing, teaching   0.5391    
    peopleInHousehold                                                 < 2e-16 ***
    ethnicity_simwhite                                               5.05e-05 ***
    house_typeA house or bungalow                                    2.10e-05 ***
    house_typeCare home                                                0.9599    
    house_typeMobile or temporary structure (i.e. caravan)             0.0398 *  
    house_typeNone of the above                                        0.2239    
    house_typeSheltered accommodation                                  0.8445    
    Dem_diagTrue                                                     3.90e-13 ***
    copdFalse                                                         < 2e-16 ***
    copdTrue                                                           0.1460    
    ibuprofenTRUE                                                      0.2856    
    ---
    Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
    
    (Dispersion parameter for binomial family taken to be 1)
    
        Null deviance: 65094  on 58351  degrees of freedom
    Residual deviance: 61281  on 58330  degrees of freedom
      (2094 observations deleted due to missingness)
    AIC: 61325
    
    Number of Fisher Scoring iterations: 10
    




```r
%%R

infection_ibu.b_odds = data.frame(name=row.names(summary(infection_ibu.b)$coefficients),
                           OR = exp(summary(infection_ibu.b)$coefficients[,1]),
                           lower = exp(summary(infection_ibu.b)$coefficients[,1] - summary(infection_ibu.b)$coefficients[,2]),
                           upper = exp(summary(infection_ibu.b)$coefficients[,1] + summary(infection_ibu.b)$coefficients[,2]),
                           p_value = summary(infection_ibu.b)$coefficients[,4])


write.csv(infection_ibu.b_odds,"data_output/infection_ibu.b_odds.csv", row.names=TRUE)

infection_ibu.b_odds$significance = "p-value > 0.05"
infection_ibu.b_odds$significance[infection_ibu.b_odds$p_value < 0.05] <- "p-value < 0.05"
infection_ibu.b_odds$variables = row.names(infection_ibu.b_odds)

library(ggplot2)

infection_ibu.b_odds_df = subset(infection_ibu.b_odds, name != "(Intercept)" & 
                                 name != "house_typeCare home" & 
                                name != "house_typeSheltered accommodation")

ggplot(infection_ibu.b_odds_df, 
       aes(x=name, y=OR)) + 
  geom_errorbar(aes(ymin=lower, ymax=upper),
                width=0,                    # Width of the error bars
                position=position_dodge(.9), color = "#939598", size = 1) +
  geom_point(aes(color = significance, fill = significance),shape=21, size = 2) +
  theme_classic() + 
  geom_hline(yintercept = 1, linetype="dotted", linetype = "longdash") +
  coord_flip()+ylab("Odds ratio") + 
  xlab("") + 
  scale_colour_manual(name = "grp",values = c("#ED2024","#939598")) + 
  scale_fill_manual(name = "grp",values = c("#ED2024","#939598")) +
  theme(legend.position = "none")

ggsave('fig/infection_ibu.pdf', width = 6, height = 3.5)

```


```r
%%R

infection_para.b_odds = data.frame(name=row.names(summary(infection_para.b)$coefficients),
                           OR = exp(summary(infection_para.b)$coefficients[,1]),
                           lower = exp(summary(infection_para.b)$coefficients[,1] - summary(infection_para.b)$coefficients[,2]),
                           upper = exp(summary(infection_para.b)$coefficients[,1] + summary(infection_para.b)$coefficients[,2]),
                           p_value = summary(infection_para.b)$coefficients[,4])


write.csv(infection_para.b_odds,"data_output/infection_para.b_odds.csv", row.names=TRUE)

infection_para.b_odds$significance = "p-value > 0.05"
infection_para.b_odds$significance[infection_para.b_odds$p_value <= 0.05] <- "p-value < 0.05"
infection_para.b_odds$variables = row.names(infection_para.b_odds)

library(ggplot2)

infection_para.b_odds_df = subset(infection_para.b_odds, name != "(Intercept)" & 
                                 name != "house_typeCare home" & 
                                name != "house_typeSheltered accommodation")
ggplot(infection_para.b_odds_df, 
       aes(x=name, y=OR)) + 
  geom_errorbar(aes(ymin=lower, ymax=upper),
                width=0,                    # Width of the error bars
                position=position_dodge(.9), color = "#939598", size = 1) +
  geom_point(aes(color = significance, fill = significance),shape=21, size = 2) +
  theme_classic() + 
  geom_hline(yintercept = 1, linetype="dotted", linetype = "longdash") +
  coord_flip()+ylab("Odds ratio") + 
  xlab("") + 
  scale_colour_manual(name = "grp",values = c("#ED2024","#939598")) + 
  scale_fill_manual(name = "grp",values = c("#ED2024","#939598")) +
  theme(legend.position = "none")

ggsave('fig/infection_para.pdf', width = 6, height = 3.5)

```
