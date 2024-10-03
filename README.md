# Tutorial

Implimenting an EHR phenotype for multiple congential anomalies (MCA)

## Introduction

In this tutorial, you will learn how to impliment the MCA algorith as described in a forthcoming publication by Brokamp et. al.

## Prerequisites

To impliment this method, you will need to have a file of ICD codes and patient identifiers. This tutorial includes a file with sythetic data which will allow you to try the method out.


## Step 1: Setting Up Your Environment

1. **Create a new directory for your project**:

   ```bash
   mkdir flask-tutorial
   cd flask-tutorial
  
sample_data <- read.csv("https://github.com/PheWAS/MCA/blob/main/data/sample_ICD_data.csv")

sample_data
   
