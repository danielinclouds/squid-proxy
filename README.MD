
# Squid Proxy

## General information
- Squid configuration is in file `/etc/squid/squid.conf`
- Squid documentation http://www.squid-cache.org/Doc/config/
- Squid docker image is from [sameersbn/docker-squid](https://github.com/sameersbn/docker-squid)
- To configure allowed websites use `http_access` keyword

## Workshop
In this workshop we will deploy squid proxy and ubuntu client. Then we will show how to whitelist a website and how to configure proxy on the client.  


Create k8s cluster
```sh
kind create cluster
```

Deploy Squid and ubuntu client
```sh
kubectl apply -f squid/configmap_1.yaml
kubectl apply -f squid/squid.yaml
kubectl apply -f client/ubuntu.yaml
```

Install curl on client
```sh
kubectl exec -it ubuntu -- /bin/bash
apt-get update
apt-get install curl -y
```

Configure proxy for curl and try connecting to `httpbin.org`
```sh
export https_proxy=squid:3128
curl -I https://httpbin.org/status/200
# You should get 403 Forbidden
```

Whitelist `httpbin.org` on proxy
```sh
# New lines in squid configuration
# acl toblock dstdomain .httpbin.org
# http_access allow toblock
kubectl apply -f squid/configmap_2.yaml
kubectl rollout restart deployment squid
```

Try connecting to `httpbin.org` again
```sh
export https_proxy=squid:3128
curl -I https://httpbin.org/status/200
# Should work HTTP/2 200
curl -I https://www.google.com
# You should get 403 Forbidden
```

## Cleanup

```sh
kind delete cluster
```
