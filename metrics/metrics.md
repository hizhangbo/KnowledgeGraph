## Metrics
- docker images
```
docker pull influxdb
docker pull grafana/grafana
docker pull chronograf
```
- docker run
```
docker run -d -p 8086:8086 -p 8083:8083 --expose 8090 --expose 8099 --name=influxdb influxdb
docker run -d -p 3000:3000 --name grafana grafana/grafana
docker run -d -p 8888:8888 -v chronograf:/var/lib/chronograf --name=chronograf chronograf
```
- influx
```
docker exec -it influxdb /bin/bash
influx
show databases;
create database test;
use test;
select * from test;
drop database test;
```
- chronograf
```
As of version 1.3, the web admin interface is no longer available in InfluxDB. 
The interface does not run on port 8083 and InfluxDB ignores the [admin] section in the configuration file if that section is present. 
Chronograf replaces the web admin interface with improved tooling for querying data, writing data, and database management. 
See Chronograf’s transition guide for more information.
```
