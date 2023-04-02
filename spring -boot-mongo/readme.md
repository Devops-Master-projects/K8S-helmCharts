#### **HELM**

Helm is a package manager for Kubernetes

*commands:*

kubens

install helm

```
helm create 
```

Define the chart templates
Define the chart values:  Once you've defined your chart,

`helm show chart demochart2`


`helmfile sync = check diff and auto applies`

```
helmfile --f 
```

```
helmfile --interactive apply --wait
```


To use helm chart from a repository, add the repo url in the helmfile 

```
repositories:
  - name: prometheus 
    url: https://prometheus-community.github.io/helm-charts
```





> ### **INSTALLATION AND CONFIGURATION
> **

```
helm Install 
```

Helmfile:

## **Install helmfile**

https://helmfile.readthedocs.io/en/latest/

## **Install helmfile diff plugin**

helm plugin install https://github.com/databus23/helm-diff

## Install helm git plugin

Verify that the plugin has been installed by running the following command:

`linux: helm plugin install https://github.com/aslafy-z/helm-git.git`

```
helm plugin list
```
