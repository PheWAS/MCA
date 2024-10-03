# Tutorial

Implimenting an EHR phenotype for multiple congential anomalies (MCA)

## Introduction

In this tutorial, you will learn how to impliment the MCA algorith as described in a forthcoming publication by Brokamp et. al.

## Prerequisites

To impliment this method, you will need to have a file of ICD codes and patient identifiers. This tutorial includes a file with sythetic data which will allow you to try the method out.


## Step 1: Load necessary packages and CM_phecodeX_R_map 
```{r}
library(dplyr)
CM_phecodeX_R_map <- read.csv("https://raw.githubusercontent.com/PheWAS/MCA/refs/heads/main/data/CM_phecodeX_R_map.csv",quote="\"")
```
#2. Load in your ICD table and organize in the same columns as the ICD dataframe example. 

ICDs <- read.csv("https://raw.githubusercontent.com/PheWAS/MCA/refs/heads/main/data/sample_ICD_data.csv",quote="\"")

#Columns inclde person ID, ICD code, whether that ICD code is an ICD9 or ICD10 code, and the age of the person when billed with the ICD code. 
```{r}
head(Fake_ICD_MCA_GitHub_Data)
```
#3. Load in CM_phecodeX_R_map and merge with your ICD file to create your phecodeX table 
```{r}
Fake_Data_CM_PhecodeX_table = merge(CM_phecodeX_R_map, Fake_ICD_MCA_GitHub_Data, by=c("code", "vocabulary_id"))
head(Fake_Data_CM_PhecodeX_table)
```
```{r}
#Group the phecodeX table so it's organized by unique person/phecode pairs and gives unique age counts (how many distinct ages that individual was billed with that ICD code, which is mapped to a phecode)
Fake_Data_CM_PhecodeX_table = Fake_Data_CM_PhecodeX_table[,.(uniq_age=uniqueN(AGE_AT_EVENT)),by=c("ID","phecode")]
head(Fake_Data_CM_PhecodeX_table)
```
#4.Filter down to individuals with at least two instances of a phecode at different ages 
```{r}
Fake_Data_CM_PhecodeX_table = Fake_Data_CM_PhecodeX_table %>% filter(uniq_age >= "2")
head(Fake_Data_CM_PhecodeX_table)
```