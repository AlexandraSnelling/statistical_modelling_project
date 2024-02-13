# Final-Project-Statistical-Modelling-with-Python

## Project/Goals
Rather than looking at multiple POI categories to find a relationship using only a single timepoint for available bikes data, 
I decided to look at a single POI category to investigate change in available bikes over time.
My ultimate goals were to answer the following questions:

**1) How did the percent of available bikes change over time on the evening of Friday February 9th, 2024 for Mobi Bikeshare stations (Vancouver, CA) within a 5-minute walk to at least one bar?**

**2) Did the number of bars nearby influence percent of available bikes?**

## Process
### Step 1: Pull data from City Bikes API
Vancouver, Canada was selected as the city for this project.
The bikeshare company information available through City Bikes API is from Mobi Bikes.
To find the relevant 'href' for Mobi Bikes to attache to my query, I used ctrl+F "city name" in the page at the following url: http://api.citybik.es/v2/networks

**City Bikes Query Results:**
Due to the temporal nature of the study, queries were sent hourly from 5pm to 3am Pacific Time, with each result being saved to a new assignment name which included the time of the query. 
The time points obtained ranged from 5pm to 3am Pacific Time.
I parsed through the response to get the details I wanted for the bike stations in that city and put the results into a DataFrame with the following columns:
'city', 'latitude', 'longitude', 'name', 'id', 'empty_slots', 'ebikes', 'normal_bikes', 'slots', 'free_bikes', 'timestamp'
Each data frame was saved as a .pkl file with a corresponding name including the query time.

### Step 2: Pull data from Foursquare APIs
Prepare Foursquare request including the url, api key, headings and parameters (categories, radius, open_at, location).

**categories**
Bar category results can be accessed using the param "query": "bar" or by using the root category ID (13000) in the param "categories":"13000"
However, using the query "bar" and root category ID both returned offtarget results such as cafes(possibly with liquor licences) and more.

Specific bar category results can be accessed via the specific category ID ( For example: 13004 Dining and Drinking > Bar > Apres Ski Bar)
Multiple categories can be added by listing all IDs in a single string, using coma separation.
Therefore, to avoid off target results, a list of specific bar category IDs was used in the params as follows:
*bar_category_ids = '13004,13005,13006,13007,13008,13009,13010,13011,13012,13013,13014,13015,13016,13017,13018,13019,13020,13021,13022,13023,13024,13025,13389'

**radius** 
A radius of '5-minute walking distance' from the bike rack locations was calculated based on evidence from the following study:

*Murtagh EM, Mair JL, Aguiar E, Tudor-Locke C, Murphy MH. 
*Outdoor Walking Speeds of Apparently Healthy Adults: A Systematic Review and Meta-analysis.
*Sports Med. 2021 Jan;51(1):125-141. 
*doi: 10.1007/s40279-020-01351-3. PMID: 33030707; PMCID: PMC7806575.

This study fount that "walking outdoors at a usual pace was associated with an average speed of 1.31 m/s"
5-minute walking distance was therefore calculated as: 5min x 60sec/min x 1.31 meters/sec = 393meters
This metric was used in the params as follows:
*five_min_walk_radius = '393'

**open_at**
Due to the temporal nature of the study, only bars open until at least 10pm were included in the results.
This ensured the data was focused on the appropriate type of establishment. 
The open_at parameter allows for results to be filtered to only those open at the specified time.
Time can be specified as DOWTHHMM (e.g., 1T2130), where DOW is the day number 1-7 (Monday = 1, Sunday = 7) and time is in 24 hour format.
This format was used in the params as follows:
*time_10pm = '5T2200'
Note: Places that do not have opening hour sare not be returned if this parameter is specified.

**location**
In order to obtain data for all locations, the latitude and longitude from each Vancouver bikeshare location was passed in an individual API request.
The latitude and longitude were pulled from each row of a Vancouver bikeshare dataframe as a tuple used as value in a dictionary.
The bikeshare location 'id' corresponding for each row in the dataframe acted as the key entry for each value pair.

