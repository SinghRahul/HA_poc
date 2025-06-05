# Multi-Application, Multi-Region Cluster Architecture

This document describes a high-availability architecture for an automobile company using a microservices approach. The setup ensures resilience, scalability, and efficient failover across multiple geographical zones and clusters.

## Assumptions

- The architecture follows a microservices design, connecting to various applications/services for authentication, authorization, entity creation, and updates.
- The company manages multiple products (referred to as "entities"), such as wheels, gears, shafts, motors, etc.

## Architecture Overview
![Multi-Region Architecture](arch.svg)

- **Geographical Zones:**  
  - Zone 1  
    - Cluster 1A  
    - Cluster 1B  
  - Zone 2  
    - Cluster 2A  
    - Cluster 2B  

- **Application Deployment:**  
  All applications are deployed and running in every cluster to ensure high availability.

- **Traffic Management:**  
  - **Load Balancer N1** (Nginx instance):  
    - Receives all incoming traffic.
    - Performs geo-based routing to Nginx 1A (Zone 1) or Nginx 2A (Zone 2).
  - **Nginx 1A and Nginx 2A:**  
    - Load balance requests between gateway endpoints within their respective zones using a round-robin algorithm.
    - Both active and passive health checks are enabled to monitor gateway health.

- **Database Replication:**  
  - Databases are replicated across zones for disaster recovery.
  - Within each zone, databases are locally replicated for redundancy.
  - All write operations are handled by a single designated cluster at any time.
  - Leases are created for new or updated entities to manage write consistency.

- **Gateway Routing:**  
  - Gateway instances ensure that write operations (`isWriteOperation = true`) are always routed to the `create/update_service` in the active region.

## Failover Strategy

Failover can be initiated manually in two ways:

1. **Cluster Failover (Within a Zone):**
   - Update Nginx configuration in Nginx 1A or 2A to route all traffic to another healthy cluster within the same zone.
   - This is achieved by pointing to other gateway instances in the zone.

2. **Zone/Region Failover:**
   - Update Nginx configuration in Load Balancer N1 to redirect traffic from Nginx 1A to Nginx 2A (or vice versa), effectively switching zones.

- **Health Checks:**  
  Nginx 1A and Nginx 2A use passive health checks to ensure requests are only sent to healthy instances. Unhealthy upstreams are automatically marked and bypassed.

## Automated Failover

Automated failover can be implemented using a monitoring tool that:

- Continuously checks Nginx logs for API names, response codes, and response times.
- Detects failures and automatically updates Nginx configurations to redirect traffic to healthy clusters or zones.

**Features:**  
This architecture ensures high availability, fault tolerance, and minimal downtime for critical services by leveraging geo-distributed clusters, robust load balancing, and automated failover mechanisms.
