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
## References
- [Deploy Apache Web Server on Kubernetes CLuster](https://shubhamksawant.medium.com/deploy-apache-web-server-on-kubernetes-cluster-0552638ca171) by [Shubham K. Sawant](https://shubhamksawant.medium.com/)
- [An https echo Docker container for web debugging](https://code.mendhak.com/docker-http-https-echo/) by [https://code.mendhak.com/](https://code.mendhak.com/)

# k8s-debugging-tools
