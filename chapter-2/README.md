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

> **NOTE:** Helm 3 introduced an experimental feature for storing Helms charts in a different kind of repository: Open Container Initiative (OCI) registries (sometimes called Docker registries)

There are many, perhaps thousands of, chart repositories on the internet. The easiest way to find the popular repositories is to use your web browser to navigate to the Artifact Hub.

Adding a Helm chart is done with the **`helm repo add`**.  The helm repo add command will add a repository.

Several Helm repository commands are grouped under the `**helm repo**`

```bash
helm repo add [name] [URL]
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Now we can verify that the Bitnami repository exists by running a second repo command:

```bash
helm repo list
```
This command shows us all of the repositories installed for Helm. Right now, we see only the Bitnami repository that we just added.
***

### 2 - Searching a Chart Repository
Search repositories for a keyword in charts

```bash
helm search repo [keyword]
```
***
### 3 - Installing a Package
When executing the helm install command, an installation name is required in addition to the chart name. Thus, the fundamental installation command typically resembles the following:
```bash
helm install [name] [name_chart]
helm install mysite bitnami/drupal
```
The preceding will create an instance of the **bitnami/drupal** chart, and will name this instance mysite.

There is now an instance named *mysite* in the cluster. If on attempts to rerun the previous command, a second instance won't be created. Instead, we would get an error because the name mysite has already been used:

```bash
$ helm install mysite bitnami/drupal
Error: cannot re-use a name that is still in use
```

In Helm 3, instance name are now scoped to kubernetes namespaces. This means that we could install two instances named *mysite* as long as they each lived in a different namespace.

To resolve the above we need to create another namespace in kubernetes

```bash
$ kubectl create ns first
$ kubectl create ns second
$ helm install --namespace first mysite bitnami/drupal
$ helm install --namespace second mysite bitnami/drupal
```
This will install one Drupal site named mysite in the first namespace, and an identically configured instance named mysite in the second namespace.

#### Configuration at Installation Time
Many charts ofter the flexibillity to customize configuration values. If we take a look at the Artifact Hub page for Drupal, there is an extensive list of configurable parameters. For example, we can configure the username of the Drupal admin account by setting the **drupalUsername** value.

There are severa; ways of telling Helm which values you want to be configured. The recommended approach involves creating a YAML file containing all the necessary configuration overrides. For example, we can create a file taht sets values **fordrupalUsername** and **drupalEmail**


```bash
drupalUsername: admin
drupalEmail: admin@example.com
```


Best practices to have a file (conventionally named values.yaml) that has all of our configuration. This file can be committed to a version control system for tracking changes over time. The Helm core maintainers consider it a good practice to keep your configuration values in a YAML file. However, caution is advised when the configuration file contains sensitive information (like a password or authentication token) to prevent inadvertent leaks.

Both **helm install** and **helm upgrade** provide a **--values** flag that points to a YAML file with value overrides:


```bash
helm install mysite bitnami/drupal --values values.yaml
```
Output

```YAML
NAME: mysite
LAST DEPLOYED: Sun Jun 14 14:56:15 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
*******************************************************************
*** PLEASE BE PATIENT: Drupal may take a few minutes to install ***
*******************************************************************

1. Get the Drupal URL:

  You should be able to access your new Drupal installation through

  http://drupal.local/

2. Login with the following credentials

  echo Username: admin
  echo Password: $(kubectl get secret --namespace default mysite-drupal -o js...
```


There is a second flag that can be used to add individual parameters to an install or upgrade. The **--set** flag takes one or more values directly. They do not need to be stored in a YAML file:

```bash
 helm install mysite bitnami/drupal --set drupalUsername=admin
```
This sets just one parameter, drupalUsername. This flag uses a simple key=value format.

Configuration parameters can be organized into sections within a configuration file. In the case of the Drupal chart, there is a specific section for **mariadb** database configuration, where relevant parameters are grouped. Building on our previous example, we could override the MariaDB database name like this:
```YAML
drupalUsername: admin
drupalEmail: admin@example.com
mariadb:
  db:
    name: "my-database"
```

Subsections are a little more complicated when using the --set flag. You will need to use a dotted notation: **--set mariadb.db.name=my-database**. This can get verbose when setting multiple values.

Helm core maintainers recommend storing configurations primarily in files like values.yaml, discouraging excessive use of the --set option.
***
### 4 - Listing Your Installations