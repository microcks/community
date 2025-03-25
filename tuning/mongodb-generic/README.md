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