# Prometheus JMX Exporter container for Kubernetes

This is a Docker container intended to be run in the same pod as your Java containers, to export their metrics for Prometheus.

The default `CMD` copies over the required [JMX Exporter](https://github.com/prometheus/jmx_exporter) files to the directory specified by the `SHARED_VOLUME_PATH` environment variable.

## Files available

* `$SHARED_VOLUME_PATH/jmx_prometheus_javaagent.jar`: the [JMX Exporter](https://github.com/prometheus/jmx_exporter) javaagent JAR file.
* `$SHARED_VOLUME_PATH/config/*.yaml`: example config files for the exporter. You can pick some [here](https://github.com/prometheus/jmx_exporter/blob/master/example_configs/)

## Using this as a Kubernetes Init Container

This container is best used as an [Init Container](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/).

Add this to `initContainers` of your `Deployment` or `StatefulSet`:

```yaml
spec:
  initContainers:
  - name: prometheus-jmx-exporter
    image: matsouch/prometheus-jmx-exporter-kubernetes:1.0.1
    env:
    - name: SHARED_VOLUME_PATH
      value: /shared-volume
    volumeMounts:
    - mountPath: /shared-volume
      name: shared-volume
```

The init container and your application container will share a volume:

```yaml
volumes:
- name: shared-volume
  emptyDir: {}
```

In your application container, set your Docker application run image params to refer to files placed by the `prometheus-jmx-exporter` container into the shared volume.

Example with a Dockerfile and the camel configuration available in the repository :

```Dockerfile
ENTRYPOINT java -javaagent:/shared-volume/jmx_prometheus_javaagent.jar=9090:/shared-volume/configs/camel-config.yml -jar /app.jar
```

Don't forget to annotate your resources so Prometheus will scrape your pod's `/metrics` endpoint:

```yaml
annotations:
  "prometheus.io/scrape": "true"
  "prometheus.io/port": "9090"
```
