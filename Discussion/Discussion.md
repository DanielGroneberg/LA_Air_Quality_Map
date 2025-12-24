#### 12.18.25

The data esp. the ArcGIS/ directory is a mess. I need to figure out which files are actually being used, because there are many named similar things e.g. la_outline . . . 

The subdirectory structure of Data/ is also a mess. I initially used Inputs/ and Outputs/ but this quickly gets confused. Reorganizing it into just rasters and vectors might make more sense. In that case, I could store all finished rasters/vectors in Data/ whereas ArcGIS/ could store scratch layers

#### 12.3.25

#### I initially tried using the LA County roads dataset to create a distance raster

![Streets_dist.png](/home/firebird/Pictures/Screenshots/Streets_dist.png)

 ![AQ.png](/home/firebird/Pictures/Screenshots/AQ.png)

This is generally fine, but as shown, there are stations outside of LA county. The distance raster there is not accurate as the roads dataset does not extend there.

Most ML models would probably require a quadrilateral shape without null areas - I could clip both the stations and distance rasters to LA county, but then the resulting shape would have missing areas.

#### Roads classification justficiation

[Map Viewer](https://www.arcgis.com/apps/mapviewer/index.html?layers=a1543cfa466b45aab01d5ee75152ccb0)

E California Blvd and N Rosemead Blvd are symbolized as "Major Highway" and Huntington Dr. as a Parkway

![](/home/firebird/.var/app/com.github.marktext.marktext/config/marktext/images/2025-12-02-10-12-25-image.png)

![](/home/firebird/.var/app/com.github.marktext.marktext/config/marktext/images/2025-12-02-10-14-54-image.png)

All three are multi-lane highways and clearly fairly major routes - they can be classified together.

### 12.2.25

[North American Roads](https://geodata.bts.gov/datasets/usdot::north-american-roads/api)

[Master Plan of Highways](https://egis-lacounty.hub.arcgis.com/maps/lacounty::master-plan-of-highways/about)

My OpenAQ points are dense in the city parts of LA county but sparse (1 or 2 locations) in the outlying areas. My idea is to maintain my OpenAQ points as the target variable. I can use NOAA points (more distributed) to provide precip and windspeed/direction data. I can also include a new layer for distance from roads. 

I can use spatial CV to break my dataset into blocks and test performance against each block. Since the data is so sparse, spatial CV would be the best approach on an LA County-scale. If I want to perform more analysis on the dense cluster within the city of LA, buffered CV would be better suited for that.

##### Roads

The LA County roads dataset only includes city/county streets. "Major highways" are roughly "stroads" e.g. Alamitos Ave is a major highway. This means Highway 1 (state) or freeways are not included in the dataset. 

I think three tiers would be the most efficent:

1. Freeways: Offset, deliminated by speeds of 60+ mph
   
   * Highway 91, I-5, 110, 105, etc. are all offset, have massive ramps, many lanes, high traffic volume and high speeds

2. "Stroads": surface-level, but still used as major routes - delim by speeds of 35+ and 2+ lanes, or "major highway" on the LA County dataset
   
         * 33.87285643780068, -118.32648870938628 The intersection of Artesia and Crenshaw. Artesa is Highway 91 - yellow on Google Maps. Crenshaw is not formally a highway, but these streets should be in the same category. G Maps would be willing to route you on these types of streets over many miles

3. Residental or slower streets

### Article

Nov. 17, 2025
https://www.sciencedirect.com/science/article/pii/S1364815224003736

So far, my plan would be to monitor PM2.5 pollution, meaning filling in gaps in real time data (not forecasting pollution into the future). I can use the above paper to review existing approaches to this problem - some of them use the same data types for features and targets

#### Datasets

1. OpenAQ

```
sensor_display_name
PM2.5              415
PM1                338
PM10               304
Temperature (C)    299
Temperature (F)    219
RH                 158
PM0.3 count         78
NO₂                 69
O₃                  38
NO                  18
NOx                 18
CO                  17
SO₂                  4
```

#### Relevant papers:

```
(Ku et al., 2022)

PM2.5, PM10, O3, NO2, CO,
and SO2
Climatic factors and air-pollution indicators,
including PM2.5, PM10, O3, NO2, CO, and
SO2,
XGBoost and GPR-based
ML models
R2 values of over 0.67 and RMSE values
below 13.9 per 10,000 inhabitants.
```

```
Hahnel et al.,
2020)

NO2 and PM10 Pollution measurements, traffic data,
weather data
Geometric DL MAE of the DL computed values was 1.7μg/cm
```

According to my current approach (pollutant and climactic data to predict PM2.5), this is effectively spot on:

```
Wang et al.,
2022a

PM2.5 PM2.5, PM10, SO2, CO, NO2 and O3
and meteorological data including
wind speed, air temperature, and dew
point
GWO-LSTM Improves the MAE, RMSE and MAPE by
32.92%, 27.65% and 30.02%, respectively
```

Depending on how things go, it may be possible to combine traffic/road layers with this approach. Since Wang et al is the closest to my existing plan, I will investigate this paper fist.

That said, they appear to be forecasting, based on the model being used: `GWO-LSTM`

### 

Nov. 12, 2025:
My initial OpenAQ section is functionally complete, but is generally a mess. I need to 

1. Refactor the initial data intake notebook
2. Create another notebook for deeper EDA

I think my plan should be to create another repo which I can start fresh from.

For the NOAA data, I still don't have any functioning data - I have station metadata using the V2 API. V1 can pull in functional data (and seems to even have a wider range of data) but does not allow for effective querrying. V2 has better querrying even if the API is less robust.

#### General concept

As of now, my concept is to perform monitoring, i.e. to be able to impute missing data

----------

1. Pull in metadata
2. Create list of recently active stations from metadata
3. Use list of stations to query Athena
4. Create database of actual sensor data

-----

## General concept

* Kriging