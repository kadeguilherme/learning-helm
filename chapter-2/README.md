# Chapter 2 - Using Helm

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
The helm list command is a simple tool to help you see installations and learn about those installations:
 
 ```bash
$ helm list
```
Output:
```YAML
NAME    NAMESPACE  REVISION  UPDATED       STATUS    CHART           APP VERSION
mysite  default    1         2020-06-14... deployed  drupal-7.0.0    9.0.0
 ```

This command will provide you with lots of usefull information, including the name and namespace of the release, the current revision number, the last time it was updated, the installation status and the versions of the chart and app. 

By defaulft, Helm uses the namespace your Kubernetrs configuration file sets as the default. Usually this is the namespace named **default**. Earlier, we installed a Drupal instance into the namespace **first**. We can see that with helm list **--namespace first**

When listing all of your releases, one useful flag is the **--all-namespaces** flag, which will query all of the Kubernetes namespaces to which you have permission, and return all of the releases it finds

### 5 - Upgrading an Installation

When we talk about upgrading in Helm, we talk about upgrading an installation, not a chart.
An installation is a particular instance of a chart in your cluster. When you run **helm install**, it creates the installation. To modify that installation, use **helm upgrade**

This is an important distinction to make in the present context because upgrading an installation can consist of two different kinds of changes:

- You can upgrade the version of the chart

- You can upgrade the configuration of the installation

The two are not mutually exclusive; you can do both at the same time. But this does introduce one new term that Helm users refer to when talking about their systems: a release is a particular combination of configuration and chart version for an installation

When first install a chart, we create the initial release of an installation. Will call this release 1. When we perform an upgrade, we are creating a *new release* of the same installation: release2, doing upgrade again, we will create release 3.
During an upgrade, then, we can create a release with new configuration, with a new chart version, or with both.

For example, we install the Drupal chart with the **ingress** turned off.

The flag **--set** to keep examples easy,but would recommend using a values.yaml file in regular scenarios:

```bash
$ helm install mysite bitnami/drupal --set ingress.enabled=false
```

With **ingress** turned off, we can work on configuring our site according to our preferences. Then we are ready for create a new release that enables the **ingress** feature:

```bash
$ helm upgrade mysite bitnami/drupal --set ingress.enabled=true
```

In this case, we are running an upgrade that will only change the configuration.

In the background, Helm will load the chart, generate all of the Kubernetes objects in that chart, and then see how those differ from the version of the chart that is already installed. It will only send Kubernetes the things that need to change. In other words, Helm will attempt to alter only the minimum.

It is worth noting that we only changes the **ingress** configuration. Nothing changes with the database, or even with the web server running Drupal. For that reason, nothing will be restarded or deleted and recreated.



On occasion, you may want to force one of your services to restart. This is not something you need to use Helm. You can use **kubectl** itself to restart things. You don’t need to use Helm to restart your web server or database

When a new version of a chart is release, you may want to upgrade your existing installation to use the new chart version. The Helm make this easy:

```bash
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈ Happy Helming!⎈
```
Fetch the latest packages from chart repositories.

```bash
$ helm upgrade mysite bitnami/drupal
```
Upgrade the mysite release to use the latest version of bitnami/drupal.

The default policy of Helm is to attempt to use the latest version of a chart. However, if you wish to use a specific version of a chart, you can explicitly declare this.

```bash
$ helm upgrade mysite bitnami/drupal --version 6.2.22
```
In this case, even if a newer version is released, only bitnaim/drupal version 6.2.22 will be installed.


### Configuration Values and Upgrades

Understand the importance of Helm installations and upgrades by utilizing the configuration file *values.yaml*. Here’s a quick illustration:

```bash
$ helm install mysite bitnami/drupal --values values.yaml
```
Install using a configuration file.

```bash
$ helm upgrade mysite bitnami/drupal
```
Upgrade without a configuration file.

What is the result of this pair of operations? The installation will use all of the configuration data supplied in values.yaml, but the upgrade will not. As a result, some settings could be changed back to their defaults. This is usually not what you want.

Helm core maintainers suggest that you provide consistent configuration with each installation and upgrade. To apply the same configuration to both releases, supply the values on each operation:

```bash
$ helm install mysite bitnami/drupal --values values.yaml
```
Install using a configuration file.

```bash
$ helm upgrade mysite bitnami/drupal --values values.yaml 
```
Upgrade using the same configuration file.

Storing configuration in a values.yaml file is recommended for ease of reproduction. Using **--set** for multiple configuration parameters would be cumbersome, requiring precise recall of settings for each release.


There is an upgrade shortcut available that will just reuse the last set of values that you sent:

```bash
$ helm upgrade mysite bitnami/drupal --reuse-values
```

The **--reuse-values** flag will tell Helm to reload the server-side copy of the last set of values, and then use those to generate the upgrade. This method is okay if you are always just reusing the same values.However, the Helm maintainers strongly suggest not trying to mix **--reuse-values** with additional **--set** or **--values options**. 


### 6 - Uninstalling an Installation
To remove a Helm installation, use the helm uninstall command:

```bash
$ helm uninstall mysite
```
Note that this command does not need a chart name (bitnami/drupal) or any configuration files. It simply needs the name of the installation.

You can supply a --namespace flag to specify that you want to delete an installation from a specific namespace:

```bash
$ helm uninstall mysite --namespace first
```

### How Helm Stores Release Information
When we first install a chart with Helm (such as with **helm install mysite bitnami/drupal**), we create the Drupal application instance, and we also create a special record that contains release information. By default, Helm stores these records as **Kubernetes Secrets** (though there are other supported storage backends).

We can see these records with kubectl get secret:

```bash
$ kubectl get secret
NAME                           TYPE                                  DATA   AGE
default-token-vjhx2            kubernetes.io/service-account-token   3      58m
mysite-drupal                  Opaque                                1      13m
mysite-mariadb                 Opaque                                2      13m
sh.helm.release.v1.mysite.v1   helm.sh/release.v1                    1      13m
sh.helm.release.v1.mysite.v2   helm.sh/release.v1                    1      13m
sh.helm.release.v1.mysite.v3   helm.sh/release.v1                    1      7m53s
sh.helm.release.v1.mysite.v4   helm.sh/release.v1                    1      5m30s
```
We can see multiple release records at the bottom, one for each revision. As you can see, we have created four revisions of mysite by running install and upgrade operations.

When we run the command **helm uninstall mysite**, it will load the latest release record for the **mysite** installation. From that record, it will assemble a list of objects that it should remove from Kubernetes. Then Helm will delete all of those things before returning and deleting the four release records:

```bash
$ helm uninstall mysite
release "mysite" uninstalled
```

The helm list command will no is show mysite:

```bash
$ helm list
NAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION

```

We now have no installations. And if we rerun the **kubectl get secrets** command, we will also see all records of **mysite** have been purged:

```
$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-vjhx2   kubernetes.io/service-account-token   3      65m
```

As we can see from this output, it's evident that not only were the two Secrets created by the Drupal chart deleted, but also the four release records were removed.

However, you can delete the application while retaining the release records.

```bash
$ helm uninstall --keep-history
```
In Helm 2, history was retained by default. In Helm 3, the default was changed to deleting history.