# Chapter 2

## Using Helm
Helm provides a command-line tool, named ```helm``` that makes available all the features necessary for working with Helm charts. The o chapter will discover the primary features of the ```helm``` client.

## Working with Kubernetes Clusters
Helm directly communicates with the Kubernetes API server. Then i Helm needs to be able to connect to a Kubernetes cluster. Helm automatically attempts to establish a connection by reading the same configuration files used by ```kubectl```

Helm will try to find this information by reading the enviroment variable ```$KUBECONFIG```. If that is not set, it will look in the same default locations that ```kubectl```  looks in (for example, $HOME/.kube/config on UNIX, Linux, and macOS)

You can also override these settings with environment variables __(HELM_KUBECONTEXT)__ and command-line flags __(--kube-context)__. You can see a list of environment variables and flags by running helm help.

## Getting Started with Helm
In the chapter 2, we will explore the most common workflow for getting started with Helm:

1. Add a chart repository.

2. Find a chart to install.

3. Install a Helm chart.

4. See the list of what is installed.

5. Upgrade your installation.

6. Delete the installation.

***

### 1 - Adding a Chart Repository

Helm chart is an **individual package** that can be installed into your Kubernetes cluster. During chart development, you will often just work with a chart that is stored on your local filesystem.

However, in the context of sharing charts, Helm defines a standardized format for indexing and sharing information about Helm charts. A Helm chart repository is essentially a collection of files accessible over the network, that conforms to the Helm specification for indexing packages.


