 1. A production envirionnement has to be scalable and the current solution can't handle automatically and easily an increasment of the trafic.
   Adding or removing nodes involves a manual configuration and a resart of the containers. The service will be down and that's not a acceptable !

2 Follow the instructions to add a new server node to the current configuration
2.1 Start a new container (=server node)
    docker run -d --name -s3 softengheigvd/webapp
2.2 Add new web server node (s3) to the configuration file of HAProxy
    server s3 <s3>:3000 check
2.3 Update the run.sh script
    sed -i 's/<s3>/$S3_PORT_3000_TCP_ADDR/g' /usr/local/etc/haproxy/haproxy.cfg
2.4 Rebuild ha container
    docker build -t softengheigvd/ha .
2.5 Run ha container
    docker run -d -p 80:80 -p 1936:1936 -p 9999:9999 --link s1 --link s2 --link s3 --name ha softengheigvd/ha

3. A better approach consit in a dynamic configuration of haproxy.cfg when adding or removing a node. This approach matches the requirement of a production environnement which is scalability.

4. The configuration of the load balancer has to be automatic. To archieve this, there is a tool called "Serf" for cluster membership, failure detection, and orchestration that is decentralized, fault-tolerant and highly available. It allows to add/remove a node dynamically.

5. The current solution is not able to run beside processes in a single container. Docker has been designed to process only one process per container.
   To bypass this limitation, we'll use a process supervisor called "s6". This tool allows more than one process running in parallel in docker.

6. In this current solution adding more web server nodes won't work as excepted, because adding node to the load balancer configuration is hardcoded.
   It needs an human action. So it definitley not dynamic ! A dynamic solution will be implemented using Handlebars and template (nodeJS).
   Each time a node is added/removed, the Sref handler script will generated a configuration file for HAProxy.