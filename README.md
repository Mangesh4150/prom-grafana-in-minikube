# prom-grafana-in-minikube
prom-grafana-in-minikube with cadvisor


1. minikube addons enable metrics-server

2. kubectl create namespace monitoring

3. create yaml file which include deployment , services and configmap if needed pv and pvc also (grafana_prometheus_minikube.yaml).

4. kubectl apply -f grafana_prometheus_minikube.yaml

5. For Verifications : 

minikube service list
minikube service grafana-service -n monitoring --url
minikube service prometheus-service -n monitoring --url

6. after create all resources in above file then , in broweser open grafana , promethesus and cadvisor.

7. In Promethesus , check available target in ---> status--->target and ensure there are UP.

8. In Grafana , ---> data Sources ---> Add New Data source ---> promethesus ---> Give name and Prometheus server URL --> Save and Test.

9. In grafana , ---> dashboard ---> import with ID (which type of dashboard we have required like kubernetes for grafana and so on) ---> select data source which we created earlier.

10. In Grafana , ---> Dashboard ---> select created dashboard ---> Monitor.


### command for create docker container for cadvisor (standalone docker container)
docker run -d \
  --name=cadvisor \
  --restart=always \
  -p 8080:8080 \
  --volume=/:/rootfs:ro \
  --volume=/var/run/docker.sock:/var/run/docker.sock:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/sys:/sys:ro \
  gcr.io/cadvisor/cadvisor:latest

