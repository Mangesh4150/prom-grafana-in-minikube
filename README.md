# prom-grafana-in-minikube
prom-grafana-in-minikube with cadvisor


1. #minikube addons enable metrics-server

2. #kubectl create namespace monitoring

3. create yaml file which include deployment , services and configmap if needed pv and pvc also (grafana_prometheus_minikube.yaml).

4. #kubectl apply -f grafana_prometheus_minikube.yaml

### 5. For Verifications : 

#minikube service list

#minikube service grafana-service -n monitoring --url

#minikube service prometheus-service -n monitoring --url


6. after create all resources in above file then , in broweser open grafana , promethesus and cadvisor.

7. In Promethesus , check available target in ---> status--->target and ensure there are UP.

8. In Grafana , ---> data Sources ---> Add New Data source ---> promethesus ---> Give name and Prometheus server URL --> Save and Test.

9. In grafana , ---> dashboard ---> import with ID (which type of dashboard we have required like kubernetes for grafana and so on) ---> select data source which we created earlier.

10. In Grafana , ---> Dashboard ---> select created dashboard ---> Monitor.


### Issues and Resolutions: cAdvisor, Prometheus, and Grafana Integration

### 1. cAdvisor Registration Failure

### Error:

I0206 08:18:23.646187       1 factory.go:219] Registration of the mesos container factory failed: unable to create mesos agent client: failed to get version
I0206 08:18:23.646347       1 factory.go:103] Registering Raw factory
I0206 08:18:23.646460       1 manager.go:1196] Started watching for new ooms in manager
W0206 08:18:23.646469       1 manager.go:306] Could not configure a source for OOM detection, disabling OOM events: open /dev/kmsg: no such file or directory

### Cause:

The error regarding Mesos can be ignored if Mesos is not in use.

The missing /dev/kmsg file indicates that the container does not have access to kernel logs.

### Solution:

This warning does not affect functionality but to resolve /dev/kmsg, ensure the container is running with --privileged mode.

If using Kubernetes, modify the cAdvisor deployment YAML:

securityContext:
  privileged: true

### 2. cAdvisor Service Not Reachable from Prometheus

### Error:

wget: unable to resolve host address 'cadvisor-service.monitoring.svc.cluster.local'

### Cause:

The Prometheus config may not have the correct service name for cAdvisor.

The cAdvisor service may not be running or exposed correctly.

### Solution:

Verify the cAdvisor service is running:

#kubectl get svc -n monitoring

Ensure the correct name is in prometheus.yml:

scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor-service.monitoring.svc.cluster.local:8083']

If the service name is incorrect, update the ConfigMap and restart Prometheus.

# 3. Command Execution Failure in Prometheus Pod

# Error:

OCI runtime exec failed: exec failed: unable to start container process: exec: "curl": executable file not found in $PATH: unknown
command terminated with exit code 126

# Cause:

The Prometheus container does not have curl installed.

# Solution:

Use wget instead of curl:

#kubectl exec -n monitoring -it <prometheus-pod> -- wget -O- cadvisor-service.monitoring.svc.cluster.local:8083

# 4. Grafana Not Displaying Metrics

# Error:

No data appears in Grafana after adding Prometheus as a data source.

# Cause:

Prometheus is not scraping cAdvisor metrics correctly.

Incorrect Prometheus service URL in Grafana.

# Solution:

Check if Prometheus is scraping metrics:

#kubectl exec -n monitoring -it <prometheus-pod> -- wget -O- cadvisor-service.monitoring.svc.cluster.local:8083

Ensure Prometheus service is reachable from Grafana:

#kubectl get svc -n monitoring

Update Grafana’s data source with:

http://prometheus-service.monitoring.svc.cluster.local:9090

Restart Prometheus and Grafana if necessary.

# Final Resolution:

## After fixing the above issues, the integration between cAdvisor, Prometheus, and Grafana is now working successfully.


### command for create docker container for cadvisor (standalone docker container)
#docker run -d \
  --name=cadvisor \
  --restart=always \
  -p 8080:8080 \
  --volume=/:/rootfs:ro \
  --volume=/var/run/docker.sock:/var/run/docker.sock:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/sys:/sys:ro \
  gcr.io/cadvisor/cadvisor:latest

