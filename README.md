# Seattle Hospital Location Finder
This web application is my final project for GEOG 495! Broadly speaking, the application generates a list of all the major hospitals in the Seattle area, and then organizes them by using the geospatial technique of sorting by distance based on user input to a geocoder.

## Project Description and Goal
My hopes for this project were fairly straightforward: first, create an application that actually works, and second, is also something that could be useful in the real world.    

My inspiration for this application came from the current state of the global COVID-19 pandemic. Unfortunately, with the onset of winter and the emergence of the new Omicron variant, the world appears to be going in the wrong direction, with infection levels trending ever higher. For most people, hospitals provide the last, best defense against the virus, and so they are clearly one of the most important tools we have to fight against the pandemic. 

As I was looking around for ideas that I could use for my final project, I came across [this website](https://data.rgj.com/covid-19-hospital-capacity/washington/53/king-county/53033/), which shows the number of remaining beds availabile in many Washington State hospitals. Although there was a great deal of very useful information on this site, one thing that I noticed was missing was a way for people to tell which hospitals were closest to them (which, of course, is important information to have when trying to find help in a panic). And so, I decided to try and create one for Seattle area residents.

## Project Development

 My first step was to find some relevant information and data. For this, I used [Seattle GeoData](https://data-seattlecitygis.opendata.arcgis.com/datasets/8e0d80152ccb404abc6ac85351e4aed6_2/explore?location=47.462000%2C-122.130100%2C9.96). I found a list of all the major Seattle area hospitals and downloaded it, then removed and/or reformatted a lot of the extraneous columns to leave only the information I actually needed and make it easier to view.  

Next, I created my own custom basemap style using [MapBox](https://www.mapbox.com/) I kept it fairly simple, and used light colors to keep it easily readable. The one major change that I added to the template style was to mark all hospital locations in red, as a visual cue that they were important to my project/application.    

The final prepatory step was to create a custom icon to display, for which I used the free site [Gravit](https://www.designer.io/en/).

  I first tried to create this application basically from scratch, which did not go very well and caused me a lot of frustration. So, I then refcoused on adapting the code of a [Mapbox Tutorial](https://docs.mapbox.com/help/tutorials/?topic=Map+design) for a different purpose, i.e. my own web application that used my own data and stylistic preferences. Through a lot of trial and error, I eventually arrived at the final product.

 ## The Web Application 

 When the application is initally loaded, this is what you will see: 
 ![startup](https://github.com/whalen323/finalproject/blob/master/finalgeog495-main/images/startup.png)

 The map display is a zoomed-out view of several major Seattle area hospitals, and even more are off-screen. Each hospital is denoted by an icon. You can click and drag the mouse to move around, and the mouse wheel to zoom in and out. 

 On the left side of the screen is a scrolling sidebar which displays the name of each hospital, the city or suburb in which it is located, and each hospital's phone number. If your cursor is within the area of the sidebar, you can scroll or drag it down to see the remainder of the list of hospitals.   

 In the top right corner of the screen is a geocoder, which I will discuss further below.

 You can click on any of the hospital names in the sidebar to have the camera brought to and zoomed in on the location of that particular hospital. A pop-up will also display the hospital's name and street address:
![clicked](https://github.com/whalen323/finalproject/blob/master/finalgeog495-main/images/clicked.png)

Finally, perhaps the most important piece of the application, the geocoder! To use it, simply type in and select any Seattle area landmarks or street addresses and click the search icon. Then, you will see something similar to this: ![geocoder](https://github.com/whalen323/finalproject/blob/master/finalgeog495-main/images/geocoder.png)  

A blue marker will pop up to denote the location that you chose, and the camera will shift to show both that location and where the closest hospital is situated. The same pop-up that you can reach from clicking on a hospital name in the sidebar will display as well. 

But, the most important change to note can be seen in the sidebar, where you can now find the distance in miles from the location you chose to every single hospital listed, ordered with the shortest distance/closest hospital on top. In this case, we can see that the closest hospital to the Space Needle is Virginia Mason Medical Center, which is 1.24 miles away. You can do this with any Seattle area street address or landmark!

## Important Coding Functions
Following is a list of the code used for some of the most important functions of my web application:
```
/**
                 * Create a new MapboxGeocoder instance.
                 */
                const geocoder = new MapboxGeocoder({
                    accessToken: mapboxgl.accessToken,
                    mapboxgl: mapboxgl,
                    marker: true,
                    bbox: [-122.40763, 47.0803367, -121.853675, 47.752643]
                });
                ```
 This function creates the geocoder and sets its boundaries.  

 ```geocoder.on('result', (event) => {
                    /* Get the coordinate of the search result */
                    const searchResult = event.result.geometry;

                    /**
                     * Calculate distances:
                     * For each hospital, use turf.disance to calculate the distance
                     * in miles between the searchResult and the hospital. Assign the
                     * calculated value to a property called `distance`.
                     */
                    const options = {
                        units: 'miles'
                    };
                    for (const hospital of hospitals.features) {
                        hospital.properties.distance = turf.distance(
                            searchResult,
                            hospital.geometry,
                            options
                        );
                    }
                    ```
This function takes the search results coordinates and calculates the distance between the user-selected location and each hospital.             

``` /**
                     * Sort hospitals by distance from closest to the `searchResult`
                     * to furthest.
                     */
                    hospitals.features.sort((a, b) => {
                        if (a.properties.distance > b.properties.distance) {
                            return 1;
                        }
                        if (a.properties.distance < b.properties.distance) {
                            return -1;
                        }
                        return 0; // a must be equal to b
                    });

                    /**
                     * Rebuild the listings:
                     * Remove the existing listings and build the location
                     * list again using the newly sorted hospitals.
                     */
                    const listings = document.getElementById('listings');
                    while (listings.firstChild) {
                        listings.removeChild(listings.firstChild);
                    }
                    buildLocationList(hospitals);
                    ```
This function sorts the hospitals by distance and then rebuilds the list displyed on the sidebar acccordingly.  

```
/**
             * Using the coordinates (lng, lat) for
             * (1) the search result and
             * (2) the closest hospital
             * construct a bbox that will contain both points
             */
            function getBbox(sortedhospitals, storeIdentifier, searchResult) {
                const lats = [
                    sortedhospitals.features[storeIdentifier].geometry.coordinates[1],
                    searchResult.coordinates[1]
                ];
                const lons = [
                    sortedhospitals.features[storeIdentifier].geometry.coordinates[0],
                    searchResult.coordinates[0]
                ];
                const sortedLons = lons.sort((a, b) => {
                    if (a > b) {
                        return 1;
                    }
                    if (a.distance < b.distance) {
                        return -1;
                    }
                    return 0;
                });
                const sortedLats = lats.sort((a, b) => {
                    if (a > b) {
                        return 1;
                    }
                    if (a.distance < b.distance) {
                        return -1;
                    }
                    return 0;
                });
                return [
                    [sortedLons[0], sortedLats[0]],
                    [sortedLons[1], sortedLats[1]]
                ];
            }
            
            ```
This function generates a viewing window that encompasses both the selected coordinates and the closest hospital. 

Of course, there are many more functions and lines of code in the application, which can be viewed in the [index.html](index.html) file in the repository.

## Other Notes
To put it mildly, coding and programming is not my strong suit, and I spent many, many hours trying to get the application to work. For whatever reason, it just doesn't come naturally to me at all. As a result, I am very certain that this application could be explored and improved in myriad ways by someone with a higher degree of proficiency than me. I have also been dealing with a lot of very stressful and difficult personal circumstances over the last few months which has made things even more difficult. In short, this may not be that great of a project, but I did work very hard on it and I'm happy to have accomplished something.

## Data Sources and Acknowledgments
* [COVID-19 Hospital Capacity in King County and Surrounding Area](https://data.rgj.com/covid-19-hospital-capacity/washington/53/king-county/53033/) - Used for background information and gave me the idea for my project.

* [Seattle GeoData](https://data-seattlecitygis.opendata.arcgis.com/datasets/8e0d80152ccb404abc6ac85351e4aed6_2/explore?location=47.462000%2C-122.130100%2C9.96)- Used to obtain data on all Seattle-area hospitals featured in the web application.

* [MapBox](https://www.mapbox.com/)- Used to create the custom basemap layer seen in the application, and the code used in a tutorial provided by this site also served as the basis for that of my own application.  

* [Gravit](https://www.designer.io/en/) - Used to design the custom icon that marks the location of each hospital on-screen.

* [Github](https://github.com/)- Used to host the web application online. 

* [Geography 495](https://github.com/jakobzhao/geog495)- For giving me the skills used to create this project and for generally being an interesting and imformative course! 


