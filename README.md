# Disparities in Life Expectancy Dashboard for DSNY9 Application

Here is a breakdown of the notebooks in the repository:

## Data Cleaning Census Notebook

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/datascisteven/DSNY9-Disparities-in-Life-Expectancy-Dashboard/blob/main/Data%20Cleaning%20Census.ipynb)

The following tables were obtained via API from the American Community Survey (ACS):

* **DP02** (Selected Social Characteristics in the United States)
* **DP03** (Selected Economic Characteristics)
* **DP04** (Selected Housing Characteristics)
* **DP05** (ACS Demographic and Housing Estimates)

These datasets cover the years **2010 to 2022**.

Since column headers vary across years, they were standardized to ensure continuity in the data. For example, the original Census headings ``MARITAL STATUS Males 15 years and over Never married`` and `MARITAL STATUS Never married` were both renamed to `males_never_married` for consistency.

Additional data cleaning steps included:

* Standardizing geographic column names
* Replacing **-888888888.00** with np.nan
* Dropping columns with more than **20% missing data**

This process resulted in over **560 features**.

Furthermore, approximately **45 features** from the Community Health Rankings (CHR) were found to be derivable using ACS Census data, following the CHR methodology. These calculated values were used to fill in missing CHR data. This required downloading additional Census Bureau CSV files relevant to the CHR feature calculations.

## Data Cleaning CHR (Community Health Rankings) Notebook

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/datascisteven/DSNY9-Disparities-in-Life-Expectancy-Dashboard/blob/main/Data%20Cleaning%20CHR.ipynb)

The files were downloaded as CSV files from the Community Health Rankings website, and it was a straightforward process of uploading the files, concatenating the dataframes into one dataframe, and then creating a dictionary to change their heading codes into human readable headers in a similar format to the Census headings.

This resulted in just **over 200 different features** with potential data ranging from **2010 to 204**.

## Data Merging and Imputation Notebook

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/datascisteven/DSNY9-Disparities-in-Life-Expectancy-Dashboard/blob/main/Data%20Merging%20and%20Imputation.ipynb)

In previous notebooks, we extracted as many features as possible from the **Census** and **Community Health Rankings (CHR)** datasets. While completeness varied across columns, we identified patterns in the missing data, which informed our imputation strategy.

### **Feature Selection and Data Cleaning**

- Before imputing missing values, we refined the dataset by removing columns with **more than 50% missing data** to create a more manageable feature set. Additionally, we addressed **FIPS code** (county boundary) changes between **2010 and 2024**. Key modifications included:

  - **Connecticut (2022**) – Some CHR data was converted from **Planning Regions** back to former counties. No changes were needed for Census data since it does not extend beyond 2022.
  - **Alaska (2013, 2015, 2019)** – Counties underwent **renaming, merging, and division**.
  - **South Dakota & Virginia** – Minor county boundary adjustments.
- All counties in the dataset were aligned with the **2019 county boundaries** for consistency.

### **Imputation Process**

- After standardizing FIPS codes, we applied **Python-based imputation techniques**, including:

  - **K-Nearest Neighbors (KNN)**
  - **Seasonal AutoRegressive Integrated Moving Average with Exogenous Variables (SARIMAX)**
  - **Random Forest Regressor**
- The resulting dataset contained **116 features with no missing values**.

## Feature Engineering Notebook

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/datascisteven/DSNY9-Disparities-in-Life-Expectancy-Dashboard/blob/main/Feature%20Engineering.ipynb)

Among the remaining features, it was evident that many were still redundant. To refine the dataset, we initially applied domain intuition to eliminate overlapping variables:

* **Children in poverty-related features:**
  * ``children_eligible_for_free_lunch`` and ``children_in_poverty`` are nearly synonymous, as children in poverty are the primary recipients of free lunch.
  * Both have strong overlap with ``children_in_single_parent_households``, though they are not identical.
* **Income inequality metrics:**
  * ``gini_index`` and ``income_inequality`` measure the same concept and are redundant.
  * ``poverty`` was removed as it overlaps with ``children_in_poverty``.
* **Education levels:**
  * ``high_school_completion`` and ``high_school_graduation`` were merged due to conceptual similarity.
  * ``some_college`` and ``college_completion`` were also consolidated.
* **Industry vs. occupation categories:**
  * Industry categories were retained, while occupation categories were removed to avoid redundancy.
* **Housing-related metrics:**
  * ``severe_housing_problems`` is a composite measure that includes:
    * ``pct_households_with_high_housing_costs``
    * ``pct_households_with_lack_of_kitchen_or_plumbing_facilities``
    * ``pct_households_with_overcrowding``
  * Only one was retained to avoid duplication.