**Sending Requests**
A for loop was used to iterate through the latitude, longitude, values and make a request for all 248 locations. 
In this loop, the information from each was indexed through to pull desired location details including "fsq_id", "name", "distance" for each bar location.
The parsed results were appended to a DataFrame along with their corrsponding bikeshare location ID.
After the dataframe was complete, it was saved as a .pkl file

*Note: A Yelp request with corresponding categories was also prepared. It can be viewed in the yelp_foursquare_EDA notebook but we will not elaborate on it here as Foursquare results were *selected over Yelp results for the project because rating and review information was not required and Foursquare results had less information per result, which meant less to store. 


### Step 3: Joining Data and EDA
All .pkl files were loaded from the previously compiled dataframes.
To join the data, the following steps were performed:

**a. Create new dataframe for percent available bikes**
Combine all timepoints with bikeshare id as the key and a column for percent available bikes at each timepoint calculated as free bikes / total slots

**b. Create new dataframe for bar count by bikeshare location id** <br>
Create new database with only bikeshare location id and the bar count (number of locations per bikeshare).

**c. Merge bikeshare percent available bikes temporal data with bar count using bikeshare location ID**

**d. Fill NaN Bar Count Values with zero

**(d.2 Group Data by Bar Count to Pull out Relationships)**
This dataframe was used only to visualize the data in a multiple linegraph where x was time, y was % available bikes and each grouped bar count was plotted as a seperate line.
This visual showed no obvious relationships.

### Step 4: Create Database using SQLite
Project database was created using SQLite and the following tables were created:

**Table (1)(City Bikes Data): 'vancouver_bikeshare_locations'** <br>
This was created from a single city bikes dataframe for bikeshare location information including the following columns:
(bikeshare)"id" as primary key, "bikeshare_name", "city", "latitude", "longitude", "slots"

**Table (2)(City Bikes Data): 'bikeshare_location_temporal_data'** <br>
This was created by looping through a list of all city bikes dataframes to call the required time dependant bikeshare location variables at all timepoints and append.
This including the following columns:
(bikeshare)"id" as foreign key, "empty_slots", "free_bikes", "timestamp", "ebikes", "normal_bikes"


**Table (3)(Foursquare Data): 'bars_within_5min_walk_to_vancouver_bikeshare_locations'** <br>
This was created from the foursquare location result dataframe including the following columns:
"fsq_id", "name", "distance", "bikeshare_location_id" as foreign key

### Step 5: Perform ANOVA, Linear Regression and EDA

The following analysis was performed:

**Two-way Time Series ANOVA** 
using time and bar count

**Linear Regression Analysis** 
using time 

**Linear Regression Analysis** 
using bar count

**(Box Plot)** 
to visualize bar count

## Results
**ANOVA Table Results:**
Timestamp:
The p-value = 0.9965668 (>0.05) 
This suggests that 'timestamp' does not have a significant effect on the number of available bikes.

Bar count:
The p-value = 1.26e-13 (<0.05) 
This suggests that bar count has a significant effect on the number of available bikes.

**Regression Analysis Results for Time:**
R-squared & Adj. R-squared : value is 0.000!
Therefore, no explanatory power.

**Regression Analysis Results for Bar Count:**
Slope: The estimated coefficient = 0.0058. 
Represents the expected change available bikes for a one-unit increase in bar count.

P-value: < 0.05 (0.000), indicating that the effect of 'bar_count' on 'bikes_available' is statistically significant.

R-squared & Adj. R-squared : value is 0.006
This indicates that approximately 0.6% of the variance in available bikes is explained by bar count. 
Therefore, very weak explanatory power.


No true relationships were determined as a result of this data

## Challenges 
**City Bikes API Challenge**
Unfortunately the 11pm timepoint was overwritten and therefore the data was lost.
In future projects with temporal data requirements, it would be useful to define a function which can be applied to perform this activity automatically.
This will help to reduce the burden of manual querying and reduce issues such as the overwrite error that occured at the 11pm timepoint.

## Future Goals
Future project directions include the following:

**Expand the timeframe of the study**
Perform temporal API requests weekly / monthly

**Investigate bar patron behavioral patterns  based on environmental factors**
Pull in data from weather or climate API 