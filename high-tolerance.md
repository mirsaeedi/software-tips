# High Tolerance and Resiliency
The AWS Well-Architected Framework defines resilience as “the capability to recover when stressed by load (more requests for service), attacks (either accidental through a bug, or deliberate through intention), and failure of any component in the workload’s components.”

## Links
[Amazon: Minimizing correlated failures in distributed systems](https://aws.amazon.com/builders-library/minimizing-correlated-failures-in-distributed-systems/?did=ba_card&trk=ba_card)
TODO: [Amazon: Timeouts, retries, and backoff with jitter]([https://aws.amazon.com/builders-library/minimizing-correlated-failures-in-distributed-systems/?did=ba_card&trk=ba_card](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/))
* TODO: [AWS Developer Webinar Series: Five design patterns to build more resilient applications](https://www.youtube.com/watch?v=CDX7oQkuf3A)
## Infrastructure and Hardware

* To achieve fault tolerance, a typical Amazon service at that time ran on multiple physical servers behind a load balancer that distributed incoming requests among those servers.
* To deal with server failures, the load balancer sent a periodic health check (typically a small HTTP request to a well-known URL) to each server. If any of the servers failed to respond several times in a row, the load balancer took that server out of consideration for future requests.
* To deal with server failures, the load balancer sent a periodic health check (typically a small HTTP request to a well-known URL) to each server. If any of the servers failed to respond several times in a row, the load balancer took that server out of consideration for future requests.
* If the system needed persistent storage, a similar technique was used to set up a pair of replicated relational databases
* If each server has a 0.01% chance of failing on any given day, then by running on two servers, the probability of both of them failing on the same day is 0.000001%. We go from a somewhat unlikely event, to an incredibly unlikely one!
* We only get the benefit of compounded probabilities if the failures are uncorrelated, that is, if individual failures are always independent. Unfortunately, in the real world that isn’t always the case. A single underlying cause can trigger multiple failures.
* At Amazon, we approach reducing such risks by organizing the underlying infrastructure into multiple Regions and Availability Zones. Each Region is designed to be isolated from all the other Regions, and it consists of multiple Availability Zones. Each Availability Zone consists of one or more discrete data centers that have redundant power, network, and connectivity, and are physically separated by a meaningful distance from other Availability Zones.
*  Because any change brings with it a risk of failure, infrastructure teams configure their continuous deployment systems to avoid deploying changes to multiple Availability Zones at the same time. 
*  Here are just a few examples of using jitter to reduce risks of correlated behaviors:
    * Varying the frequency at which each server runs housekeeping jobs.
    * Adding a small amount of randomness to the delay before a failed API is retried. To learn more about this, go to Marc Brooker’s Amazon Builders’ Library article on adding jitter to retries: Timeouts, retries, and backoff with jitter.
    * If the system contains mode shifts (e.g., refreshing its state from a remote data store), varying the thresholds at which such mode shifts occur.
    * Varying expiration times of short-term credentials and secrets.
    * Varying cache time to live (TTL) across servers.
    * Where applicable, adding a small amount of variability to the various resource limits, such as Java virtual machine (JVM) heap and connection pool sizes.


## Retries

