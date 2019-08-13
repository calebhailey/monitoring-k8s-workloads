# Monitoring Kubernetes: The sidecar pattern

This repository contains content from my August 8, 2019 webinar for the [Cloud
Native Computing Foundation][cncf], entitled "Monitoring Kubernetes: The sidecar
pattern".

## Prerequisites

1. A Kubernetes cluster

## Overview

1. **Deploy Sensu.**

   _Coming soon..._

2. **Deploy an example application as a Kubernetes deployment (NGINX website).**

   The NGINX deployment in `kubernetes/nginx-deployment.yaml` is a very simple
   example K8s service + deployment. Let's deploy it using `kubectl apply`:

   ```
   $ kubectl create namespace webinar
   $ kubectl --namespace webinar apply -f kubernetes/nginx-deployment.yaml
   $ kubectl --namespace webinar get services
   NAME    TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
   nginx   LoadBalancer   10.27.251.183   35.230.122.31   8000:30161/TCP   33d
   ```

   Visit your example application in your browser by visiting the "EXTERNAL-IP"
   address. For example, the Kubernetes service shown above is accessible at
   http://35.230.122.31:8000

   ![](/files/example-nginx-screenshot.png)

3. **Add the Sensu Go agent as a sidecar for your example application.**

   The NGINX deployment in `kubernetes/nginx-deployment-with-sensu-sidecar.yaml`
   shows how easy it is to add the Sensu Go agent as a sidecar. Let's updated
   our deployment using `kubectl apply`:

   ```shell
   $ kubectl --namespace webinar apply -f kubernetes/nginx-deployment-with-sensu-sidecar.yaml
   ```

   If you visit your Sensu dashboard in your browser, you will see that our
   sidecars have already registered themselves with Sensu.

   ![](/files/sensu-dashboard.png)

   How does it work? Here's the relevant excerpt from the `spec.template.spec.containers`
   scope in the Kubernetes deployment manifest:

   ```yaml
   - name: sensu-agent
     image: sensu/sensu:5.11.1
     command: ["/opt/sensu/bin/sensu-agent", "start", "--log-level=debug", "--insecure-skip-tls-verify", "--keepalive-interval=5", "--keepalive-timeout=10"]
     env:
     - name: SENSU_BACKEND_URL
       value: wss://sensu-backend-0.sensu.sensu-system.svc.cluster.local:8081 wss://sensu-backend-1.sensu.sensu-system.svc.cluster.local:8081 wss://sensu-backend-2.sensu.sensu-system.svc.cluster.local:8081
     - name: SENSU_NAMESPACE
       value: webinar
     - name: SENSU_SUBSCRIPTIONS
       value: linux kubernetes nginx
     - name: SENSU_DEREGISTER
       value: "true"
     - name: SENSU_STATSD_EVENT_HANDLERS
       value: statsd
   ```
