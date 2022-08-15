# Logging

## Links
* [Amazon: Instrumenting distributed systems for operational visibility](https://aws.amazon.com/builders-library/instrumenting-distributed-systems-for-operational-visibility/)

## Amazon Practices

* The team who owns a service is known as the service owner.
* As the owners, teams set goals on the operational performance of all of the associated services.
* To set goals and get that visibility, teams must instrument systems.
* Instrumentation feeds data into operational dashboards, so that operators can view real-time metrics. It also feeds data into alarms, which trigger and engage operators when the system is behaving in an unexpected way. The operators use the detailed output of the instrumentation to quickly diagnose why things went wrong.
* We want each component to emit metrics about its behavior.
* We also want metrics on how each component perceives the behavior of other components.
* Here’s a sample code, annotated with a few of the questions I’d need to be able to answer about the production system as a whole, or for a particular request:
```Java
public GetProductInfoResponse getProductInfo(GetProductInfoRequest request) {

  // Which product are we looking up?
  // Who called the API? What product category is this in?

  // Did we find the item in the local cache?
  ProductInfo info = localCache.get(request.getProductId());
  
  if (info == null) {
    // Was the item in the remote cache?
    // How long did it take to read from the remote cache?
    // How long did it take to deserialize the object from the cache?
    info = remoteCache.get(request.getProductId());
	
    // How full is the local cache?
    localCache.put(info);
  }
  
  // finally check the database if we didn't have it in either cache
  if (info == null) {
    // How long did the database query take?
    // Did the query succeed? 
    // If it failed, is it because it timed out? Or was it an invalid query? Did we lose our database connection?
    // If it timed out, was our connection pool full? Did we fail to connect to the database? Or was it just slow to respond?
    info = db.query(request.getProductId());
	
    // How long did populating the caches take? 
    // Were they full and did they evict other items? 
    localCache.put(info);
    remoteCache.put(info);
  }
  
  // How big was this product info object? 
  return info;
}
```
* The code for answering all of those questions (and more) is quite a bit longer than the actual business logic. 
* To piece together the puzzle, we find it helpful to pull together in one place all of the measurements about all of these systems. Before we can do that, each service must be instrumented to record a trace ID for each task, and to propagate that trace ID to each other service that collaborates on that task. 
* While aggregate timers and metrics might help us rule out causes or highlight an area of investigation, they don’t always provide a complete explanation. 
* At this point, we look at the raw, detailed log data emitted by the service for that time window. The raw logs then show the source of problem.
* We most commonly instrument our services to emit two types of log data: 
  * request data: Is typically represented as a single structured log entry for each unit of work. This data contains properties about the request and who made the request, what the request was for, counters of how often things happened, and timers of how long things took. The request log serves as an audit log and a trace for everything that happened in the service.
  * debugging data: Includes unstructured or loosely structured data of whatever debugging lines the application emits. 
* Although it can be expensive to log so much detail about every request, at Amazon we find that it is incredibly important to do. After all, we need to investigate availability blips, latency spikes, and customer-reported problems. Without detailed logs, we can’t give customers answers, and we won't be able to improve their service.  
* Plan for a way to log at increased verbosity. For troubleshooting some kinds of problems, the log will not have enough detail about problematic requests to figure out why they failed. That information might be available in the service, but the volume of information might be too great to justify logging all the time. It can be helpful to have a configuration knob that you can dial to increase the log’s verbosity temporarily while you investigate an issue. You might turn the knob on individual hosts, or for individual clients, or at a sampling rate across the fleet. It’s important to remember to turn the knob back down when done.


## What to log
* Log the availability and latency of all of dependencies.
* Break out dependency metrics per call, per resource, per status code, etc. If we interact with the same dependency multiple times in the same unit of work, we include metrics about each call separately, and make it clear which resource each request was interacting with. For example, when calling Amazon DynamoDB, some teams have found it helpful to include timing and latency metrics per table, as well as per error code, and even per the number of retries. This make it easier to troubleshoot cases where the service was slow from retries due to conditional check failures. These metrics also revealed cases where client-perceived latency increases were actually due to throttling retries or paginating through a result set, and not from packet loss or network latency.
* Record memory queue depths when accessing them.
* Add an additional counter for every error reason.
* Log a trace ID and propagate it in backend calls.
*  Log important metadata about the unit of work.
