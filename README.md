# consul-haproxy-service-discovery

I'm going to write better instructions but if you want to run this lab you just need to following the steps.

1. Clone the repo and access the directory `consul-haproxy-service-discovery`

 

```
 cd consul-haproxy-service-discovery/
 ```

2. Build the `consul-server` image

 

```
 cd consul-server
 docker build --rm -t consul:flama .
 ```

 I'm using my own tag `flama` but you can use any other, but don't forget to update the `docker-compose.yml` .
 This image alone runs a single node of a Consul Cluster, but the configuration file `server.json` expects at least 3 nodes, so if you decided to run this container out of Docker Compose, you many need to create 3 different containers with 3 different names ( `consul-server1` , `consul-server2` and `consul-server1` ) 

3. Build the Webserver Image
 

```
 cd ../consul-client-webserver
 docker build -t webserver:flama .
 ```

 Here I'm also using my own tag and the same instructions used in step #1 applies with the exception of the container name, for the client we don't want a container name since we will scale the number of containers and it will cause problems and for this purpose the container name doesn't make any difference.
 This container is kinda special : D because to simulate a Consul node that runs a Consul Client along with the Application we need to find a way to run at least two process, the Application (simulated here by the dummy webserver) and the Consul Client but Docker alone doesn't implement the concept of `sidecar` containers like we have in Kubernetes Pods, so we have to improvise.
 To make it happen, I'm using the `Supervisord` to help me spawn the Apache process as well as the Consul Client process. It's a super simple configuration but it's functional.

4. Gossip Key and TLS Certs.
 
 You can use the certs and the key like they are, however, if you decide to change the gossip key and recreate the TLS certificates, make sure to updated all `json` files for the Consul Server and Consul Client.

5. Running the Lab
 
 

```
 cd ../
 docker-compose up -d
 ```

 The output should be something like this:

 

```
 docker-compose up -d
Docker Compose is now in the Docker CLI, try `docker compose up`

Pulling haproxy (haproxy:2.4)...
2.4: Pulling from library/haproxy
69692152171a: Already exists
4a113359dddc: Already exists
e62dbde04094: Already exists
4ba498b86933: Already exists
Digest: sha256:2730fd2c99f633d4b515e69935b3f97d47dc0469354329f657a81e94212c1de4
Status: Downloaded newer image for haproxy:2.4
Creating haproxy_consul-client-webserver_1 ... done
Creating consul-server1                    ... done
Creating consul-server3                    ... done
Creating haproxy                           ... done
Creating consul-server2                    ... done
```

6. Checking the results
 
 

```
 source env.sh
 consul members
Node            Address          Status  Type    Build  Protocol  DC   Segment
consul-server1  172.29.0.4:8301  alive   server  1.9.6  2         dc1  <all>
consul-server2  172.29.0.3:8301  alive   server  1.9.6  2         dc1  <all>
consul-server3  172.29.0.5:8301  alive   server  1.9.6  2         dc1  <all>
10321b3ca23e    172.29.0.6:8301  alive   client  1.9.6  2         dc1  <default>
```
Access the Consul Cluster via http://localhost:8500

Access the HAProxy statistics page via http://localhost:9090

Test the Load Balancer via HA Proxy via http://localhost:8080

7. Testing

_Scaling Up_

While run the next command make sure to watch the HAProxy statistics page (configured to refresh each 2 seconds) to see the the nodes being added automatically to the Load Balancer using the Cluster Service Discovery Capability.

```
docker-compose up --scale consul-client-webserver=5 -d
Docker Compose is now in the Docker CLI, try `docker compose up`

consul-server2 is up-to-date
consul-server3 is up-to-date
haproxy is up-to-date
consul-server1 is up-to-date
Creating haproxy_consul-client-webserver_2 ... done
Creating haproxy_consul-client-webserver_3 ... done
Creating haproxy_consul-client-webserver_4 ... done
Creating haproxy_consul-client-webserver_5 ... done
```

_Scaling Down_

```
docker-compose up --scale consul-client-webserver=2 -d
Docker Compose is now in the Docker CLI, try `docker compose up`

Stopping and removing haproxy_consul-client-webserver_3 ...
consul-server3 is up-to-date
Stopping and removing haproxy_consul-client-webserver_4 ...
Stopping and removing haproxy_consul-client-webserver_5 ...
Stopping and removing haproxy_consul-client-webserver_3 ... done
Stopping and removing haproxy_consul-client-webserver_4 ... done
Stopping and removing haproxy_consul-client-webserver_5 ... done
```


8. Wrapping up
 You can add a webpage to each of your webservers and explore the consul configurations but this lab show very well how to create a very dynamic load balancer using Consul service discovery aligned to HA Proxy.

 To shutdown everything and remove all images just run the following command:

 ```
 docker-compose down --rmi all
Stopping haproxy_consul-client-webserver_2 ... done
Stopping haproxy_consul-client-webserver_1 ... done
Stopping consul-server3                    ... done
Stopping consul-server1                    ... done
Stopping consul-server2                    ... done
Stopping haproxy                           ... done
Removing haproxy_consul-client-webserver_2 ... done
Removing haproxy_consul-client-webserver_1 ... done
Removing consul-server3                    ... done
Removing consul-server1                    ... done
Removing consul-server2                    ... done
Removing haproxy                           ... done
Removing network consul
Removing image consul:flama
Removing image consul:flama
WARNING: Image consul:flama not found.
Removing image consul:flama
WARNING: Image consul:flama not found.
Removing image haproxy:2.4
Removing image webserver:flama
```


### References.

* https://www.consul.io/docs/agent/options
* https://learn.hashicorp.com/tutorials/consul/docker-compose-datacenter?in=consul/docker
* https://learn.hashicorp.com/collections/consul/production-deploy
* https://learn.hashicorp.com/tutorials/consul/load-balancing-haproxy?in=consul/load-balancing
* https://docs.docker.com/compose/networking/

