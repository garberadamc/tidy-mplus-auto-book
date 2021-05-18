# Introduction {#intro}



______________________________________________

## Guidlines
   
0.  guideline for MplusAutomation workflow
1.  create an R project in a dedicated project folder (on the desktop or in a class folder)
2.  install & load packages
3.  read in data to R
4.  view data in R
5.  view metadata (from SPSS files)
6.  write .sav / .csv / .dat files
7.  fix character names to have less than 8 character 
8.  filtering rows & selecting columns
9.  change variable classes in R
10.  visualize and explore data
11. introduction to mplusObjects

______________________________________________   

## Preparing to work with MplusAutomation 

1. R PROJECTS: We will use R Projects for **ALL** labs & assignments. 
- This is because `MplusAutomation` involves specifying many filepaths.
2. THE {`here`} package: To make filepaths unbreakable (reproducible) 
- The same code will work across operating systems
3. PROJECT SUB-FOLDERS: Thoughtfully organize files in sub-folders. 
- This is **critical**, by the end of the quarter the number of Mplus files for an assignment will multiply rapidly
4. LOCATION OF PROJECT FOLDERS: on **desktop** or within a **single enclosing folder**. 
- There is a limitation with the "mplusObject" function due to the fact that Mplus only reads the first 90 columns in each line. 

e.g., if/your/filepath/has/many/nested/folders/it/will/be/longer/than/the/90character/limit/data.dat 

______________________________________________

## Tools we will use in lab

<table class="table" style="width: auto !important; ">
 <thead>
  <tr>
   <th style="text-align:left;"> Tool/Package </th>
   <th style="text-align:left;"> Purpose/Utility </th>
   <th style="text-align:left;"> Advantages </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> {MplusAutomation} package </td>
   <td style="text-align:left;"> Current capabilities supporting full SEM modeling </td>
   <td style="text-align:left;"> Flexibility (approaching infinite) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> R Project </td>
   <td style="text-align:left;"> Unbreakable file paths &amp; neatness </td>
   <td style="text-align:left;"> Reproducibility (kindness to your future self) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> {tidyverse} package </td>
   <td style="text-align:left;"> Intuitive/descriptive function names </td>
   <td style="text-align:left;"> Accessibility to new users </td>
  </tr>
  <tr>
   <td style="text-align:left;"> {here} package </td>
   <td style="text-align:left;"> Unbreakable/consistent file paths across OS </td>
   <td style="text-align:left;"> Reproducibility (for Science's sake!) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> {haven} package </td>
   <td style="text-align:left;"> View-able metadata in R from SPSS datafiles </td>
   <td style="text-align:left;"> Getting to know your measures </td>
  </tr>
  <tr>
   <td style="text-align:left;"> {ggplot2} package </td>
   <td style="text-align:left;"> Beautiful, customizable, reproducible figures </td>
   <td style="text-align:left;"> Publication quality data visualizations </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pipe operator (%&gt;%) notation </td>
   <td style="text-align:left;"> Ease of reading/writing scripts </td>
   <td style="text-align:left;"> e.g., first() %&gt;% and_then() %&gt;% and_finally() </td>
  </tr>
</tbody>
</table>

______________________________________________

## Creating an R-Project

1. create a **project folder** (that will enclose all files associated with a given lab or assignment)
- the project folder should be located on the desktop or within a designated class folder
- each lab or assignment should have its own designated project folder
2. create a **new project** (upper right hand corner of the R-studio window)
3. create **two sub-folders** in the project folder, one called "data", and one called "basic_mplus"

______________________________________________

## install the “rhdf5” package to read gh5 files

```r
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
 BiocManager::install("rhdf5")
```


```r
# how to install packages?
install.packages("tidyverse")
```

## load packages

```r
library(tidyverse)
library(haven)
library(here)
library(MplusAutomation)
library(rhdf5)
library(reshape2)
```

## Keyboard shortcuts 

-   ALT + DASH(-)  =  **<-**
- SHIFT + CONTROL  =  **%>%**

______________________________________________

## Read in data

______________________________________________


```r
# object_name <- function1(nested_function2("dataset_name.sav"))

exp_data <- read_spss("https://garberadamc.github.io/project-site/data/els_sub1_spss.sav")
```

______________________________________________

## View dataframe with labels & response scale meta-data 

Note: Use the "print" option to save a PDF as a codebook of metadata.

______________________________________________


```r
# the {haven} package keeps the meta-data from SPSS files

# package_name::function_within_package()

sjPlot::view_df(exp_data)
```

______________________________________________

## Types of data for different tasks

- SAV (e.g., spss_data.sav): this data format is for SPSS files & contains variable labels (meta-data)
- CSV (e.g., r_ready_data.csv): this is the preferable data format for reading into R (no labels)
- DAT (e.g., mplus_data.dat): this is the data format used to read into Mplus (no column names or strings)

NOTE: Mplus also accepts TXT formatted data (e.g., mplus_data.txt)

______________________________________________

## Writing, reading, and converting data between 3 formats 

______________________________________________

Prepare datasets, **remove SPSS labeling**

```r
# write a CSV datafile (preferable format for reading into R, without labels)
write_csv(exp_data, here("exp_lab1_data.csv"))

# write a SPSS datafile (preferable format for reading into SPSS, labels are preserved)
write_sav(exp_data, here("exp_lab1_data.sav"))
```


```r
# read the unlabeled data back into R
nolabel_data <- read_csv(here("exp_lab1_data.csv"))
```


```r
# write a DAT datafile (this function removes header row & converts missing values to non-string)
prepareMplusData(nolabel_data, here("exp_lab1_data.dat"))
```

______________________________________________

## Preparing column-names to be `MplusAutomation` ready 

Task: Make all variable names fit within the 8-character name limit (Mplus) while avoiding duplicates.

______________________________________________

## Renaming columns manually...

```r
# use function: rename(new_name = old_name)
new_names <- nolabel_data %>% 
  rename( school_motiv1 = item1 ,  
          school_motiv2 = item2 ,
          school_motiv3 = item3 ,
          school_comp1  = item4 ,
          school_comp2  = item5 ,
          school_comp3  = item6 ,
          school_belif1 = item7 ,
          school_belif2 = item8 ,
          school_belif3 = item9 )
```

## What do you do if you have a large dataset with many column names that are > 8 characters?

- first, remove all characters greater than 8 using str_sub()
- second, make sure you don't now have duplicate variable names
- third, locate and change all duplicate names


```r
# remove characters from the variable names that are greater than 8 characters 
names(new_names) <- str_sub(names(new_names), 1, 8)

# check if culumn names are unique 
test.unique <- function(df) {  ## function to identify unique columns
  
  length1 <- length(colnames(df))
  length2 <- length(unique(colnames(df)))        
  if (length1 - length2 > 0 ) {
    print(paste("There are", length1 - length2, " duplicates", sep=" "))
  }     
}

test.unique(new_names)

# locate duplicates (this will find the column of the first duplicate)
anyDuplicated(colnames(new_names))
```


______________________________________________

## A note on coding style:

- Naming conventions: **Be consistent!** 
- I use the style lower snake case (e.g., this_is_lower_snake_case)
- Annotate code generously
- Let your code breath: use return often to spread code chunks out vertically (dense paragraphs of code are a headache to look at)

