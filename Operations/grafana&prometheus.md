1. 镜像获取
```shell
docker pull grafana/grafana
```
2. 启动容器
```shell
docker run -d --name=grafana -p 3000:3000 grafana/grafana
```
3. prometheus（开源系统监视和警报工具包）
```shell
docker pull prom/prometheus
docker run -d --name=prometheus -p 9090:9090 -v /usr/local/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```
4. Spring Boot 监控
### Installing
```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
  <version>${micrometer.version}</version>
</dependency>
```
### Configuring
```java
PrometheusMeterRegistry prometheusRegistry = new PrometheusMeterRegistry(PrometheusConfig.DEFAULT);

try {
    HttpServer server = HttpServer.create(new InetSocketAddress(8080), 0);
    server.createContext("/prometheus", httpExchange -> {
        String response = prometheusRegistry.scrape(); (1)
        httpExchange.sendResponseHeaders(200, response.getBytes().length);
        try (OutputStream os = httpExchange.getResponseBody()) {
            os.write(response.getBytes());
        }
    });

    new Thread(server::start).start();
} catch (IOException e) {
    throw new RuntimeException(e);
}
```
5. prometheus.yml
```properties
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
    - targets: ['127.0.0.1:9090']

  - job_name: 'spring-actuator'
    metrics_path: '/actuator/prometheus'
    scrape_interval: 5s
    static_configs:
    - targets: ['HOST_IP:8080']
```
6. Dashboard Template  
[Dashboard 模板](https://grafana.com/dashboards?dataSource=prometheus&search=spring)

7. registry
##### docker
```shell
tar -zxvf node_exporter-0.15.2.linux-amd64.tar.gz
sudo ./node_exporter
sudo docker run -d -p 9100:9100 --restart=always prom/node-exporter:latest
```
##### RHEL/CentOS
```shell
# curl -Lo /etc/yum.repos.d/_copr_ibotty-prometheus-exporters.repo https://copr.fedorainfracloud.org/coprs/ibotty/prometheus-exporters/repo/epel-7/ibotty-prometheus-exporters-epel-7.repo
# yum install node_exporter
```
[利用grafana性能监控](http://micrometer.io/docs/registry/prometheus#_installing)