To further reduce multicollinearity, **Variance Inflation Factor (VIF) scores** were used. High VIF values inflate standard errors in regression coefficients, making them unreliable. Our goal was to keep **VIF scores below 5**, either by eliminating redundant features or feature engineering related ones.

Additionally, we analyzed the **top positive and negative correlations** during feature selection to retain the most informative variables.

After this process, we finalized a dataset with **52 features,** ensuring strong correlations while maintaining VIF scores below 5

## Preparing for Tableau Notebook

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/datascisteven/DSNY9-Disparities-in-Life-Expectancy-Dashboard/blob/main/Preparing%20for%20Tableau.ipynb)

The final steps in the data preparation involved:

* **Adding Geographic Variables**
  * Allowing for analysis by region and division
* **Adding Urbanization Level by Year**
  * Metropolitan (large, medium, small) vs. Non-metropolitan (micropolitan, non-core) counties
* **Adding County Life Expectancy Data by Race/Ethnicity**
  * Combining 2010-2019 from IHME and 2020-2024 from CHR
  * Resolving FIPS Code changes from 2010-2024
  * Used for calculating change in life expectancy for race and ethnicity from 2010 to 2024
* **Creating Percentile Ranks for Features**
  * Used for calculating life expectancy gaps between top and bottom percentiles

## Final Dataset

Here is the final set of features that was uploaded to Tableau for further analysis:

### **Health Outcomes**

- **Mortality/Length of Life:** Life Expectancy, Premature Age-Adjusted Mortality, Premature Death
- **Morbidity/Quality of Life:** Frequent Physical Distress, Frequent Mental Distress, Diabetes Prevalence, HIV Prevalence, Poor or Fair Health, Poor Physical Health Days, Poor Mental Health Days, Low Birthweight

### **Health Factors**

- #### **Health Behaviors**

  - **Tobacco Use:** Adult Smoking
  - **Diet and Exercise:** Adult Obesity, Food Environment Index, Food Insecurity, Limited Access to Healthy Foods, Physical Inactivity, Access to Exercise Opportunities
  - **Alciohol and Drug Use:** Motor Vehicle Crash Deaths, Drug Overdose Deaths, Excessive Drinking, Alcohol-Impaired Driving Deaths
  - **Sexual Activity:** Sexually Transmitted Infections, Teen Births
  - **Other Health Behaviors:** Insufficient Sleep
- #### **Clinical Care**

  - **Access to Care:** Uninsured, Uninsured Adults, Uninsured Children, Primary Care Physicians, Dentists, Mental Health Providers, Other Primary Care Providers
  - **Quality of Care:** Preventable Hospital Stays, Mammography Screening, Flu Vaccinations
- #### **Social and Economic Factors**

  - **Employment:** Unemployment
  - **Education:** High School Completion, High School Graduation, Some College, Disconnected Youth, Reading Scores, Math Scores, School Segregation, School Funding Adequacy
  - **Income:** Children in Poverty, Gender Pay Gap, Median Household Income, Income Inequality, Living Wage, Children Eligible for Free or Reduced Price Lunch
  - **Family and Social Support:** Inadequate Social Support, Children in Single-Parent Households, Residential Segregation - Black/White, Child Care Cost Burden, Child Care Centers, Social Assocations
  - **Community Safety:** Homicides, Suicides, Firearm Fatalities, Motor Vehicle Crash Deaths, Juvenile Arrests, Injury Deaths, Violent Crime
- #### **Physical Environment**

  - **Air and Water Quality**: Air Pollution - Particulate Matter, Drinking Water Violations
  - **Housing and Transit:** Homeownership, Severe Housing Problems, Driving Alone to Work, Long Commute - Driving Alone, Housing Units

### **Demographics**

- **Population:**  Population
- **Age and Gender:**  % Below 18 Years of Age, % 65 and Older, % Female
- **Race/Ethnicity:**  % Non-Hispanic Black, % Non-Hispanic White, % American Indian or Alaska Native, % Asian, % Native Hawaiian or Other Pacific Islander, % Hispanic,
- **Language Proficiency:** % Not Proficient in English
- **Urban/Rural:** Population, Housing Unit Density, % Rural

## Citations:

Institute for Health Metrics and Evaluation. (2022). *United States mortality rates and life expectancy by county, race, and ethnicity, 2000–2019* [Data set].  Institute for Health Metrics and Evaluation (IHME) . https://www.healthdata.org

University of Wisconsin Population Health Institute. County Health Rankings & Roadmaps 2024. www.countyhealthrankings.org.

U.S. Census Bureau. (2022). *Selected Social, Economic, Housing, and Demographic Characteristics: American Community Survey (DP02, DP03, DP04, DP05), 2010-2022* [Data set]. https://data.census.gov
