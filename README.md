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
## Step 2: Load in your ICD table and organize in the same columns as the ICD dataframe example. 
```{r}
ICDs <- read.csv("https://raw.githubusercontent.com/PheWAS/MCA/refs/heads/main/data/sample_ICD_data.csv",quote="\"")
```
#Columns inclde person ID, ICD code, whether that ICD code is an ICD9 or ICD10 code, and the age of the person when billed with the ICD code. 
```{r}
head(Fake_ICD_MCA_GitHub_Data)
```
## Step 3: Load in CM_phecodeX_R_map and merge with your ICD file to create your phecodeX table 
```{r}
Fake_Data_CM_PhecodeX_table = merge(CM_phecodeX_R_map, Fake_ICD_MCA_GitHub_Data, by=c("code", "vocabulary_id"))
head(Fake_Data_CM_PhecodeX_table)
```
```{r}
#Group the phecodeX table so it's organized by unique person/phecode pairs and gives unique age counts (how many distinct ages that individual was billed with that ICD code, which is mapped to a phecode)
Fake_Data_CM_PhecodeX_table = Fake_Data_CM_PhecodeX_table[,.(uniq_age=uniqueN(AGE_AT_EVENT)),by=c("ID","phecode")]
head(Fake_Data_CM_PhecodeX_table)
```
## Step 4: Filter down to individuals with at least two instances of a phecode at different ages 
```{r}
Fake_Data_CM_PhecodeX_table = Fake_Data_CM_PhecodeX_table %>% filter(uniq_age >= "2")
head(Fake_Data_CM_PhecodeX_table)
```

## Step 5: Identify the MCA Phecode population (those with at least two instances of the specific MCA Phecode: CM_766) and create dataframe with MCA Phecode population unique IDs 
```{r}
Fake_Data_MCA_Phecode = Fake_Data_CM_PhecodeX_table[ Fake_Data_CM_PhecodeX_table$phecode=="CM_776"]
Fake_Data_MCA_Phecode
```
```{r}
#Create dataframe that has the unique IDs of individuals in the MCA Phecode population 
Fake_Data_MCA_Phecode_Unique_IDs_Only <- Fake_Data_MCA_Phecode %>%
  distinct(ID)
Fake_Data_MCA_Phecode_Unique_IDs_Only
```
## Step 6: To identify the MCA All population create an Organ_System column which indicates which organ system the phecode corresponds to 
```{r}
Fake_Data_CM_PhecodeX_table <- Fake_Data_CM_PhecodeX_table %>%
  mutate(Organ_System = case_when(
    grepl("^CM_750", phecode) ~ "Nervous",
    grepl("^CM_751", phecode) ~ "Eye",
    grepl("^CM_752", phecode) ~ "Ear",
    grepl("^CM_753", phecode) ~ "FaceNeck",
    grepl("^CM_754", phecode) ~ "TongueMouthPharynx",
    grepl("^CM_755", phecode) ~ "SkullFaceJaw",
    grepl("^CM_757", phecode) ~ "UpperGI",
    grepl("^CM_758", phecode) ~ "Digestive",
    grepl("^CM_760", phecode) ~ "Genital",
    grepl("^CM_761", phecode) ~ "Urinary",
    grepl("^CM_762", phecode) ~ "Respiratory",
    grepl("^CM_763", phecode) ~ "Heart",
    grepl("^CM_764", phecode) ~ "Circulatory",
    grepl("^CM_765", phecode) ~ "Hip",
    grepl("^CM_766", phecode) ~ "Feet",
    grepl("^CM_767", phecode) ~ "Limb",
    grepl("^CM_768", phecode) ~ "ChestBonyThorax",
    grepl("^CM_769", phecode) ~ "Spine",
    grepl("^CM_770", phecode) ~ "Musculoskeletal",
    grepl("^CM_771", phecode) ~ "Integument",
    grepl("^CM_772", phecode) ~ "EndocrineGlands",
    grepl("^CM_773", phecode) ~ "Spleen",
    grepl("^CM_774", phecode) ~ "SitusInversus",
    TRUE ~ NA_character_ # default value if no match
  ))

print(Fake_Data_CM_PhecodeX_table)
```
## Step 7: To identify the MCA All population, re-group by IDs and filter to those who have a phecode in at least two unique organ systems and create dataframe with MCA All population unique IDs 
```{r}
Fake_Data_MCA_All <- Fake_Data_CM_PhecodeX_table %>%
  group_by(ID) %>%
  filter(n_distinct(Organ_System) > 1) %>% 
  ungroup()

print(Fake_Data_MCA_All)
```
```{r}
Fake_Data_MCA_All_Unique_IDs_Only <- Fake_Data_MCA_All %>%
  distinct(ID)
Fake_Data_MCA_All_Unique_IDs_Only
```
## Step 8: Combine MCA Phecode and MCA All populations into the MCA Total population and create dataframe with MCA Total population unique IDs 
```{r}
#Combine and then pull out distinct IDs
Fake_Data_MCA_Total_Overlap = rbind(Fake_Data_MCA_Phecode_Unique_IDs_Only, Fake_Data_MCA_All_Unique_IDs_Only)

Fake_Data_MCA_Total_Unique_IDs <- Fake_Data_MCA_Total_Overlap %>%
  distinct(ID)
Fake_Data_MCA_Total_Unique_IDs
```