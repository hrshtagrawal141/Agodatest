## Design Diagram


[
![Untitled Diagram](https://user-images.githubusercontent.com/52158609/60016507-0c7f8f00-96a4-11e9-95a4-094717d4ca8e.jpg)
](url)

### Running the application

cd Agoda/

Agoda > mvn spring-boot:run

### Get weekly summary API

### Running the tests
Agoda/> mvn test

## Available Endpoint

GET /getWeeklySummary  (Sample Input Request/Response):

project1.json: 

Sample Request:
{
"project_id" : "project1",
"from_week" : "2019W01",
"to_week" : "2019W03",
"types" : ["bug"],
"states" : ["open"]
}

Response-


 {
   
  "project_id" : "project1",

   "weekly_summaries" : [

      {

         "week" : "2019W01",

         "state_summaries" : [

            {
               "state" : "open",
               "count" : 1,
               "issues" : [
                  {
                     "issue_id" : "issue1",
                     "type" : "bug"
                  }
               ]

            }
         ]
      },
      {
         "week" : "2019W02",
         "state_summaries" : [
            {
               "state" : "open",
               "count" : 1,
               "issues" : [
                  {
                     "issue_id" : "issue2",
                     "type" : "bug"
                  }
               ]
            }
         ]
      },
      {
         "week" : "2019W03",
         "state_summaries" : []
      },
      {
         "week" : "2019W04",
         "state_summaries" : [
            {
               "state" : "open",
               "count" : 2,
               "issues" : [
                  {
                     "issue_id" : "issue1",
                     "type" : "bug"
                  },
                  {
                     "issue_id" : "issue2",
                     "type" : "bug"
                  }
               ]
            }
         ]
      }
   ]
 }

## Design Explained:

Our Bug tracking system has 3 main things:

1. Populate local data structures for input requests.
2. Scheduler Service
3. Get Weekly Summary endpoint

### Input Request:

Each input request is validated and once the validation is successful, we maintain 2 HashMap for storing the data

New Project Map: Each input project is initially put into new project map with frequency 
set to 1 and incremented for subsequent requests of that particular project.

Old Project Map: This map holds already synched projects and its frequency similar to New
Project Map.


### Scheduler Service: 

Sync service is responsible for communicating with third party getIssues API every minute 
(according to properties in application.yml, default is 60000 ms = 1 min) and updates the local data store.

The key idea in our implementation is, given that we can invoke getIssues API only once per minute,
How do we choose project and sync the data in the local data store?

The idea is, we choose the maximum frequency project from the New Project Map. Once we sync it,
we remove that project from New Project Map and put it into Old Project Map resetting its frequency.

In case if the New Project Map is empty, which means we have already synched all the projects at least once.
We then again choose the maximum frequency project from Old Projects Map and sync it. Once it is completed,
we will reset its frequency in the Old Project map.

Once the project is chosen, sync service communicates with third Party getIssues API and updates the 
response (can be new/diff of existing) in the local data store.

### Get Weekly Summary:

For the input request, it fetches the available response from the local data store and returns it.


### Concurrency Handling Explained

Concurrency Hash Map is used. In our system, we have multiple readers (getWeeklySummary) and 1 writer (Sync service).
It allows concurrent access to the map, where part of the map is only getting locked while adding or updating the map.
So ConcurrentHashMap allows concurrent threads to read the value without locking at all, to improve the performance.


## Time Complexity

GET /getWeeklySummary:

For a particular week, it takes O(1) time to fetch the issues from the local data store. 
Overall, it takes O(weekDiff), where weekDiff=ToWeek-FromWeek+1, to return the entire response. 


