# Splits & Iterators



### Lab 4 outline

1. Randomly split data into 2 equal parts (calibration & validation samples)
2. Introduction to MplusAutomation with iterators
3. Dealing with large data

____________________________________

### Getting started - following the routine...

1. Create an R-Project
2. Install packages ($\color{red}{\text{ONLY IF NEEDED}}$)
3. Load packages

### R-Project instructions:

1. click "NEW PROJECT" (upper right corner of window)
2. choose option "NEW DIRECTORY"
3. choose location of project (on desktop OR in a designated class folder)

Within R-studio under the files pane (bottom right):

1. click "New Folder" and name folder "data"
2. click "New Folder" and name folder "efa_mplus"
3. click "New Folder" and name folder "figures"

____________________________________

New packages this week:

- {`janitor`}
- {`haven`}

____________________________________

## Begin 

____________________________________

DATA SOURCE: This lab exercise utilizes the NCES public-use dataset: Education Longitudinal Study of 2002 (Lauff & Ingels, 2014) [$\color{blue}{\text{See website: nces.ed.gov}}$](https://nces.ed.gov/surveys/els2002/avail_data.asp)

____________________________________

### loading packages...


```r
library(janitor)
library(tidyverse)
library(haven)
library(MplusAutomation)
library(rhdf5)
library(here)
library(corrplot)
```

### read in the raw dataset 


```r
lab_data <- read_spss("https://garberadamc.github.io/project-site/data/els_sub1_spss.sav")
```

### create a subset of the dataset called `school_trouble`

```r
school_trouble <- lab_data %>% 
  select(41:55)
```

### make a new codebook from the `school_trouble` subset

```r
sjPlot::view_df(school_trouble)
```

### write a `CSV` datafile

```r
write_csv(school_trouble, here("data", "school_trouble_data.csv"))
```

### read the unlabeled data back into R

```r
trouble_data <- read_csv(here("data", "school_trouble_data.csv"))
```

### check items to see if reverse coding is needed 

```r
cor_matrix <- cor(trouble_data, use = "pairwise.complete.obs")

corrplot(cor_matrix, method="circle",
         type = "upper")
```

____________________________________
#
## Randomly split a sample into 2 equal parts
#
____________________________________

### find the size of half of original sample. 
### The "floor()" function helps with rounding

```r
smp_size <- floor(0.50 * nrow(trouble_data))
```

### set the seed to make your partition reproducible

```r
set.seed(123)
```

### the function `sample()` will pick at random the values of the specified number

```r
calibrate_smp <- sample(seq_len(nrow(trouble_data)), size = smp_size)
```

### create two samples called "calibrate" & "validate"

```r
calibrate <- trouble_data[calibrate_smp, ]
validate <- trouble_data[-calibrate_smp, ]
```

### Let's run an EFA with the "calibrate" sample

```r
m_efa_1  <- mplusObject(
  TITLE = "School Trouble EFA - LAB 4 DEMO", 
  VARIABLE = 
    "usevar = BYS22A-BYS24G;", 
  
  ANALYSIS = 
    "type = efa 1 5;   
     estimator = mlr;  
     parallel=50; ! run parallel analysis",
  
  MODEL = "" ,
  
  PLOT = "type = plot3;",
  OUTPUT = "sampstat;",
  
  usevariables = colnames(calibrate), 
  rdata = calibrate)

m_efa_1_fit <- mplusModeler(m_efa_1, 
                            dataout=here("efa_mplus", "lab4_efa1_trouble.dat"),
                            modelout=here("efa_mplus", "lab4_efa1_trouble.inp"),
                            check=TRUE, run = TRUE, hashfilename = FALSE)
```

### Plot Parallel Analysis & Eigenvalues

### read into R an Mplus output file

```r
efa_summary <- readModels(here("efa_mplus", "lab4_efa1_trouble.out"))
```

### extract relavent data & prepare dataframe for plot

```r
x <- list(EFA=efa_summary[["gh5"]][["efa"]][["eigenvalues"]], 
          Parallel=efa_summary[["gh5"]][["efa"]][["parallel_average"]])

plot_data <- as_data_frame(x)
plot_data <- cbind(Factor = paste0(1:nrow(plot_data)), plot_data)

plot_data <- plot_data %>% 
  mutate(Factor = fct_inorder(Factor))
```

### pivot the dataframe to "long" format

```r
plot_data_long <- plot_data %>% 
  pivot_longer(EFA:Parallel,               # The columns I'm gathering together
               names_to = "Analysis",      # new column name for existing names
               values_to = "Eigenvalues")  # new column name to store values
```

### plot using ggplot

```r
plot_data_long %>% 
  ggplot(aes(y=Eigenvalues,
             x=Factor,
             group=Analysis,
             color=Analysis)) +
  geom_point() + 
  geom_line() + 
  theme_minimal()
```

### save figure to the designated folder

```r
ggsave(here("figures", "eigenvalue_elbow_rplot.png"), dpi=300, height=5, width=7, units="in")
```




____________________________________

## Introduction to MplusAutomation with iterators 

____________________________________

### Alternate way to run an EFA with the "calibrate" sample

```r
m_efa  <- lapply(1:5, function(k) {
  m_efa2  <- mplusObject(
    TITLE = "School Trouble EFA - LAB 4 DEMO", 
    VARIABLE = 
      "usevar = BYS22A-BYS24G;", 
    
    ANALYSIS = 
      paste("type=efa", k, k), 
    
    MODEL = "" ,
    
    PLOT = "type = plot3;",
    OUTPUT = "sampstat;",
    
    usevariables = colnames(calibrate), 
    rdata = calibrate)
  
  m_efa_2_fit <- mplusModeler(m_efa2, 
                              dataout=sprintf(here("efa_mplus2", "efa_trouble.dat"), k),
                              modelout=sprintf(here("efa_mplus2", "efa_%d_trouble.inp"), k),
                              check=TRUE, run = TRUE, hashfilename = FALSE)
})
```


____________________________________

## Cleaning & subsetting large datasets 

____________________________________

### reading SPSS files is a lot slower than reading CSV formatted files

```r
hsls_raw <- read_spss(here("data", "hsls_16_student_sub_v1.sav"))
```

### make all column names "lower_snake_case" style

```r
hsls_tidy <- hsls_raw %>% 
  clean_names()
```

### select usig the starts_with() function

```r
hsls_x1 <- hsls_tidy %>% 
  select(starts_with("x1")) # columns with first 2 characters "x1"
```

### select using the end_with() function

```r
hsls_not_sex <- hsls_tidy %>% 
  select(!ends_with("sex")) # columns that do NOT end with "sex"
```

### select using the end_with() function

```r
hsls_science <- hsls_tidy %>% 
  select(contains("sci")) # columns that contain characters "sci"

hsls_math <- hsls_tidy %>% 
  select(contains(c("mth" , "math"))) # columns that contain "mth" or "math"
```

### combine different select() arguements

```r
hsls_math_sci <- hsls_tidy %>% 
  select(contains(c("mth" , "math", "sci"))) %>%
  select(!starts_with("x1")) %>% 
  select(!ends_with("sex"))
```

### remove characters from the variable names that are greater than 8 characters 

```r
names(hsls_math_sci) = str_sub(names(hsls_math_sci), 1, 8)
```

### check if culumn names are unique

```r
test.unique <- function(df) { ## function to identify unique columns
  
  length1 <- length(colnames(df))
  length2 <- length(unique(colnames(df))) 
  if (length1 - length2 > 0 ) {
    
  print(paste("There are", length1 - length2, " duplicates", sep=" ")) 
    }
} 

test.unique(hsls_math_sci)
```

### locate duplicates (this will find the column of the first duplicate)

```r
anyDuplicated(colnames(hsls_math_sci))

names(hsls_math_sci)
```


### Other functions to consider from the "stringr" package (part of tidyverse): 

- str_remove()
- str_replace() # replace one string pattern with another
- str_match()
- str_pad()     # to remove spaces 
- str_count()
- str_detect()
- str_dup()
- str_extract_all()

____________________________________

## End of lab 4 exercise

____________________________________


## References

Hallquist, M. N., & Wiley, J. F. (2018). MplusAutomation: An R Package for Facilitating Large-Scale Latent Variable Analyses in Mplus. Structural equation modeling: a multidisciplinary journal, 25(4), 621-638.

Horst, A. (2020). Course & Workshop Materials. GitHub Repositories, https://https://allisonhorst.github.io/

Muth??n, L.K. and Muth??n, B.O. (1998-2017).  Mplus User???s Guide.  Eighth Edition. Los Angeles, CA: Muth??n & Muth??n

R Core Team (2017). R: A language and environment for statistical computing. R Foundation for Statistical Computing, Vienna, Austria. URL http://www.R-project.org/

Wickham et al., (2019). Welcome to the tidyverse. Journal of Open Source Software, 4(43), 1686, https://doi.org/10.21105/joss.01686
