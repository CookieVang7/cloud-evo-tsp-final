# Cloud Computing 4610 Final - The Traveling Salesmen Problem (TSP) and Evolutionary Computation
## Chineng "Cookie" Vang
##### Thanks to Nic McPhee, Peter Dolan, Jack Perala, and Rob Beane


# Table of Contents
- [Overview-Purpose](#overview-purpose)
- [Solution-Structure](#solution-structure)
- [User Documentation](#user-documentation)
- [Table Structure](#table-structure)
- [About Lambda Functions](#about-lambda-functions)
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
- [Appendix I (Lambda function code)](#appendix-i-lambda-function-code)
    - [`GetCityData()`](#getcitydata-code)
    - [`GenerateRandomRoute()`](#generaterandomroute-code)
    - [`GetBestRoutes()`](#getbestroutes-code)
    - [`GetRouteById()`](#getroutebyid-code)
    - [`MutateRoute()`](#mutateroute-code)

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

### About Lambda Functions:

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

### Appendix I (Lambda function code)

#### `GetCityData()`: 

```
const AWS = require('aws-sdk');
const ddb = new AWS.DynamoDB.DocumentClient();
 
exports.handler = (event, context, callback) => {
 
    getCityData()
        .then(dbResults => {
            const minnesotaCities = dbResults.Item.cities; //grabbing just the cities from the object in the database
            callback(null, {
                statusCode: 201,
                body: JSON.stringify(minnesotaCities),
                headers: {
                    'Access-Control-Allow-Origin': '*'
                }
            });
        })
        .catch(err => {
            console.log(`Problem getting city data from.`);
            console.error(err);
            errorResponse(err.message, context.awsRequestId, callback);
        });
};
 
/*
Gets the 'Minnesota' object from the first table/database. The object
includes the cities and their distances from each other
*/
function getCityData() {
    return ddb.get({
        TableName: 'cityDistance_data',
        Key: {region: 'Minnesota'}, //Specifies the key to look at in the object
    }).promise();
}
 
function errorResponse(errorMessage, awsRequestId, callback) {
  callback(null, {
    statusCode: 500,
    body: JSON.stringify({
      Error: errorMessage,
      Reference: awsRequestId,
    }),
    headers: {
      'Access-Control-Allow-Origin': '*',
    },
  });
}
```
#### `GenerateRandomRoute()`:
```
const AWS = require('aws-sdk');
const ddb = new AWS.DynamoDB.DocumentClient();
const randomBytes = require('crypto').randomBytes;
 
exports.handler = (event, context, callback) => {
    const requestBody = JSON.parse(event.body);
    const runId = requestBody.runId;
    const generation = requestBody.generation;
    const runGen = runId + '#' + generation;
    
    generateRandomRoute(runId, generation, callback, runGen);
};
 
/*
Generates a random route based off of the cities and distances object in the 
cityDistance_data table. This method runs when getCityData gets 
information back from the database A callback function is passed as a parameter 
which will echo back the routeId and length. The callback function after the 
put request is done
*/
function generateRandomRoute(runId, generation, callback, runGen){
    getCityData().then(minnesotaObject =>{ //getCityData is a promise object, when that function returns the 
    //object from the database, then the following code will be executed
        
        const cityDistances = minnesotaObject.Item.distances; 
        const numberOfCities = Object.keys(minnesotaObject.Item.cities).length;
        
        const arrayOfCities = new Array(); //creating an empty array to be populated and randomized
        populateCityArray(arrayOfCities,numberOfCities); //now the array has entries [0,1,...,n-1]
        cityRandomizer(arrayOfCities); //randomizing the cities
       
        const routeId = toUrlString(randomBytes(16));
        const routeDistance = calculateDistance(arrayOfCities,cityDistances);
        
        return ddb.put( //the put request that enters the route information in the evo-tsp-routes database/table
        {TableName: 'evo-tsp-routes',
        Item: {
        // runId: runId,
        // generation: generation,
        runGen: runGen,
        routeId: routeId,
        route: arrayOfCities,
        len: routeDistance},}).promise().then(dbResults => { //"length" is what I named the sort key in the "routes" table
            callback(null, { //what the database will tell the user
                statusCode: 201,
                body: JSON.stringify({
                    routeId: routeId,
                    length: routeDistance,
                }),
                headers: {
                    'Access-Control-Allow-Origin': '*'
                }
            });
        });
    });
}
 
/*
Gets the 'Minnesota' object from the first table/database. The object
includes the cities and their distances from each other
*/
function getCityData() {
    return ddb.get({
        TableName: 'cityDistance_data',
        Key: {region: 'Minnesota'}, 
    }).promise();
}
 
/*
Takes in a randomized array and calculates the total distance of the route by 
referencing the distance portion of the minnesotaObject
*/
function calculateDistance(array, cityDistances){
    let totalDistance = 0;
    for (let i = 0; i < array.length; i++){
        if (i == array.length-1){
            const finalCity = array[i];
            const startCity = array[0];
            const homeTripDistance = cityDistances[finalCity][startCity];
            totalDistance = totalDistance + homeTripDistance;
        }
        else {
            const city1 = array[i];
            const city2 = array[i+1];
            const aDistance = cityDistances[city1][city2];
            totalDistance = totalDistance + aDistance;
        }
    }
    return totalDistance;
}
 
/*
Populates an array with numbers 0-(n-1)
*/
function populateCityArray(arrayOfCities, numberOfCities){
    for (let i=0; i < numberOfCities; i++){
        arrayOfCities[i] = i;
    }
}
 
/*
Randomizes the array [0,1,....n-1]
*/
function cityRandomizer(array){
    for (let i = array.length - 1; i > 0; i--) {
        let j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
}
 
function toUrlString(buffer) {
    return buffer.toString('base64')
        .replace(/\+/g, '-')
        .replace(/\//g, '_')
        .replace(/=/g, '');
}
 
function errorResponse(errorMessage, awsRequestId, callback) {
  callback(null, {
    statusCode: 500,
    body: JSON.stringify({
      Error: errorMessage,
      Reference: awsRequestId,
    }),
    headers: {
      'Access-Control-Allow-Origin': '*',
    },
  });
}
```
#### `GetBestRoutes()`:
```
const AWS = require('aws-sdk');
const ddb = new AWS.DynamoDB.DocumentClient();
 
exports.handler = (event, context, callback) => {
    const queryStringParameters = event.queryStringParameters;
    const runId = queryStringParameters.runId;
    const generation = queryStringParameters.generation;
    const numToReturn = queryStringParameters.numToReturn;
    
    
    getBestRoutes(runId, generation, numToReturn)
        .then(dbResults => {
            const bestRoutes = dbResults.Items;
            callback(null, {
                statusCode: 201,
                body: JSON.stringify(bestRoutes),
                headers: {
                    'Access-Control-Allow-Origin': '*'
                }
            });
        })
        .catch(err => {
            console.log(`Problem getting best runs for generation ${generation} of ${runId}.`);
            console.error(err);
            errorResponse(err.message, context.awsRequestId, callback);
        });
};
 
function getBestRoutes(runId, generation, numToReturn) { //gets the best/shortest routes when invoked and given parameters
    const runGen = runId + "#" + generation; //runGen which is passed into the query to find the specified number of routes
    return ddb.query({
        TableName: 'evo-tsp-routes',
        IndexName: 'runGen-len-index', //specifying the index to search
        ProjectionExpression: "routeId, len, route, runGen", //identifies fields to be returned
        KeyConditionExpression: "runGen = :runGen", //getting back the routes with the particular runGen specified on line 
        ExpressionAttributeValues: {
                ":runGen": runGen,
            },
        Limit: numToReturn
    }).promise();
}
 
function errorResponse(errorMessage, awsRequestId, callback) {
  callback(null, {
    statusCode: 500,
    body: JSON.stringify({
      Error: errorMessage,
      Reference: awsRequestId,
    }),
    headers: {
      'Access-Control-Allow-Origin': '*',
    },
  });
}
```
#### `GetRouteById()`:
```
const AWS = require('aws-sdk');
const ddb = new AWS.DynamoDB.DocumentClient();
 
exports.handler = (event, context, callback) => {
    const pathParameters = event.pathParameters;
    const userRouteId = pathParameters.routeId;
    
    getRouteById(userRouteId)
        .then(dbResults => {
            if (dbResults == null){
                callback(null, {
                statusCode: 400,
                body: JSON.stringify({
                Error: "Response from database failed",
                }),
                headers: {
                'Access-Control-Allow-Origin': '*',
                },
            });
            }
            callback(null, {
                statusCode: 201,
                body: JSON.stringify(dbResults.Item), //Item and not Items since only one thing is returned
                headers: {
                    'Access-Control-Allow-Origin': '*'
                }
            });
        })
        .catch(err => {
            console.log(`Problem getting route from ${userRouteId}.`);
            console.error(err);
            errorResponse(err.message, context.awsRequestId, callback);
        });
};
 
function getRouteById(userRouteId) { 
    return ddb.get({
        TableName: 'evo-tsp-routes',
        Key: {
          "routeId": userRouteId  
        },
    }).promise();
}
 
function errorResponse(errorMessage, awsRequestId, callback) {
  callback(null, {
    statusCode: 500,
    body: JSON.stringify({
      Error: errorMessage,
      Reference: awsRequestId,
    }),
    headers: {
      'Access-Control-Allow-Origin': '*',
    },
  });
}
```
#### `MutateRoute()`:
```
const AWS = require('aws-sdk');
const ddb = new AWS.DynamoDB.DocumentClient();
const randomBytes = require('crypto').randomBytes;
 
/*
 * Parts of this are already in working order, and
 * other parts (marked by "FILL THIS IN") need to be
 * done by you.
 * 
 * For reference, here's a list of all the functions that
 * you need to complete:
 * - `getDistanceData()`
 * - `getRouteById()`     
 * - `generateChildren()`  
 * - `addOneToGen()`   
 * - `recordChildren()`    
 * - `returnChildren`   
 * - `computeDistance`  
 */
 
// This will be called in response to a POST request.
// The routeId of the "parent" route will be
// provided in the body, along with the number
// of "children" (mutations) to make.
// Each child will be entered into the database,
// and we'll return an array of JSON objects
// that contain the "child" IDs and the length
// of those routes. To reduce computation on the
// client end, we'll also sort these by length,
// so the "shortest" route will be at the front
// of the return array.
//
// Since all we'll get is the routeId, we'll need
// to first get the full details of the route from
// the DB. This will include the generation, and
// we'll need to add one to that to create the
// generation of all the children.
exports.handler = (event, context, callback) => {
    const requestBody = JSON.parse(event.body);
    const routeId = requestBody.routeId;
    const numChildren = requestBody.numChildren;
    let lengthStoreThreshold = requestBody.lengthStoreThreshold;
    if (lengthStoreThreshold == null) {
        lengthStoreThreshold = Infinity;
    }
    
    // Batch writes in DynamoDB are restricted to at most 25 writes.
    // Because of that, I'm limiting this Lambda to only only handle
    // at most 25 mutations so that I can write them all to the DB
    // in a single batch write.
    //
    // If that irks you, you could create a function that creates
    // and stores a batch of at most 25, and then call it multiple
    // times to create the requested number of children. 
    if (numChildren > 25) {
        errorResponse("You can't generate more than 25 mutations at a time", context.awsRequestId, callback);
        return;
    }
 
    // Promise.all makes these two requests in parallel, and only returns
    // it's promise when both of them are complete. That is then sent
    // into a `.then()` chain that passes the results of each previous
    // step as the argument to the next step.
    
    Promise.all([getDistanceData(), getRouteById(routeId)])
        .then(([distanceData, parentRoute]) => generateChildren(distanceData.Item, parentRoute.Item, numChildren))
        .then(children => recordChildren(children, lengthStoreThreshold))
        .then(children => returnChildren(callback, children))
        .catch(err => {
            console.log("Problem mutating given parent route");
            console.error(err);
            errorResponse(err.message, context.awsRequestId, callback);
        });
};
 
/*
Get the city-distance object for the region 'Minnesota'
*/ 
function getDistanceData() {
    return ddb.get({
        TableName: 'cityDistance_data',
        Key: {region: 'Minnesota'}, 
    }).promise();
}
 
function getRouteById(userRouteId){
    return ddb.get({
        TableName: 'evo-tsp-routes',
        Key: {routeId: userRouteId}
    }).promise();
}
 
// Generate an array of new routes, each of which is a mutation
// of the given `parentRoute`. You essentially need to call
// `generateChild` repeatedly (`numChildren` times) and return
// the array of the resulting children. `generateChild` does
// most of the heavy lifting here, and this function should
// be quite short.
function generateChildren(distanceData, parentRoute, numChildren) {
    let arrayOfChildren = new Array();
    for (let i=0; i<numChildren; i++){ //creating a child route based off of the parent route and putting it into spot i of the array
        arrayOfChildren[i] = generateChild(distanceData, parentRoute); 
    }
    
    return arrayOfChildren;
    // You could just use a for-loop for this, or see
    // https://stackoverflow.com/a/42306160 for a nice description of
    // how to use of Array()/fill/map to generate the desired number of
    // children.
}
 
// This is complete and you shouldn't need to change it. You
// will need to implement `computeDistance()` and `addOneToGen()`
// to get it to work, though.
function generateChild(distanceData, parentRoute) {
    const oldPath = parentRoute.route;
    const numCities = oldPath.length;
    // These are a pair of random indices into the path s.t.
    // 0<=i<j<=N and j-i>2. The second condition ensures that the
    // length of the "middle section" has length at least 2, so that
    // reversing it actually changes the route. 
    const [i, j] = genSwapPoints(numCities);
    // The new "mutated" path is the old path with the "middle section"
    // (`slice(i, j)`) reversed. This implements a very simple TSP mutation
    // technique known as 2-opt (https://en.wikipedia.org/wiki/2-opt).
    const newPath = 
        oldPath.slice(0, i)
            .concat(oldPath.slice(i, j).reverse(), 
                    oldPath.slice(j));
    const len = computeDistance(distanceData.distances, newPath);
    const child = {
        routeId: newId(),
        runGen: addOneToGen(parentRoute.runGen),
        route: newPath,
        len: len,
    };
    return child;
}
 
// Generate a pair of random indices into the path s.t.
// 0<=i<j<=N and j-i>2. The second condition ensures that the
// length of the "middle section" has length at least 2, so that
// reversing it actually changes the route. 
function genSwapPoints(numCities) {
    let i = 0;
    let j = 0;
    while (j-i < 2) {
        i = Math.floor(Math.random() * numCities);
        j = Math.floor(Math.random() * (numCities+1));
    }
    return [i, j];
}
 
// Take a runId-generation string (`oldRunGen`) and
// return a new runId-generation string
// that has the generation component incremented by
// one. If, for example, we are given 'XYZ#17', we
// should return 'XYZ#18'.  
/*
Adds one to the generation in the runGen. Grabs the index of 
the #. Then creates substrings of the runId and generation.
Turns the generation into a number value, adds one, then 
creates a new string to be the updated runGen
*/
function addOneToGen(oldRunGen) {
    const poundIndex = oldRunGen.indexOf("#");
    const oldLength = oldRunGen.length;
    const genString = oldRunGen.substring(poundIndex + 1, oldLength); //grabbing the generation to be added which is still a string
    const genNumber = Number(genString); //the function Number() turns other types into a number
    const nextGen = genNumber + 1;
    
    const oldRunId = oldRunGen.substring(0,poundIndex); //grabbing the runId
    const newRunGen = oldRunId + '#' + nextGen;
    return newRunGen;
}
 
// Write all the children whose length
// is less than `lengthStoreThreshold` to the database. We only
// write new routes that are shorter than the threshold as a
// way of reducing the write load on the database, which makes
// it (much) less likely that we'll have writes fail because we've
// exceeded our default (free) provisioning.
function recordChildren(children, lengthStoreThreshold) {
    // Get just the children whose length is less than the threshold.
    //It returns an array with elements that match the condition
    const childrenToWrite = children.filter(child => child.len < lengthStoreThreshold);
 
 
    // FILL IN THE REST OF THIS.
    // You'll need to generate a batch request object (described
    // in the write-up) and then call `ddb.batchWrite()` to write
    // those children to the database.
    let childrenObject = { //creating an object to be populated with routes
      RequestItems:{
          'evo-tsp-routes':[] //specifies the table 
      }  
    };
    
    childrenToWrite.forEach(element => addRouteToObject(element, childrenObject));
    
    //function(err,data) is what's called when a response from the service is returned
    //If there were failed writes, we would get a semi-complex JSON object back
    //For this project, we're assuming that all writes succeeded
    //I referenced https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html#batchWrite-property
    ddb.batchWrite(childrenObject, function(err, data) {
        if (err) console.log(err);
        else console.log(data);
    }).promise();
    
    return childrenToWrite;
 
    // After the `ddb.batchWrite()` has completed, make sure you
    // return the `childrenToWrite` array.
    // We only want to return _those_ children (i.e., those good
    // enough to be written to the DB) instead of returning all
    // the generated children.
}
 
/*
A helper function that puts a child route from the childrenToWrite array into 
the JSON object. The JSON object will be passed as a parameter to the. The method
"push" puts a new object into an array. aRoute should have the fields 
routeId, runGen, len (length) and route. So that's all I need when I say 
Item: aRoute in the PutRequest
I referenced this website: https://www.w3schools.com/jsref/jsref_foreach.asp
*/
function addRouteToObject(aRoute, childrenObject){
    childrenObject.RequestItems['evo-tsp-routes'].push({ 
        PutRequest: {
            Item: aRoute
        }
    });
}
 
// Take the children that were good (short) enough to be written
// to the database. 
//
//   * You should "simplify" each child, converting it to a new
//     JSON object that only contains the `routeId` and `len` fields.
//   * You should sort the simplified children by length, so the
//     shortest is at the front of the array.
//   * Use `callback` to "return" that array of children as the
//     the result of this Lambda call, with status code 201 and
//     the 'Access-Control-Allow-Origin' line. 
function returnChildren(callback, children) {
    const sortedChildren = children.sort(sortByProperty("len")); //sorting the array of child routes by their length
    let simpleChildren = []; //creating an object to be populated with the routeId and length of child routes
    
    sortedChildren.forEach(child => addSimpleRouteToObject(child, simpleChildren));
 
    callback(null,{
       statusCode: 201,
       body: JSON.stringify(simpleChildren), 
       headers: {
           'Access-Control-Allow-Origin': '*'
       }
    });
}
 
/*
A helper function that puts the length and routeId of a child route into the 
JSON object to be returned to the user
*/
function addSimpleRouteToObject(aRoute, simpleChildren){
    simpleChildren.push({ 
        routeId: aRoute.routeId,
        len: aRoute.len,
        route: aRoute.route,
    
    });
}
 
 
/*
Sorts items in an array by a property. The property we're using is length.
Referenced from this website:
https://medium.com/@asadise/sorting-a-json-array-according-one-property-in-javascript-18b1d22cd9e9
*/
function sortByProperty(property){  
   return function(a,b){  
      if(a[property] > b[property])  
         return 1;  
      else if(a[property] < b[property])  
         return -1;  
  
      return 0;  
   };  
}
 
// Compute the length of the given route.
// function computeDistance(distances, route) {
//     // FILL THIS IN
 
//     // REMEMBER TO INCLUDE THE COST OF GOING FROM THE LAST
//     // CITY BACK TO THE FIRST!
// }
function computeDistance(distances, route){
    let totalDistance = 0;
    for (let i = 0; i < route.length; i++){
        if (i == route.length-1){
            const finalCity = route[i];
            const startCity = route[0];
            const homeTripDistance = distances[finalCity][startCity];
            totalDistance = totalDistance + homeTripDistance;
        }
        else {
            const city1 = route[i];
            const city2 = route[i+1];
            const aDistance = distances[city1][city2];
            totalDistance = totalDistance + aDistance;
        }
    }
    return totalDistance;
}
 
function newId() {
    return toUrlString(randomBytes(16));
}
 
function toUrlString(buffer) {
    return buffer.toString('base64')
        .replace(/\+/g, '-')
        .replace(/\//g, '_')
        .replace(/=/g, '');
}
 
function errorResponse(errorMessage, awsRequestId, callback) {
  callback(null, {
    statusCode: 500,
    body: JSON.stringify({
      Error: errorMessage,
      Reference: awsRequestId,
    }),
    headers: {
      'Access-Control-Allow-Origin': '*',
    },
  });
}
```
