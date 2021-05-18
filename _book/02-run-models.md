# Run a Simple Model with `MplusAutomation`

______________________________________________

## A minimal example of writing, running, & reading models

______________________________________________

**What does the `mplusObject()` function do:**

1. It generates an Mplus **input** file (does not need full variable name list, its automated for you!)
2. It generates a **datafile** specific to each model 
3. It **runs** or estimates the model (hopefully) producing the correct output. **Always check!**

______________________________________________

## PRACTICE: Using MplusObject() method (`type = BASIC;`)

```r
m_basic  <- mplusObject(
  TITLE = "PRACTICE 01 - Explore TYPE = BASIC", 
  VARIABLE = 
 "usevar=
 item1 item2 item3 item4 item5
 item6 item7 item8 item9 female; 
 
 ! use exclamation symbol to make comments, reminders, or annotations in Mplus files",
  
  ANALYSIS = 
 "type = basic; ",
 
  usevariables = colnames(nolabel_data), 
  rdata = nolabel_data)

m_basic_fit <- mplusModeler(m_basic, 
                            dataout=here("basic_mplus", "basic_Lab1_DEMO.dat"),
                            modelout=here("basic_mplus", "basic_Lab1_DEMO.inp"),
                            check=TRUE, run = TRUE, hashfilename = FALSE)
```

______________________________________________

## PRACTICE SUBSETTING: Now explore descriptives for observations that reported as "female" 

Add line of syntax: "useobs = female == 1;"

\newline


```r
fem_basic  <- mplusObject(
  TITLE = "PRACTICE 02 - Explore female observations only", 
  VARIABLE = 
 "usevar=
 item1 item2 item3 item4 item5
 item6 item7 item8 item9; 
 
 useobs = female == 1; !include observations that report female in analysis",
  
  ANALYSIS = 
 "type = basic;",
 
  usevariables = colnames(nolabel_data), 
  rdata = nolabel_data)

fem_basic_fit <- mplusModeler(fem_basic, 
                            dataout=here("basic_mplus", "fem_basic_Lab1_DEMO.dat"),
                            modelout=here("basic_mplus", "fem_basic_Lab1_DEMO.inp"),
                            check=TRUE, run = TRUE, hashfilename = FALSE)
```

______________________________________________

## PRACTICE: Exploratory Factor Analysis (EFA) 

______________________________________________


```r
## EXPLORATORY FACTOR ANALYSIS LAB DEMONSTRATION

efa_demo  <- mplusObject(
  TITLE = "EXPLORATORY FACTOR ANALYSIS - LAB DEMO", 
  VARIABLE = 
 "usevar=
 item1 item2 item3 item4 item5
 item6 item7 item8 item9;" ,
  
  ANALYSIS = 
 "type = efa 1 5;
  estimator = MLR;
  parallel=50;",
  
  MODEL = "" ,
  
  PLOT = "type = plot3;",
 
  OUTPUT = "sampstat standardized residual modindices (3.84);",
 
  usevariables = colnames(nolabel_data), 
  rdata = nolabel_data)

efa_demo_fit <- mplusModeler(efa_demo, 
                            dataout=here("basic_mplus", "EFA_Lab_DEMO.dat"),
                            modelout=here("basic_mplus", "EFA_Lab_DEMO.inp"),
                            check=TRUE, run = TRUE, hashfilename = FALSE)
```
