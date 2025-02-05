# Flu Modeling Project

## Motivation
Efforts to reliably forecast influenza seasonal illnesses have long been a challenge due to a variety of factors: virus mutation, vaccine efficacy, population willingness to become vaccinated, etc. An accurate influenza forecast can help hospital preparation for incoming patients as well as prepare drug manufacturers and distributors to have adequate supply.

In an effort to prepare for influenza related pharmaceutical purchasing, I have created this repository for estimating flu severity in the U.S. utilizing southern hemisphere flu rates as predictors. In order to ensure patients have their medications during the flu season, the suppliers that manufacture the pharmaceuticals need as much notice as possible. Estimating the flu season as early as the summer before can give suppliers the adequate time needed to prepare the necessary supply. Typical state of the art techniques have a forecast range of ~1-4 weeks, so this approach sacrifices some of the week-to-week accuracy in order to increase the forecasting power to ~5-6 months. 

## Data
In the pharmaceutical sourcing space, drug distributors forecast seasonal illness demand months before the season takes place. Due to the length of the required forecast, Australia's flu season is often used as a crude estimation of the incoming U.S. season due to the nature of the southern hemisphere experiencing winter when 6-months offset from the north. It is my belief that there exists some combination of southern hemisphere countries' flu seasons that can provide an improvement to that forecast for the U.S. Data used in this project comes from the WHO as well as the CDC's FluSurv-NET data set and can be found below. 

WHO data available at https://www.who.int/teams/global-influenza-programme/surveillance-and-monitoring/influenza-surveillance-outputs

CDC data available at https://gis.cdc.gov/GRASP/Fluview/FluHospRates.html


The target data used to estimate influenza severity is CDC's FluSurv-NET dataset, which details patient hospitalization information in participating hospitals representing ~9% of the U.S. population. This was chosen because patients that have undergone viral testing in a hospital setting likely have reliable results. The WHO provides publicly available influenza dataset for global rates, where I first determine which countrys' flu seasons are highly correlated with the U.S.

<p>
    <img src="./Pictures/SH_Corrs.PNG" width="1000" height="500" />
    <caption>Figure 1: Countries with highest correlation with the U.S. flu season in the "sweet spot" (~5-6 months before the U.S. season). </caption>
</p>

## Results

The model architecture is explained in the Paper section, and can be summarized into two separable pieces; a RNN and a sequential piece. The RNN piece features a varying combination of different flavors of RNNs and linear layers, and the sequential piece possesses a series of linear and drop-out layers with ReLU activation functions used between layers. This design was chosen to capture the time-dependance in the data without imposing any restraints as well as allow for creativity to build an ensemble of acceptable solutions. Each model is trained on data between 2013-2020 (COVID creates a break in the seasonality) and the 22-23 season was used as a validation set to stop the training (a self designed loss function was used to prioritize peak location and height with details found in Paper). Models that had a sufficient loss curve for the validation metric were accepted as candidates. In an effort to combine the ensemble into one prediction, Lorenzian curves were used as an approximate parameterization for the flu season with the peak heights, locations, and widths defined from accepted models. 


<p>
    <img src="./Pictures/Forecasts.png" width="1000" height="500" />
    <caption>Figure 2: The seasons in blue were both done using the previous season as the validation set. Due to Covid and previous data availability, the 22-23 season (red) was the first season used as validation to stop the training and therefore carries a bias. The shaded regions reflect the variance from the different model variabilities.</caption>
</p>

This project is designed to be simple such that it can be easily explained to a non-technical audience (model architecture not included), so naturally there are many extensions that could improve upon this process. Including U.S. holiday features, including previous years vaccine efficacy and adherence, and dominant virus type could all be natural data extensions worth fine tuning these models. Model results could be improved as well by exploring multi-peak structures of the flu season rather than the single peak structure parameterized here.


