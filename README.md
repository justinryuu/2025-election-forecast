# 2025 Election Forecast using Non-representative Survey Data

## Introduction

Election forecasting is an important tool that can be used to predict the outcome of an electoral race ahead of the polling date. Information about which candidate is favoured to win an election is incredibly valuable to many stakeholders, including media outlets, campaign managers, and most cruicially, to voters themseleves. Creating polls that sample a representative segment of the target population is often the greatest logistical barrier to accurate and unbiased election forecasting, and requires considerable time and effort.

In this report, we hypothesize that an representative election forecast can be generated by using a census to post-stratify model results from a non-representative survey sample. We obtain voting preferences from a large-scale 2019 phone survey, which also collected detailed demographic information. We also use 2016 census data representative of the Canadian voter population in order to reweight model results in a representative manner.  

If elections can be accurately forecast by using non-representative datasets, poll predictions can be made more quickly, and with less resources required. This is important because election information is very time sensitive, so the ability to generate nuanced and timely election predictions is of high value, from both a practical and academic perspective.

## Data

$$Y^{Lib}_i=log\left(\frac{p_i}{1-p_i}\right) = \beta_0 + \phi_g + \epsilon_h$$
