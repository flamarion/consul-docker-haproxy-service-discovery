# consul-haproxy-service-discovery

I'm going to write better instructions but if you want to run this lab you just need to following the steps and requirements.

* Requirements
Run the `consul-server` first and preferable being the single container running using de default Docker network when you start the lab.
If for any reason you're not sure which IP adddress the container is using, just run the command:

`docker inspect -f "{{ .NetworkSettings.IPAddress }}" 8a0dde8ba70f`

Of course, you need to know the Consul Server container name or id


1. [Build and run the consul server](consul-server/README.md) **make sure to get the IP address &#8593;**
2. [Build and run the haproxy](haproxy/README.md)
3. [Build and run the webservers and consul clients](web-service/README.md)

The [haproxy config file](haproxy/haproxy.cfg) and [consul client config file](web-service/client.json) use the Consul Server ip address, so make sure to adjust it accordingly.

The idea of the lab is showing the Service Discovery capability registering nodes to the Load Balancer according to they come up, the lab is based on [this](https://learn.hashicorp.com/tutorials/consul/load-balancing-nginx-plus?in=consul/load-balancing) tutorial and is very basic (no one security capabilities is being used)

After startup the Consul Server, you will be able to access the web interface via http://localhost:8500
After startup the HAProxy, you will be able to access the statistics page via http://localhost:9090 and the load balancer functionality will be available via http://localhost
To test the service discovery you can run as many [webservers](#3) as you want and observe them being registered on the load balancer and being removed as you kill the containers they're running.
