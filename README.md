# 2025 Election Forecast using Non-representative Survey Data

# Introduction

Election forecasting is an important tool that can be used to predict the outcome of an electoral race ahead of the polling date. Information about which candidate is favoured to win an election is incredibly valuable to many stakeholders, including media outlets, campaign managers, and most cruicially, to voters themseleves. Creating polls that sample a representative segment of the target population is often the greatest logistical barrier to accurate and unbiased election forecasting, and requires considerable time and effort.

In this report, we hypothesize that an representative election forecast can be generated by using a census to post-stratify model results from a non-representative survey sample. We obtain voting preferences from a large-scale 2019 phone survey, which also collected detailed demographic information. We also use 2016 census data representative of the Canadian voter population in order to reweight model results in a representative manner.  

If elections can be accurately forecast by using non-representative datasets, poll predictions can be made more quickly, and with less resources required. This is important because election information is very time sensitive, so the ability to generate nuanced and timely election predictions is of high value, from both a practical and academic perspective.

# Data

## Data Cleaning
```
survey_data <- 
  survey_og_data %>% 
  mutate(vote_liberal = ifelse(q11==1, 1, 0),
         vote_conservative = ifelse(q11==2, 1, 0),
         vote_other = ifelse(q11==3|q11==4|q11==5|q11==6|q11==7, 1, 0),
         gender = case_when(
           q3==1 ~ "Male",
           q3==2 ~ "Female"),
         current_province = case_when(
           q4==1 ~ "Newfoundland and Labrador",
           q4==2 ~ "Prince Edward Island",
           q4==3 ~ "Nova Scotia",
           q4==4 ~ "New Brunswick",
           q4==5 ~ "Quebec",
           q4==6 ~ "Ontario",
           q4==7 ~ "Manitoba",
           q4==8 ~ "Saskatchewan",
           q4==9 ~ "Alberta",
           q4==10 ~ "British Columbia"),
         party_vote = case_when(
           q11==1 ~ "Liberal (Grits)",
           q11==2 ~ "Conservatives (Tory, PCs, Conservative Party of Canada)",
           q11==3 ~ "Other",
           q11==4 ~ "Other",
           q11==5 ~ "Other",
           q11==6 ~ "Other",
           q11==7 ~ "Other"),
         income_family = case_when(
           (q69 < 24999 & q69 > -1) ~ "Less than $25,000",
           (q69 < 49999 & q69 > 25000) ~ "$25,000 to $49,999",
           (q69 < 74999 & q69 > 50000) ~ "$50,000 to $74,999",
           (q69 < 99999 & q69 > 75000) ~ "$75,000 to $99,999",
           (q69 < 124999 & q69 > 100000) ~ "$100,000 to $ 124,999",
           (q69 > 125000) ~ "$125,000 and more"),
         age_category = case_when(
           (age > 17 & age < 30) ~ "18-29",
           (age > 29 & age < 45) ~ "30-44",
           (age > 44 & age < 65) ~ "45-64",
           (age > 64) ~ "65+"),
         education = case_when(
           (q61 == 1 | q61 == 2 | q61 == 3 | q61 == 4 | q61 == 5 | q61 == 6 | q61 == 7 | q61 == 8) ~ "No completed post-secondary education",
           (q61 == 9 | q61 == 10 | q61 == 11) ~ "Completed post-secondary education")
        ) %>% 
  select(gender, current_province, party_vote, income_family, age_category, 
         education, vote_liberal, vote_conservative, vote_other)
survey_data = na.omit(survey_data)

survey_data$income_family <- factor(survey_data$income_family,
                                    levels = c("Less than $25,000",
                                                             "$25,000 to $49,999",
                                                             "$50,000 to $74,999",
                                                             "$75,000 to $99,999",
                                                             "$100,000 to $ 124,999",
                                                             "$125,000 and more"))
survey_data$age_category <- factor(survey_data$age_category,
                                   levels = c("18-29",
                                              "30-44",
                                              "45-64",
                                              "65+"))
survey_data$education <- factor(survey_data$education,
                                levels = c("No completed post-secondary education",
                                           "Completed post-secondary education"))
```

### Survey Data

Analysis was performed on the publicly available [2019 Canadian Election Study (CES) data](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/8RHLG1), which is collected annually and includes questions about past voting behaviour, as well as many demographic factors. The survey was administered over the phone during the campaign and post-election period, and sampled Canadian citizens and permanent residents over 18 years of age. Detailed sampling techniques can be found in the “2019 Canadian Election Study - Online Survey Technical Report and Codebook”. The original, uncleaned data contains 4,021 observations of 273 variables.

In the survey data's original state, each variable is named after its question code (q1, q2, p1, p2, etc). Thus, the cleaning process began with identifying important variables and modifying variable names for convenience and clarity. Each of the selected variables have been renamed to better describe the information being collected.
The table below displays the important variables from the survey data after the names have been changed.

Furthermore, the responses to each question in the survey data are coded as numbers, each representing a specific answer that can be found in the survey data documentation. This is not ideal for this report, so the responses for variables *gender*, *current_province* and *party_vote* are modified
in order to directly display the individual's response as its entire a string representation.

In addition, the income family variable is modified so that it is a categorical value
with 6 categories that correspond with the family income categories in the cleaned
census data. A new variable is also created in order to display and categorize the age 
of the respondent into one of four age groups. As well, the education variable is 
modified in order to only show whether the individual has achieved a Bachelor's 
degree or not. Following that, 3 binary variables are created based on *party_vote* to indicate an individual's likely votes for the Liberal, Conservative and 'Other' parties. Lastly, any observations with a N/A in any variable is removed. 

The newly cleaned data set contains 1923 observations and 9 variables. Below are the final cleaned variables from the survey data.

*age_category*: Age bracket of the individual. Categorical variable.\
*gender*: Gender of the individual. Categorical variable.\
*current_province*: "Current Canadian province the individual resides in. Categorical variable.\
*party_vote*: The political party that the individual claimed they would vote for. Categorical variable.\
*income_family*: The total household income bracket of the individual, before taxes. Categorical variable.\
*education*: Whether or not the individual has completed post-secondary education. Binary variable.\
*vote_liberal*: Whether or not the individual is likely to vote Liberal. Binary variable.\
*vote_conservative*: Whether or not the individual is likely to vote Conservative. Binary variable.\
*vote_other*: Whether or not the individual is likely to vote NDP, Bloc, Green or other parties. Binary variable.

One additional thing to note that is that while Gender is not limited to female and male in the survey, there is only 1 observation out of 1923 observations for "Other" in the *gender* variable. Thus, that observation was excluded as there is an insufficient sample size to draw any meaningful conclusions for those in the "Other" category.

### Census Data
[The data used as the census data comes from the 2017 General Social Survey (GSS)](https://sda-artsci-utoronto-ca.myaccess.library.utoronto.ca/legacy_sda/dli2/gss/gss31/gss31/more_doc/index.htm) that was conducted
between February to November 2017. The goal of the survey was to gather and view the changes
in the quality of life of Canadians as well as provide a general census on current
issues.
It targeted non-institutionalized people above the age of 15 that resided in a Canadian
province. The data collection process was done through telephone calls, in which telephone 
numbers were retrieved from the Address Register provided by Statistics Canada. 
The data itself contains 20,602 observations of 81 variables. 

```

```
