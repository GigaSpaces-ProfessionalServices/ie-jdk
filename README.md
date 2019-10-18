# ie-jdk

This repository contains a sample Dockerfile that will allow you to use JDK 11 with InsightEdge.

Prepare your docker image.

1. `docker login --username=<your username>`
2. `docker build -t <username>/ie-jdk11:15.0.0-m15 .`
3. `docker push <username>/ie-jdk11:15.0.0-m15`

Add the early access repository:
1. `helm repo add gigaspaces-ea https://resources.gigaspaces.com/helm-charts-ea`
2. `helm update repo`


Sample deployment command:

`helm install gigaspaces-ea/insightedge --version=15.0.0-m15 --name demo --set manager.image.repository=<username>/ie-jdk11,manager.image.tag=15.0.0-m15,pu.image.repository=<username>/ie-jdk11,pu.image.tag=15.0.0-m15,zeppelin.image.repository=<username>/ie-jdk11,zeppelin.image.tag=15.0.0-m15`


