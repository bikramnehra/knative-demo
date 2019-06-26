# knative-demo

This repository contains resources for knative demo prepared for SUSE HackWeek18.

![alt text](https://knative.dev/docs/images/knative-audience.svg)


## Install

Install knative on minikube by following instructions [here](https://knative.dev/docs/install/knative-with-minikube/) or `install.txt`.

## Build

Create a `Secret` to store docker credentials:

```
kubectl create secret generic regcred \
    --from-file=.dockerconfigjson=/path/to/.docker/config.json \
    --type=kubernetes.io/dockerconfigjson
```

Create a `ServiceAccount` using the above `Secret`:

```
kubectl apply -f build/sa.yaml
```

Create a `BuildTemplate` by replacing git url(with your app) and docker repo in `build/build.yaml` and run:

```
kubectl apply -f build/build.yaml
```

## Serving

Deploy a service using the generated container image in the build step:

```
kubectl apply -f serve/service.yaml
```

Check the created revision by running:

```
kubectl get revisions
```

Find the IP/NodePort of the service by running:

```
INGRESSGATEWAY=istio-ingressgateway && \
IP_ADDRESS=$(minikube ip):$(kubectl get svc $INGRESSGATEWAY --namespace istio-system --output 'jsonpath={.spec.ports[?(@.port==80)].nodePort}')
```

To check if service is up and running:

```
curl -H "Host: knative-demo-app.default.example.com" ${IP_ADDRESS}
```

## Autoscaling

The default autoscaling in Knative `kpa.autoscaling.knative.dev` is based on concurrency. It can be tweaked and customized though, in this demo we used following annotations:

```
# Knative concurrency-based autoscaling (default).
autoscaling.knative.dev/class: kpa.autoscaling.knative.dev
autoscaling.knative.dev/metric: concurrency
# Target 10 requests in-flight per pod.
autoscaling.knative.dev/target: "10"
# Disable scale to zero with a minScale of 1.
autoscaling.knative.dev/minScale: "1"
# Limit scaling to max pods.
autoscaling.knative.dev/maxScale: "3"
```

For using the Standard Kubernetes CPU-based autoscaling:

```
autoscaling.knative.dev/class: hpa.autoscaling.knative.dev
autoscaling.knative.dev/metric: cpu 
```

For load testing:

```
hey -z 30s -c 50 \
  -host "knative-demo-app.default.example.com" \
  "{IP_ADDRESS}"
```

## Grafana

Use following command to port foward and access Grafana Dashboards:

```
kubectl port-forward --namespace knative-monitoring $(kubectl get pods --namespace knative-monitoring --selector=app=grafana  --output=jsonpath="{.items..metadata.name}") 3000
```

## Links:

- https://github.com/knative
- https://github.com/knative/docs/tree/master/docs/serving/samples/autoscale-go
- https://www.youtube.com/watch?v=OPSIPr-Cybs
- https://github.com/rakyll/hey
- https://medium.com/google-cloud/hands-on-knative-part-1-f2d5ce89944e
- https://medium.com/google-cloud/knative-buildpacks-source-code-to-container-image-without-dockerfile-34cb2dbfc49c
