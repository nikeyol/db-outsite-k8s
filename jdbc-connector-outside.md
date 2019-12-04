# JDBC Connector with DB outside of k8s clusterInfo

## Build the Docker image
### Dockerfile
Please make sure the jar file under subpath of dockerfile(./build directory)
```
# Alpine Linux with OpenJDK JRE
FROM openjdk:8-jre-alpine
# copy jar into image
COPY ./build/*.jar  /app.jar
# run application with this command line
CMD ["/usr/bin/java", "-jar", "/app.jar"]

### build Dockerfile
```
docker build -t jdbc-connector:0.1 .

## Prepare the k8s Configuration
### ConfigMap for JDBC connector pod
Note: refer to jdbc-connector-configmap.yaml and modify the property as need.
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-connector
  labels:
    app: db-connector
    env: prod
data:
  server.config: |
    authToken=m6vXhyC6DfBgF7TlRZsSjWOz-7wCSbITde1L-HCFh7E=
    sources=jdbc:mysql://outside-db:3306/db
    targetServer=https://dev.vantiq.cn/
```
### ServiceAccount for JDBC connector pod
Note: refer to jdbc-connector-serviceaccount.yaml
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jdbc-connector

### Service for JDBC connector pod
Note: map the outside db IP/host into k8s service, refer to jdbc-connector-service.yaml
```
kind: Service
apiVersion: v1
metadata:
  name: outside-db
spec:
  type: ExternalName
  externalName: a.b.com
```
 ### StatefulSet for JDBC Connector
 Note: refer to jdbc-connector-statefulset.yaml
```
kind: StatefulSet
metadata:
  name: jdbc-connector
spec:
  selector:
    matchLabels:
      app: jdbc-connector
      env: prod
  serviceName: jdbc-connector
  replicas: 1
  template:
    metadata:
      labels:
        app: jdbc-connector
        env: prod
    spec:
      serviceAccountName: jdbc-connector
      containers:
      - name: jdbc-connector
        image: jdbc-connector:0.1
        env:
        volumeMounts:
        - name: server.conf
          mountPath: /data/config/
        resources:
          requests:
            cpu: 500m
            memory: 1Gi
      volumes:
      - name: server.conf
        configMap:
          name: jdbc-connector
          
```
