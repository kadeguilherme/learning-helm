# Chapter 1 - Introduction Helm

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
***

## 2 Package Management
The job of the package manager is to make it easy to find, install, upgrade, and delete the programs on an operating system.
***
## 3 Security, Reusability and Configurability
 Security is a broad category. In this context, though, we are referring to the idea that when a user examines a package, the user has the ability to verify certain things about the package:

- The package comes from a trusted source.

- The network connection over which the package is pulled is secured.

- The package has not been tampered with.

- The package can be easily inspected so the user can see what it does.

- The user can see what configuration the package has, and see how different inputs impact the output of a package.

Helm provides a provenance feature to establish verification about a package’s origin, author, and integrity. Helm supports Secure Sockets Layer/Transport Layer Security (SSL/TLS) for securely sending data across the network. And Helm provides dry-run, template, and linting commands to examine packages and their possible permutations.
***
Reusability
Helm charts are the key to reusability. A chart provides a template for generating the same Kubernetes manifests.
***
Configurability
Helm provides patterns for talking a Helm chart and then supplying additional configuration. For example, install a WordPress with Helm.
Helm is a package manager, not a configuration management tool, Configuration management are Puppet, Ansible and Chef.
***

# Helm's Architecture

Charts
In Helm's terminology, a package is referred to as a chart.<br>
A chart is a set of files and directories that adhere to the chart specification for describing the resources to be installed into Kubernetes.<br>
A chart contains a file called Chart.yaml that describes the chart. It has information about the chart version, the name and description of the chart, and who authored the chart.<br>
A chart may also contain a values.yaml file that provides default configuration. This file contains parameters that you can override during installation and upgrade.

# Resources, Installations, and Releases
In summary, when Helm chart is installed in Kubernetes, this is the process that takes place

1. Helm reads the chart(downloading if necessary).
2. It sends the values into the templates, generating Kubernetes manifests. 
3. The manifests are sent to Kubernetes.
4. Kubernetes creates the requested resources inside of the cluster.