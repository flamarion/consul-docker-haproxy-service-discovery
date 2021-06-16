You don't need to build this image if you use the Docker Compose, but if you decided to run the HAProxy alone, you can build and run it.


`docker build -t haproxy:flama .`

`docker run -rm --network="consul" -p 9090:9090 -p 8080:80 haproxy:flama`
