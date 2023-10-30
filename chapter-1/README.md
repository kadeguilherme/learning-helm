# Chapter 1

## Introduction Helm
Helm is a popular package manager and deployment tool for Kubernetes, designed to simplify the management of containerized applications within a Kubernetes cluster. It helps users define, install, and upgrade even the most complex Kubernetes applications with ease. Helm provides a way to package applications and their dependencies into a single unit called a "chart," which is a collection of files and templates describing a set of Kubernetes resources. These charts can then be shared and easily deployed to different Kubernetes clusters.

This chapter provides a conceptual overview of the cloud native ecosystem, in which Kubernetes is a key technology. We will also provide an overview of kubernetes, covering concepts such what is containers pods, services, deployments and what are goals the HELM and problems that his resolve.


## Kubernetes

## POD
The Kubernetes abstract, a container in the pod that is the smallest unit of Kubernetes.
Pods in Kubernetes typically contain just one container, but there are exceptions. Some pods include *"init containers"* that perform a pre-configuration task for the main container and then exit before the main container starts. Othet times are containers that run alongside the main container and provide auxialiary services. These are called *"sidecar containers"*


```YAML
apiVersion: v1 
kind: Pod # define the Kubernetes kind(v1 Pod) 
metadata:
    name: example-pod
spec:
    containers: # A pode can have on or more containers
    - image: "nginx:latest"
      name: example-nginx
```
***

## ConfigMaps and Secrets
A Pod describes what configuration the container or containers need (such as network ports or filesystem mount points).
Configuration data can be stored in ConfigMaps or Secrets for sensitive information.

The Pod definitions cam link these ConfigMaps and Secrets to environment variables or files within each container. Then Kubernetes will automatically attach and configure this configuration data according to the Pod definition.

```YAML
apiVersion: v1 # We have declared a v1 ConfigMap object
kind: ConfigMap
metadata:
    name: configuration-data
data: # Insede of data, we declare some arbitrary name/value pairs.
    backgroundColor: blue
    title: Learning Helm
```

>  📝 **_NOTE :_**   A *Secret* is structurally similar to a ConfigMap, except that the values in the data section must be Base64 encoded.


Pods are linked to configuration objects (like ConfigMap or Secret) using volumes. In this example, we take the previous Pod example and attach the Secret above:

```YAML
apiVersion: v1
kind: Pod
metadata:
    name: example-pod
spec:
    volumes: # The volumes section tells Kubernetes which storage sources this pods needs 
    - name: my-configuration
      configMap:
        name: configuration-data # The name configuration-data is the name of our ConfigMap 
    containers:
    - image: "nginx:latest"
      name: example-nginx
      env: # The env section injects environment variables into the container
        - name: BACKGROUND_COLOR # The environment variable will be named BACKGROUND_COLOR inside of the container.
          valueFrom:
            configMapKeyRef:
                name: configuration-data # This is the name of the ConfigMap it will use. This map must be in volumes if we want to use it as a filesystem volume.
                key: backgroundColor # This is the name of the key inside the data section of the ConfigMap.
```
***
# Deployment
Deployment in Kubernetes allows us to create our applications with a single pod, scale it up to five pods, reduce it back to three.
We can attach a *HorizontalPodAutoscaler* and configure that to scale our pod based on resource usage.
```YAML
apiVersion: apps/v1 # This is an apps/v1 Deployment object.
kind: Deployment
metadata:
    name: example-deployment
    labels:
        app: my-deployment
spec:
    replicas: 3 # Inside of the spec, we ask for three replicas of the following template.
    selector:
        matchLabels:
            app: my-deployment
    template: # The template specifies how each replica pod should look.
        metadata:
            labels:
                app: my-deployment
        spec:
            containers:
            - image: "nginx:latest"
              name: example-nginx
```
***
# Service
Kubernetes offers Service definitions to connect applications to the network. A Service is a persistent network resource, similar to a static IP, that remains available even if the associated pods are removed. This allows Kubernetes Pods to be dynamic, while the network ensures traffic is directed to the same Service endpoint.
```YAML
apiVersion: v1 
kind: Service
metadata:
  name: example-service
spec:
  selector:
    app: my-deployment # This Service will route to pods with the app: my-deployment label.
  ports:
    - protocol: TCP # TCP traffic to port 80 of this Service will be routed to port 8080 on the pods that match the app: my-deployment labe
      port: 80
      targetPort: 8080
```
***
# Helm’s Goals
When we wrote Helm, we had three main goals:
1. Make it easy to go from “zero to Kubernetes”
2. Provide a package management system like operating systems have
3. Emphasize security and configurability for deploying applications to Kubernetes

## 1 Make it easy to go from "zero to kubernetes"
The goal of Helm has been to simplify the process of getting started with Kubernetes, allowing new users to download and install applications on an existing Kubernetes cluster in some minutes.

Helm is not just a learning tool, it is a **package manager**. 

## 2 Package Management
The job of the package manager is to make it easy to find, install, upgrade, and delete the programs on an operating system.
