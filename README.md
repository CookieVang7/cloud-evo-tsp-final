# Cloud Computing 4610 Final - The Traveling Salesmen Problem (TSP) and Evolutionary Computation
## Chineng "Cookie" Vang
##### Thanks to Nic McPhee, Peter Dolan, Jack Perala, and Rob Beane


# Table of Contents
- [Overview-Purpose](#overview-purpose)
- [Solution-Structure](#solution-structure)
- [User Documentation](#user-documentation)
- [Table Structure](#table-structure)
- [Lambda Functions](#lambda-functions)
    - [`GetCityData()`](#getcitydata)  
    - [`GenerateRandomRoute()`](#generaterandomroute)
    - [`GetRouteById()`](#getroutebyid)
    - [`GetBestRoutes()`](#getbestroutes)
    - [`MutateRoute()`](#mutateRoute)
- [API Details](#api-Details)
    - [`/best`](#best)
    - [`/city-data`](#city)
    - [`/mutateroute`](#mutate)
    - [`/routes`](#routes)
    - [`/routes/{routeId}`](#getRoute)
- [IAM Roles](#iam-roles)
- [Leaflet Details](#leaflet-details)

### Overview-Purpose
This project tackles the Traveling Salesman Problem (TSP) by evolving TSP routes. The Traveling Salesman Problem inquires that if you are given a list of cities and their distances from one another, what is the shortest possible route that visits each city including looping back to the beginning city? An answer to this can be accomplished by taking an initial population of routes between all the cities and evolving the best of them (called parent routes) to create shorter routes (called child routes). The process is then repeated with the new set of child routes. Each iteration of the process is called a generation where the number of generations is specified by the user.

In the application, the user specifies the size of the initial population, the number of parent routes to evolve from that population, the number of generations they want to run and then they give a name to their simulation (a single run of the application) as a Run ID. Once they click the "Run evolution" button, a popup alert lets the user know that the application will run with the given parameters. As the simulation is running, the page will continually update and display four key things. It will show the details of the best overall route, a map that draws out the best overall route, the routes created in a single generation and a list of the best routes to come out of each generation. Once the simulation is complete, there will be a popup alert letting the user know it is done. The route that remains in the best overall route section is the best route that was generated during the simulation.

### Solution-Structure:

My application is hosted on GitHub pages which grabs HTML and JavaScript files along with other dependencies from an associated repository. Once that was set up (with the starter code given to us), I created 2 tables in DynamoDB through AWS. The first table is called cityDistance_data and holds information about the cities and their distances from one another. The second table is called evo-tsp-routes and initially does not hold anything, but will eventually be populated with the created routes. Next I created 5 lambda functions with AWS Lambda. They are GetCityData, GenerateRandomRoute, GetRouteById, GetBestRoutes and MutateRoute. GetCityData gets information about the cities (not the distances) from the first table. GenerateRandomRoute gets information about the distances between cities from the first table and creates a route that connects all the cities (although not necessarily optimal). Then it puts the route into the evo-tsp-routes table. GetRouteById gets the data about a specified route from the evo-tsp-routes table. GetBestRoutes gets a specified number of routes from the evo-tsp-routes table which have the same Run ID and generation. MutateRoute creates child routes based on a parent route (the parent and child route have the same Run ID but the child route is part of a different generation) and then puts the newly created child route(s) into the evo-tsp-routes table.

I created a role on AWS IAM (identity and access management) to allow the lambda functions to interact with DynamoDB through get, put, query, and batchWrite requests. Then I used AWS API Gateway to create API's for the lambda functions. The API's will allow the application to access and utilize the lambda functions. Functionality of the application is controlled through a JavaScript file called evotsp.js. This file uses the lambda functions to generate child routes and it accesses them through asynchronous AJAX calls. When the user enters global parameters and clicks the "Run evolution" button, the file will generate child routes. The page displays the best routes from each generation, the routes generated in each generation and details about the best route. The best route is displayed on a map (and also updates as the simulation goes) where Leaflet (a JavaScript library) allows each city to be highlighted by a red circle and the path is dashed in blue. The map tiles are provided by Mapbox.

### User Documentation:

When the user opens the application, they will immediately see the map of the cities from the first table with each city highlighted by a red circle. The "Run evolution" button is what runs the application, but it first needs 4 fields to be entered by the user. ![](https://lh3.googleusercontent.com/DACYm7-Qq50-vmgR4h26WFpYN2cZXj3rUyDzK_P2pgbLFOp6tgenkRDACdC1fjM1gB8hBvk_PTqpqz6mGsrpJ9lMRyu4J1ASAzQnHOYWQpiOKcUH6Ddo5ZCkpqhR8ilEd4nXxUzS)The first field is Run ID. It is essentially the name of the user's simulation. If the user does not enter a Run ID, a randomly generated one composed of 16 numbers and letters will be made for them. The second field is the population size and the third field is the number of parents to keep. In each generation processed in the simulation, there will be an initial number of routes (the population size field). A subset of the initial population will be used as "parent" routes (the number of parents to keep) which will generate child routes to make up the next generation. As an example, an initial population of 100 and a parent number of 20 means that 20 of the initial 100 routes will be parents. Thus, each parent will have 5 child routes to get back to the initial population number. The new 100 child routes will act as the next generation. The number of child routes each parent has to make will not always be equal (but relatively close) to other parents if the numbers do not divide perfectly.  The last field is the number of generations that the simulation will run. Leaving blank any of the previous three fields will reload the page and have the user enter the information in again.

When the user fills in each field and clicks the "Run evolution" button, there will be a popup alert notifying the user that the simulation has begun: ![](https://lh4.googleusercontent.com/LAejs0Liq2q9xCzUZ8mb2yD4JWDBT66N163NSKe73kp48Legxs6Upf3Kvg1oFcg5nx4sNaQRiRtS7yJt7xxmdAkwd_whD1jM1FIiyhOSVdHs4lQz2nfcO5E9zYKaC3nmV4RvT0tg)

As the application is running, it will update information on the page about the best overall route, display a map of the best route, show the routes generated in a current generation, and the best routes from each generation: ![](https://lh5.googleusercontent.com/68oNnplj8qgcxvsJ966bsY8NXkWSjHWJ6efOuvXd2daM9NGPNGvwNZYVaUClm785uNegInVuJx25OgDa9Bg_s3wGGurk3IKPezN0befkf5Una-YGCwUW0FGOTn9o2NE0bKmOKVX_)![](https://lh4.googleusercontent.com/bkAZ1E9keTaFTnUYyoI865OG0YEO9OU2qVMfRWudy39vNObPZhUGadMaDAYjU7Wf3x68PROAMkt_hh_qBlyuE-3B-NodKlC_y2gO53xtmKM94gOmuEKth3xwOjnQYIqrNv4znjgJ)

Similar to the popup alert to signal the start of the simulation, there will be a popup alert to let the user know the application is done running the simulation. This means that the route remaining on the map and the details in the best route details section is the best route that was generated in the simulation. In the best routes from previous generation(s) section, the generation a route was generated in is included in the information returned along with the route, length, and Route ID. Length of routes are measured in meters as the crow flies, a link to more to that is added next to the best length of the best route.

### Table Structure:

The first table I made is called cityDistance_data. The primary key (and a field) of this table is "Region". There is only one object in the table with a partition key of "Minnesota". If we were to expand on this project, we could include other regions such as Kansas or Japan. The other two fields in this table are "cities" and "distances". The "cities" field holds information about cities in the region while "distances" holds information about the distance from each city to every other city. The data about cities is used by the GetCityData function to grab the location of each city to display on the map. The distance data is used by the GenerateRandomRoute and MutateRoute lambda functions in order to compute the length of routes.

The second table I made is called evo-tsp-routes. It starts out empty, but will be filled with routes when the application is run. Fields of this table are routeId, len (length), route and runGen. The runGen is the Run Id concatenated with # and then concatenated with the generation of the route. An example is helloWorld#2. So the Run ID is helloWorld and the generation the route came from is generation 2 of the simulation. The field "route" is an array that has a possible combination of the cities. So if there are a k number of cities in the cityDistances_data table, then "route" looks like a permutation of [0,1,...,k-1] where each number is an index that corresponds to a city in the cityDistances_data table. The partition key of evo-tsp-routes is the Route ID. Each Route ID is unique so making it the partition key makes grabbing a specific route easier. I added an index for this table called runGen-len-index. The primary key of the index is the runGen. Making runGen the partition key for the index allows us to specify the routes we want to grab from the table easier. All the routes in a simulation have the same Run ID, so if multiple simulations are run (with different Run IDs) we can quickly tell the table we want to see all the routes from a specific simulation. The secondary key of the index is the length of the route. Having length as the secondary key helps the GetBestRoutes lambda function filter out routes that have lengths above a certain threshold since our objective is to find routes with the shortest distance(s) possible.

### Lambda Functions:

There are 5 lambda functions for this project. They are GetCityData, GenerateRandomRoute, GetRouteById, GetBestRoutes and MutateRoute. 

####   `GetCityData()`: 

This function gets the information about the cities from the cityDistance_data table. Both sets of information about cities and distances are returned by this get request. Then in the exports.handler for the function, the cities portion of the object will be selected and then returned. If there were more regions in the table, there would likely have to be additional indexes and sort keys so the get request would then have been a query request.

####   `GenerateRandomRoute()`: 

This function has a helper method called generateRandomRoute. It is passed a Run ID, generation number, callback method and runGen. It gets the information about the distances between cities from the cityDistance_data table. Each city in the cityDistance_data table is associated with an index number. Those index numbers are put into an array and shuffled which creates a new route. The length of that route is then computed. Lastly, it uses a put request to put the new route (other info for the route was passed as parameters) in the evo-tsp-routes table.

####   `GetRouteById()`: 

In this function, a Route ID is passed to a helper function called getRouteById. It uses a get request on the evo-tsp-routes database with the Route ID specified. Since the evo-tsp-routes table has the Route ID as the partition key and Route IDs are unique, no other search parameters have to be used. Then all the information about that route (Route ID, runGen, length and the route of cities) is returned.

####   `GetBestRoutes()`: 

This function uses the helper method getBestRoutes. It's passed a Run ID, generation and number of routes to return. It then queries the evo-tsp-routes table to return routes that have the same Run ID and generation. How many routes is specified by the number of routes to return. The routes that are returned have lengths below a certain length threshold (since we're trying to find the shortest routes) and are ordered in terms of their length. So if there are 10 routes returned, of those 10 routes, the first route is the shortest and tenth route is the longest. 

####   `MutateRoute()`: 

This function gets distance data from the cityDistance_data table and gets a single route (acting as a parent route) from the evo-tsp-routes table. Then it generates a specified number of child routes and puts them into an array. The array is filtered so that only the child routes who have lengths below a certain threshold are kept. Then this new subset of child routes is turned into a JSON object and BatchWrite is used to put all the new child routes in the evo-tsp-routes table. Each child route has the same Run ID as the parent route, but the child route has a generation of the parent route's plus one. So if helloWorld#7 is the parent route, then a child route will have helloWorld#8.

### API Details: 

For each lambda, I had to make a corresponding API resource and include methods to allow the lambdas to make their various requests.

####   `/best`: 

For the GetBestRoutes lambda function, the resource I made had a path of /best. The method associated with it was GET. The GetBestRoutes lambda function takes in the parameters Run ID, generation and the number of routes to return (called numToReturn). So in the evotsp.js file, adding /best?runId=${runId}&generation=${generation}&numToReturn=${numToReturn} to the end of the invoke url will allow the application to make its query request to evo-tsp-routes. The fields in the url are needed since the function is using querying the table and has multiple parameters to search the table by.

####   `/city-data`: 

This resource path is associated with the GetCityData lambda function. It has a GET method since it is getting data from the cityDistance_data table. To access the method from evotsp.js, the url is /city-data concatenated to the end of the invoke url. There are no parameters needed unlike GetBestRoutes.

####   `/mutateroute`: 

This resource path is associated with the MutateRoute lambda function. It has a POST method since it is putting multiple routes into the evo-tsp-routes table. To access the method from evotsp.js, the url is /mutateroute concatenated to the end of the invoke url. There are no parameters needed since we are posting items into the table.

####   `/routes`: 

This resource path is associated with the GenerateRandomRoute lambda function. It has a POST method since it is putting a route into the evo-tsp-routes table. To access the method from evotsp.js, the url is /routes concatenated to the end of the invoke url. Similar to /mutateroute, no parameters are needed since we are posting an item into the table.

####   `/routes/{routeId}`: 

This resource path is associated with the GetRouteById lambda function. It has a GET method since it is getting a route from the cityDistance_data table. To access the method from evotsp.js, the url is /routes/routeId concatenated to the end of the invoke url. The routeId is added at the end here since that is the parameter we are searching the table by (similarly to GetBestRoutes)

### IAM Roles:

An IAM role will define other AWS services that can interact with the lambda functions. If this was a bigger scale project, I definitely would have created multiple specific roles, but for this project, I created a single IAM role on IAM AWS. There are 4 permissions needed from DynamoDB. Within the role, I created a new inline policy and after choosing the DynamoDB service, I added GetItem, Query, PutItem and BatchWriteItem as actions. A get request grabs a single item, so GetRouteById will now be able to grab a route's information from the evo-tsp-routes table. GetCityData will also use a get request to grab the object in the cityDistance_data table (as it is the only object in the table). A query request grabs a collection of items, so GetBestRoutes will now be able to request a certain number of routes with the same Run Id from the evo-tsp-routes table. GenerateRandomRoute can now use a put request to put a single route in the evo-tsp-routes table and MutateRoute can use BatchWrite to put up to 25 routes into the evo-tsp-routes table at a time.

### Leaflet Details: 

I used Mapbox as the tile provider for the map on the webpage. To do this, I created a token on the [Mapbox website](https://www.mapbox.com/). Putting this access token in the html file for the application allowed it to use their map data. Leaflet is a JavaScript library that lets us interact with the map we got from Mapbox. The CSS and JavaScript files are from the [Leaflet Quickstart Guide](https://leafletjs.com/examples/quick-start/). Using Leaflet, we can highlight the cities on the map using red circles. Clicking on a red circle would tell the name of the city. We can also use Leaflet to draw out the best map with a dashed blue line. The dashed path will update to always match the best generated route: ![](https://lh6.googleusercontent.com/VCnHJ80_ybakXnyeCuW7y5AxQrjRMnxwMm0JIjtMi9lGs9MNWbsJURNHtMmwyNYvbJjmpdsnAKBaUGcCUcZM1Z924WbPDZ1XHlyQaLjkYIit1xDH7Ncjz9AAc77kPc2Sxr9ud1Cp)

Mapbox tiles are a bit clearer than OpenStreetMap tiles, which are a bit more blurry. I also chose Mapbox over OpenStreetMap because Mapbox includes solid dots on the cities which is a nice contrast to the lighter colored background. The size of the red circular highlights are sized the way they are so they do not take up too much space on the map, but big enough so that they surround the city's dot on the map if it has one.
