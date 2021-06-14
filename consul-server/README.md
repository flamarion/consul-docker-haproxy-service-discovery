`docker build --no-cache -t consul-server:latest .` 

`docker run -p 8500:8500 -p 8600:8600/udp consul-server` 
