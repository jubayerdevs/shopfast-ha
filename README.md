ShopFast High-Availability Web Service with Load Balancing
Name: Jubayer Ahmed
Batch: 14
Chosen Option: Option B – NGINX Load Balancer (Local)
1. Project Overview
This project demonstrates a high-availability web service using open-source NGINX and two web-server containers. The load balancer distributes HTTP requests between two simulated Availability Zones. The application servers expose a dedicated /health endpoint, and NGINX uses passive failure detection to avoid repeatedly routing traffic to a failing backend.
2. Architecture
            User / Browser
                    |
             localhost:8080
                    |
             NGINX Load Balancer
                 /       \
                /         \
       Simulated AZ-A   Simulated AZ-B
          web-a            web-b
       Instance 1        Instance 2
          /health          /health
The AZs are represented by separate Docker bridge networks. The load balancer connects to both networks. Backend ports are not published directly to the host.
Limitation: Both simulated AZs run on one physical host and therefore do not provide real infrastructure-level availability-zone redundancy.
3. Technologies
Docker and Docker Compose
NGINX open-source
Linux
Git and GitHub
curl for HTTP and failover testing
4. Setup Instructions
Install Docker Engine and the Docker Compose plugin.
Clone this repository.
Navigate to the project directory.
Validate the Compose configuration:
docker compose config
Start the services:
docker compose up -d
Check service health:
docker compose ps
Validate NGINX:
docker exec shopfast-lb nginx -t
Open http://localhost:8080.
5. Health Checks
The backend servers expose /health, which returns HTTP 200 when the web server responds.
Docker checks the endpoint every 5 seconds, with a 2-second timeout and two retries. NGINX uses max_fails=1 and fail_timeout=10s for passive upstream failure detection.
Docker health status and NGINX upstream health are separate mechanisms. NGINX OSS does not automatically use Docker's health status to remove an upstream.
6. Test Results and Evidence
T1 – Health Status
Command: docker compose ps
Expected result: Both backend containers are healthy.
Actual result: Record your observed result here.
T2 – Load Distribution
Command: curl -s http://localhost:8080/instance-info
Expected result: Responses identify both Instance 1 and Instance 2.
Actual result: Record your observed result here.
T3 – Browser Test
URL: http://localhost:8080
Expected result: The ShopFast page displays backend identity information.
Actual result: Record your observed result here.
T4 – AZ-A Failure
Command: docker compose stop web-a
Expected result: Requests are served by AZ-B after failure detection. Record any errors during the transition.
Actual result: Record your observed result here.
T5 – Recovery
Command: docker compose start web-a
Expected result: AZ-A becomes available again and traffic resumes reaching both backends.
Actual result: Record your observed result here.
T6 – Access Logs
Command: docker compose logs --tail=50 load-balancer
Expected result: Logs show incoming requests and their selected upstream addresses.
Actual result: Record your observed result here.
7. Reflection
7.1 AWS ALB vs. NGINX
AWS Application Load Balancer is a managed service that integrates with EC2, target groups, active health checks, and multiple Availability Zones. AWS manages much of its underlying infrastructure, reducing operational effort. However, it requires an AWS account and can incur charges.
NGINX is an open-source web server and reverse proxy that can perform load balancing with low resource requirements. It is suitable for local learning and provides control over routing and logging. Its limitations include the need to manage configuration and infrastructure yourself. Open-source NGINX uses passive upstream health checks rather than the native active target health checks available in AWS ALB. This project simulates AZs using separate Docker networks, so it does not offer true physical or host-level redundancy.
7.2 What if the Load Balancer Fails?
If the only NGINX container or its host fails, users cannot reach either backend through the configured entry point. The load balancer is therefore a single point of failure in this local implementation.
In production, redundancy can be achieved using multiple load-balancer instances across separate Availability Zones, with a managed service or highly available virtual IP routing traffic to a healthy load balancer. Monitoring, automated recovery and tested failover procedures also help reduce downtime.
7.3 Problem Faced and Solution
Replace this example with a real problem you encountered.
One challenge was understanding the difference between backend health checks and load-balancer failure detection. Docker can report a container as unhealthy, but the standard NGINX open-source upstream configuration does not automatically remove that backend based on Docker health status. I investigated the behavior, configured NGINX passive failure detection and upstream retries, and tested the service by stopping and restarting a backend container. I used curl and the access logs to observe the results and document the limitations.
8. Cleanup
Stop and remove the containers and networks:
docker compose down
9. Conclusion
This project demonstrates load distribution, backend health monitoring, passive failure detection, recovery and access-log inspection using a local NGINX-based architecture.
