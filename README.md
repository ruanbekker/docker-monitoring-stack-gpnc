# docker-monitoring-stack-gpnc
Grafana Prometheus Node-Exporter cAdvisor Loki - Docker Monitoring Stack

## About

Get your monitoring stack up and running with one command using a Docker Compose stack featuring:

- **Grafana**: Dashboarding.
- **Prometheus**: Timeseries database for metrics.
- **Node-Exporter**: Node metrics.
- **cAdvisor**: Container metrics.
- **Alertmanager**: Alerting system.
- **Loki**: Logs (including explore-logs).

## Makefile

[Note](https://docs.docker.com/compose/install/linux/): Due to `docker-compose` and the `compose` plugin, you might have one of the two installed. I have a `Makefile` that will detect which on you have installed.

You can list the targets using `make`.

## Deployment

You can use one of the following deployment methods:

- **default**: deploys the containers mentioned above.
- **remote-write**: adds a second prometheus and enables remote write with prometheus and loki.

### Boot: Default

Boot the stack with docker compose (or `make up`), which is the default deployment:

```bash
docker-compose up -d
```

Ensure all containers are running:

```bash
docker-compose ps
```

The output should looke like this:

```bash
    Name                   Command                  State               Ports         
--------------------------------------------------------------------------------------
cadvisor        /usr/bin/cadvisor -logtostderr   Up (healthy)   8080/tcp              
grafana         /run.sh                          Up             0.0.0.0:3000->3000/tcp
node-exporter   /bin/node_exporter --path. ...   Up             9100/tcp              
prometheus      /bin/prometheus --config.f ...   Up             0.0.0.0:9090->9090/tcp
alertmanager    /bin/alertmanager --config ...   Up             0.0.0.0:9093->9093/tcp
loki            /usr/bin/loki -conf ...          Up             0.0.0.0:3100->3100/tcp
promtail        /usr/bin/promtail ...            Up
```

### Boot: Remote Write

To deploy the remote write deployment, you can run:

```bash
make up-remote
```

This will deploy the default stack, as well as the remote prometheus container and remote-write configuration, to do the following:
- **prometheus**: push metrics from `cluster: local` to `cluster: remote` using remote-write.
- **loki**: push recording rules metrics to `cluster: remote` prometheus.

Ensure all containers are running:

```bash
docker-compose ps
```

The output should looke like this:

```bash
    Name                   Command                  State               Ports
--------------------------------------------------------------------------------------
cadvisor          /usr/bin/cadvisor -logtostderr    Up (healthy)   8080/tcp
grafana           /run.sh                           Up             0.0.0.0:3000->3000/tcp
node-exporter     /bin/node_exporter --path. ...    Up             9100/tcp
prometheus        /bin/prometheus --config.f ...    Up             0.0.0.0:9090->9090/tcp
prometheus-remote /bin/prometheus --config.f ...    Up             0.0.0.0:9091->9090/tcp
alertmanager      /bin/alertmanager --config ...    Up             0.0.0.0:9093->9093/tcp
loki              /usr/bin/loki -conf ...           Up             0.0.0.0:3100->3100/tcp
promtail          /usr/bin/promtail ...             Up
```

## Access Grafana

Access grafana on [Grafana Home](http://localhost:3000/?orgId=1) (or `make open`) and you should see the three dashboards that was provisioned:

![](./assets/grafana-home.png)

Once you select the **Node Metrics** dashboard, it should look something like this:

![](./assets/grafana-dashboard.png)

When you select ["Alerting" and "Alert rules"](http://localhost:3000/alerting/list) you will find the recording and alerting rules:

![](./assets/grafana-alerting-home.png)

We can expand the alerting rules:

![](./assets/grafana-alerting-rules.png)

And then we can view more detail on a alert rule:

![](./assets/grafana-alerting-detail.png)

And for our container metrics we can access the **Container Metrics** dashboard:

![](./assets/grafana-container-metrics.png)

Then for our last dashboard, the **Container Log Search**, by default the metric panel will be collapsed, but to expand it for visibility it will look like this:

![](./assets/grafana-logs-search-dashboard.png)

And we can also view our **Container Logs** in the explore section:

![](./assets/grafana-logs-view.png)

For discovering the **Logs** we can navigate to the Explore / Logs view:

![](./assets/grafana-explore-logs.png)

## Remote Write

### Remote Write: Prometheus

The prometheus cluster is configured to push its metrics to our `prometheus-remote` container, and we can access our remote prometheus on port 9091 using the following query:
- [`prometheus_build_info{cluster="local"}`](http://localhost:9091/graph?g0.expr=prometheus_build_info%7Bcluster%3D%22local%22%7D&g0.tab=1&g0.stacked=0&g0.show_exemplars=0&g0.range_input=1h)

As you can see the external label `cluster: local` is being applied on our `prometheus` container.

The config is accessible: `./configs/prometheus/prometheus-remotewrite.yml`

### Remote Write: Loki

The remote write configuration under the Loki Ruler has been configured to write the recording rules to our `prometheus-remote` container endpoint, and can be access using:
- [`instance:service_log_bytes:sum_rate1m`](http://localhost:9091/graph?g0.expr=instance%3Aservice_log_bytes%3Asum_rate1m&g0.tab=1&g0.stacked=0&g0.show_exemplars=0&g0.range_input=1h)

The config is accessible: `./configs/loki/loki-remotewrite.yaml`

## Endpoints

The following endpoints are available:

| Container         | Internal Endpoint             | External Endpoint     | Note                               |
| ----------------- | ----------------------------- |---------------------- | ---------------------------------- |
| Grafana           | http://grafana:3000           | http://localhost:3000 | Enabled in default deployment      |
| Prometheus        | http://prometheus:9090        | http://localhost:9090 | Enabled in default deployment      |
| Prometheus-Remote | http://prometheus-remote:9090 | http://localhost:9091 | Enabled in remote-write deployment |
| Node-Exporter     | http://node-exporter:9100     | http://localhost:9100 | Enabled in default deployment      |
| cAdvisor          | http://cadvisor:8080          | N/A                   | Enabled in default deployment      |
| Alertmanager      | http://alertmanager:9093      | http://localhost:9093 | Enabled in default deployment      |
| Loki              | http://loki:3100              | http://localhost:3100 | Enabled in default deployment      |

## Cleanup

To remove the containers: 

```bash
make clean
```

## Stargazers over time

[![Stargazers over time](https://starchart.cc/ruanbekker/docker-monitoring-stack-gpnc.svg)](https://starchart.cc/ruanbekker/docker-monitoring-stack-gpnc)

## Resources

Heavily inspired from [this exporter guide](https://grafana.com/oss/prometheus/exporters/node-exporter/)
