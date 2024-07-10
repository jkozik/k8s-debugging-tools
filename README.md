# Troubleshooting Tools
After a couple of troubleshooting challenges with my kubernetes cluster, I found a few useful  tools.

## http/https-echo
I borrowed yaml from [Shubham K. Sawant](https://shubhamksawant.medium.com/) and deployed a rather generic web server on my cluster.  I substituted an echo server from  [https://code.mendhak.com/](https://code.mendhak.com/) for the apache image.  See web.yaml
```
jkozik@knode202:~/httpd$ cat web.yaml
apiVersion: v1
kind: Service
metadata:
  name: httpd
spec:
  type: NodePort
  selector:
    app: httpd_app
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30004
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deployment-devops
  labels:
    app: httpd_app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: httpd_app
  template:
    metadata:
      labels:
        app: httpd_app
    spec:
      containers:
        - name: httpd-container-devops
          #image: httpd:latest
          #image: php:7.2-apache
          image:  mendhak/http-https-echo
          ports:
            - containerPort: 8080
jkozik@knode202:~/httpd$
```
## Ingress k8s.kozik.net
I want to be able to get at my httpd-echo server from the internet, so I setup an ingress.  I have a normal NGINX ingress controller already installed. I map it to this server using the following yaml:
```
jkozik@knode202:~/httpd$ cat web-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    #kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /
  name: httpd-ingress
  namespace: default
spec:
  ingressClassName: nginx
  rules:
  - host: k8s.kozik.net
    http:
      paths:
      - backend:
          service:
            name: httpd
            port:
              number: 80
        path: /
        pathType: Prefix
jkozik@knode202:~/httpd$

```
I have an apache httpd reverse proxy setup that maps the external URL to route to the port number for this service.

## Verify httpd-echo service/ingress
```
jkozik@knode202:~/httpd$ kubectl get svc httpd
NAME    TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
httpd   NodePort   10.101.175.143   <none>        80:30004/TCP   6d18h

jkozik@knode202:~/httpd$ kubectl get ing httpd-ingress
NAME            CLASS   HOSTS           ADDRESS           PORTS   AGE
httpd-ingress   nginx   k8s.kozik.net   192.168.100.200   80      6d18h

jkozik@knode202:~/httpd$ curl -H "Host: k8s.kozik.net" http://192.168.100.202:30004
{
  "path": "/",
  "headers": {
    "host": "k8s.kozik.net",
    "user-agent": "curl/7.81.0",
    "accept": "*/*"
  },
  "method": "GET",
  "body": "",
  "fresh": false,
  "hostname": "k8s.kozik.net",
  "ip": "::ffff:192.168.100.202",
  "ips": [],
  "protocol": "http",
  "query": {},
  "subdomains": [
    "k8s"
  ],
  "xhr": false,
  "os": {
    "hostname": "httpd-deployment-devops-59ccbdf487-qhrlr"
  },
  "connection": {}
}jkozik@knode202:~/httpd$
```

## References
- [Deploy Apache Web Server on Kubernetes CLuster](https://shubhamksawant.medium.com/deploy-apache-web-server-on-kubernetes-cluster-0552638ca171) by [Shubham K. Sawant](https://shubhamksawant.medium.com/)
- [An https echo Docker container for web debugging](https://code.mendhak.com/docker-http-https-echo/) by [https://code.mendhak.com/](https://code.mendhak.com/)

# k8s-debugging-tools
