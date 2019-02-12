
# 1-Introduction
My project is to search for potential locations based in a set of conditions, such as distance from one type of place. My example will be to find a location for a secondary house within a region. In this case, the conditions will be that this location should be no more than 55 minutes distance from my house and 30 minutes from the beach. The location of the venues will come from *Foursquare*. Aditional information, such as rating of the venues can also be used to help us determine the best emplacement. The program can also be used to determine the location of a business based in some conditions of distance from specific venues.

This kind of project could interest people looking to determine an ideal location based on specific geographical conditions. This is particularly interesting in the real estate business, but also for all kinds of business, such as a pizza restaurante could look for a place far enough from other pizza places and maybe close from an high-school. To do this, I tried to make my project as general and robust as possible, not fixating the search parameters and conditions nor the sizes of the dataframes.

# 2- Data where you describe the data that will be used to solve the problem and the source of the data.

The data needed for this project was gathered from several sources, namely the data from the venues is collected from *Foursquare*. The initial potential locations data was gathered from the portguese Postal Code data (http://centraldedados.pt/codigos_postais/). To determine distance and times of travel, *Google Maps* API was used, specifically the Distance Matrix to get this information. The actual geographical positions of the potential location in the municipality area was also collected from *Google Maps* API (Geocoding).


|Data                   | Source                                                    |
|-----------------------------------------------------------------------------------|
|Potencial locations    |Portuguese Postal Code information, collected from centraldedados.pt and introduced directly in the code|
|Time/distance from starting point to potential locations|*Google Maps* API (Distance Matrix API)|
|Geographical coordinates of the potential locations|*Google Maps* API (Geocoding API)|
|Location of the beaches|*Foursquare* API                                             |
|Time/distance from potential locations to beaches|*Google Maps* API (Distance Matrix API)|
|Other venues close to the potential locations| *Foursquare* API                      |


# 3- Methodology section 

The objective of this project was to find a set of suitable locations from a range of hundreds of possibilities in the municipality of Grandola (with a surface of 825 km2), based in several conditions. These conditions wouldn't be easily tested without the tools of data analytics.

The conditions of the problem were:
1. The base location is Setúbal, Portugal (50 km south of the capital of Portugal, Lisbon)
2. The locations to test should be in the Southwest coast of Portugal, in the municipality of Grândola, around 85 km south of Setúbal
3. The locations must be at a driving time of less than 55 minutes
4. The locations must be at a driving time of less than 30 minutes from a the selected venue, in our case, the beach

The first task of this project was to find and preprocess the data related to the potential locations. As we needed to try every locality against the desired constraints, we had to collect the locations of the target area, which was the Grândola Municipality, with  the 825 km2. The data was collected from the Portuguese Postal Codes and after filtering, we had the initial potential locations for our project.

The second task was to test every potential location against the imposed constraint of the driving time from the base location. This information was not easily accessible for all the potential locations as we had an initial set of hundreds of points, to test against the driving time from Setúbal. This task was made with the use of *Google Maps* Distance Matrix API, as this information was not accessible with *Foursquare* API. The Distance Matrix API receives as input a set of origin points and destination points and returns a *json* matrix of the distance in kilometers and time of all the possible combinations. As the number of results is  limited to 100, it was necessary to split the potential locations in 3 sets, make 3 separate requests to the API and then join the 3 outputs in *json* format. After this, it was now possible to test the condition of 55 minutes from the base location.

The information of the beach locations was collected from the *Foursquare*. The request used as base location the geographical coordinates of Grândola city, and a radius of 35 km and we searched by the category type of "Beach", which was available in *Foursquare*. After a visual inspection of the results, 7 beaches were found within that radius.

So, with 21 potential locations and 7 beaches, there were a total of 147 tests to make in order to find all the locations that respected the imposed conditions. Again, as the Distance Matrix results were limited to 100, it was necessary to split the potential locations (used as destination points in the matrix) in 2 sets and then join the results. After filtering the locations that were at a driving distance greater then 30 minutes, we had a set of 17 locations which respected the initial conditions. 

In order to refine these results, a *k means* clustering algorithm was modeled in order to have a segmentation of the locations. The selected features for the model were: 
- distance from the base location
- driving time from the base location
- driving time to the closest beach

It was necessary to create a new dataframe with these features and to standardise the data, as most likely the difference of scales between the feaures would give us more weight to the driving distance feature (the distance was in the scale of 80,000 meters, as the driving times were in the magnitudes of tens of minutes). A sample of the unstandardized features dataframe whith the predicted cluster is shown below.

![table2.png](attachment:table2.png)

# 4- Results section where you discuss the results.
The Portuguese Postal Code data was used to find the potential locations. There were initially 323 thousand entries, but with repeated localities. After filtering this data we got 289 unique locations in the Municipality of Grândola.

The test of the third condition, driving time of less than 55 minutes between Setúbal and the potential location, the set of potential locations was reduced to 21 locations, after filtering the results. We can see in the image below that the potential locations are mostly close to the highway beacause close to the coast the driving distance is longest but also beacause as this is a rural area, there are many large properties (agricultural or for turism) and so no potential locality is nearby the coast. Furthermore, these locations would be the most expensive ones, so this was already aknowledged in the initial conditions. Signaled in green, in the north, we see the city of Setúbal, the base location. 

![image1.png](attachment:image1.png)

Using the *Foursquare* API, 9 beaches were returned within a 35 km radius from Grândola. Nevertheless, after a visual analysis of the ploted locations in the map, it was possible to see that there were repeated locations within the *Foursquare* results, enphasising the interest of the visual inspection of the results in data science.

![table1.png](attachment:table1.png)

The time of travel between the potential sites and the beaches was calculed. The 21 locations were reduced to 17. Against our initial thoughts, the closest beach for all the locations is the same: Carvalhal beach. This is the case because of the national road that passes very close to the beach, compared to the more difficult access to other beaches.

![image2b.png](attachment:image2b.png)

After chosing and standardizing the features for the *k means* model, several numbers of clusters were tested, between 3 and 6 clusters, but due to the reduced number of locations (only 17) and after analysing both visually in the map and from the cluster feature analysis (using the mean of the features per cluster, as seen in the image below), the number of clusters selected was 3.

![cluster2.png](attachment:cluster2.png)

The analysis of the mean of the features per cluster seems very clear and gives us the following interpretation of the clusters;
###### -Cluster 0 (red): Locations farthest and longest from home, but faster to the beach
###### -Cluster 1 (blue): Locations closest and fastest from home, but longer to the beach
###### -Cluster 2 (green): Locations farthest and longest from home and longer from the beach

Although the differences are not that important, it seems clear that any choice would be made between cluster 0 and 1, most probably choosing the locations in cluster 0, because of the shortest time to the beach. Below we can see a detailed image of the ploting of the locations per cluster in the map and we can see the closest location to the beach (the farthest to the left in the map), in cluster 0 (red), which is Apaulinha location, 20 minutes from the beach of Carvalhal.
![cluster1.png](attachment:cluster1.png)

# 5- Discussion section 
Althogh my proposed project didn't have an enormous amount of data to treat, this would have been an impossible to reach result without the data science tools that were used. Of course, we can always use additional datasets, such as population density, or additional venues analysis and conditions to deepen the analysis. The results are also conditioned by the availability and the fiability and "freshness" of the data.

These results can support the decision making related to geographical position of real-estate or any other business. In any case, the results from this project can and most probably will be used to analyse with a different perspective the business oportunities that can occur in the case of buying a secondary house in the region of Grândola.

# 6-Conclusion section 
This was a very interesting project to close the data science certificate. As a non developer (althogh I have programed some times over my engineer career), I faced many challenges throughout this project, but with the help of the forum and lots of online search, I managed to finalize this report, making use of several procedures learnt during the course, mainly the use of python and the libraries (such as pandas, matplotlib, etc.), the need for data visualisation and the use of machine learning to analyse the results, in this case with *k means* algorithm.

I think this kind of project idea can be usefull, namelly in the real-estate business but also to suport the decision making of the geographical location of other businesses.
