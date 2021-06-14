docker build -t haproxy .
docker run -p 9090:9090 -p 8080:80 haproxy
