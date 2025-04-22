# MongoDB Indexing Guide for Microcks Performance Optimization


Before applying these indexes, ensure you have:

1. A running Microcks instance that is connected to MongoDB. You can check detailed steps for installing microcks with docker and kubernetes here:
   - [Docker Compose Installation Guide](https://microcks.io/documentation/guides/installation/docker-compose/)
   - [Kubernetes Installation with Minikube and Helm](https://microcks.io/documentation/guides/installation/minikube-helm/)
2. Access to MongoDB with admin credentials

## Connecting to MongoDB

### Remote Connection

If your MongoDB instance is accessible remotely:

```shell
$ mongo -u <admin-username> -p <admin-password> --authenticationDatabase admin mongodb://<hostname>:<port>
```

### Kubernetes Deployment

If running in Kubernetes:

1. Identify your MongoDB pod:
   ```shell
   # If using the standard Microcks installation, the namespace will be 'microcks'
   $ kubectl get pods -n microcks | grep mongodb
   # Example output: microcks-mongodb-d584889cf-wnzzb
   ```

2. Obtain admin credentials from Kubernetes secrets:
   ```shell
   $ kubectl get secret microcks-mongodb-connection -n microcks -o jsonpath='{.data.adminUsername}' | base64 --decode
   $ kubectl get secret microcks-mongodb-connection -n microcks -o jsonpath='{.data.adminPassword}' | base64 --decode
   ```

3. Connect directly using admin credentials:
   ```shell
   For Kubernetes connection:

   $ kubectl exec -it <mongodb-pod-name> -n microcks -- mongo -u <admin-username> -p <admin-password> --authenticationDatabase admin
   # Example: kubectl exec -it microcks-mongodb-d584889cf-wnzzb -n microcks -- mongo -u admin -p password --authenticationDatabase admin
   ```
   
### Docker Deployment

For accessing the MongoDB shell, directly run:
```shell
$ docker exec -it microcks-db mongo
```

## Why Use Indexes?

As your MongoDB database grows, queries can become slower due to the increasing volume of data. Indexes significantly improve query performance by allowing MongoDB to locate documents more efficiently instead of scanning the entire collection.

## Key Indexes for Microcks

Microcks adopters can benefit from specific indexes to optimize query performance. Below are recommended indexes and how to create them.

### Creating Indexes

Run the following commands in your MongoDB shell to create indexes on key collections:

```javascript
// Optimize query performance for daily statistics
db.dailyStatistic.createIndex( {day: -1, serviceName: 1, serviceVersion: -1}, {name: "day-1_serviceName1_serviceVersion-1"} );

// Improve lookup efficiency for event messages
db.eventMessage.createIndex( {operationId: -1}, {name: "operationId-1"} );
db.eventMessage.createIndex( {testCaseId: -1}, {name: "testCaseId-1"} );

// Speed up request searches
db.request.createIndex( {operationId: -1}, {name: "operationId-1"} );
db.request.createIndex( {testCaseId: -1}, {name: "testCaseId-1"} );

// Enhance response retrieval
db.response.createIndex( {operationId: -1}, {name: "operationId-1"} );
db.response.createIndex( {testCaseId: -1}, {name: "testCaseId-1"} );

// Optimize test result queries
db.testResult.createIndex( {serviceId: -1}, {name: "serviceId-1"} );
db.testConformanceMetric.createIndex( {serviceId: -1}, {name: "serviceId-1"} );
```

### Removing Inefficient Indexes

If you find that certain indexes are not improving performance or consuming unnecessary resources, you can remove them using:

```javascript
db.dailyStatistic.dropIndex("day-1_serviceName1_serviceVersion-1");
db.eventMessage.dropIndex("operationId-1");
db.eventMessage.dropIndex("testCaseId-1");
db.request.dropIndex("operationId-1");
db.request.dropIndex("testCaseId-1");
db.response.dropIndex("operationId-1");
db.response.dropIndex("testCaseId-1");
db.testResult.dropIndex("serviceId-1");
db.testConformanceMetric.dropIndex("serviceId-1");
```


## Performance Results
We conducted tests on a sample Microcks database to evaluate the impact of indexes on query performance. Below are the results showing the performance differences between queries with and without indexes.
### Test Setup

- MongoDB version: 4.4.29
- Collection: dailyStatistic
- Query: Finding a specific service by day, name, and version
- Test dataset: Small test database with 7 documents

### With Index
Create index:
```javascript
db.dailyStatistic.createIndex(
  {day: -1, serviceName: 1, serviceVersion: -1}, 
  {name: "day-1_serviceName1_serviceVersion-1"}
);
```

Run the query with explain
```javascript

db.dailyStatistic.find({
  day: "20250205",
  serviceName: "Petstore API",
  serviceVersion: "1.0.0"
}).explain("executionStats");

// Full explain output (abbreviated):
{
  "queryPlanner": {
    // ...
    "winningPlan": {
      "stage": "FETCH",
      "inputStage": {
        "stage": "IXSCAN",
        "keyPattern": {
          "day": -1,
          "serviceName": 1,
          "serviceVersion": -1
        },
        "indexName": "day-1_serviceName1_serviceVersion-1"
        // ...
      }
    }
  },
  "executionStats": {
    "executionSuccess": true,
    "nReturned": 1,
    "executionTimeMillis": 25,
    "totalKeysExamined": 1,
    "totalDocsExamined": 1,
    // ...
  }
}
```

**Results:**

- Execution time: 25ms
- Documents examined: 1 (optimal)
- Query method: IXSCAN (using index)
- Keys examined: 1
- Works: 2

### Without Index
Drop the index:
```javascript
db.dailyStatistic.dropIndex("day-1_serviceName1_serviceVersion-1");
```

Run the same query without the index:
```javascript
db.dailyStatistic.find({
  day: "20250205",
  serviceName: "Petstore API",
  serviceVersion: "1.0.0"
}).explain("executionStats");

// Full explain output (abbreviated):
{
  "queryPlanner": {
    // ...
    "winningPlan": {
      "stage": "COLLSCAN",
      "filter": {
        "$and": [
          { "day": { "$eq": "20250205" } },
          { "serviceName": { "$eq": "Petstore API" } },
          { "serviceVersion": { "$eq": "1.0.0" } }
        ]
      },
      "direction": "forward"
    }
  },
  "executionStats": {
    "executionSuccess": true,
    "nReturned": 1,
    "executionTimeMillis": 37,
    "totalKeysExamined": 0,
    "totalDocsExamined": 7,
    // ...
  }
}
```
**Results:**

- Execution time: 37ms
- Documents examined: 7 (entire collection)
- Query method: COLLSCAN (full collection scan)
- Keys examined: 0
- Works: 9

### Code for Monitoring Index Performance
To monitor index usage in an ongoing production environment, you can use:
```javascript
db.dailyStatistic.aggregate([
  { $indexStats: {} }
]);
```

To check if specific queries are using indexes effectively:
```javascript
db.setProfilingLevel(1, { slowms: 100 });  // Log queries taking > 100ms
```

After some time, check the slow query log:
```javascript
db.system.profile.find(
  { millis: { $gt: 100 }, ns: "microcks.dailyStatistic" },
  { op: 1, millis: 1, query: 1, planSummary: 1 }
).sort({ millis: -1 });
```

### Performance Analysis
Even with our small test dataset of only 7 documents, we observed:

- 48% increase in execution time when the index was removed
- 7x more documents examined without the index
- 4.5x more internal operations (works) performed

### Understanding docsExamined
The totalDocsExamined metric shows how many documents MongoDB had to read and check during execution:

- **With index (1 document examined)**: MongoDB read exactly the document it needed
- **Without index (7 documents examined)**: MongoDB read every document in the collection

This metric directly impacts:

- I/O operations (disk reads)
- Memory usage during query execution
- CPU usage (checking conditions)

As your collection grows, the difference between examining 1 document versus millions becomes critical for performance.

### Key Observations

- The MongoDB query planner correctly chose the index when available
- The indexed query examined exactly the documents it needed (1 document)
- Without the index, MongoDB had to scan every document in the collection
- Even with minimal data, the performance difference was measurable


## Best Practices for Indexing

* **Analyze Query Patterns:** Use MongoDB's `explain()` method to see how queries are executed and determine if an index is beneficial.
* **Limit the Number of Indexes:** Each index requires storage and updates when data changes, so avoid excessive indexing.
* **Use Compound Indexes Wisely:** Indexes with multiple fields (like the one on `dailyStatistic`) help optimize queries that filter on multiple attributes.
* **Monitor Performance Regularly:** Use `db.collection.stats()` and `db.collection.getIndexes()` to check index efficiency.

## Next Steps

1. **Validate Performance Gains:** After adding indexes, compare query performance using `explain()`.
2. **Gather Community Feedback:** If you're a Microcks adopter, share your indexing strategies and challenges.
3. **Explore Additional Optimizations:** Consider sharding and replica sets for large-scale deployments.

By implementing these indexing strategies, Microcks users can ensure faster query execution and an optimized database experience! 🚀