# Using k3d with Codezero

In order to use a local kubenetes cluster such as [k3d](https://k3d.io) with [codezero](https://codezero.io) 
you need an external local balancer so that other members of your team route traffic to and from your cluster.

To do that we make use of the [inlets-controller](https://github.com/inlets/inlets-operator) from [inlets.dev](https://inlets.dev)

## Install a local kubenetes cluster

Using k3d v5.6.0 from https://k3d.io/v5.6.0/

```
brew install k3d
```

Create a cluster without a load-balancer

```
k3d cluster create czdemo --api-port 6550 --agents 1 \
  --k3s-arg "--disable=traefik@server:0" \
  --k3s-arg "--disable=servicelb@server:0" \
  --no-lb \
  --wait
```

Make sure you can connect to it

```
kubectl cluster-info
```

## Prep

Get the inlets pro license from their [site](https://inlets.dev/pricing)

For this demo we'll be using Digital Ocean.  Get a personal access token from your [control panel](https://cloud.digitalocean.com/account/api/tokens).

## Install 

Create the namespace and install the license and inlets-access-key secrets

```
kubectl create namespace inlets
```

```
kubectl create secret generic -n inlets \
  inlets-license --from-literal license=<inlets license key>
```

```
kubectl create secret generic -n inlets \
inlets-access-key \
--from-literal inlets-access-key=<digital oceano personal access token>
```

Install the inlets-operator using helm

```
helm repo add inlets https://inlets.github.io/inlets-operator/
helm repo update && \
  helm upgrade inlets-operator --install inlets/inlets-operator \
  --namespace inlets \
  --set region=lon1
```

## Test

Install nginx

```
kubectl apply -f nginx.yaml
```

Get the service IP address

```
kubectl get svc -n demo1
```

You should see the Welcome to nginx! page.

## Codezero

Follow the codezero getting started guide: https://docs.codezero.io/guides/getting-started

Consume the nginx service

```
czctl compose start -f cz-nginx.yaml
```

Test

```
curl http://nginx/
```

Will return the Welcome to nginx! page



