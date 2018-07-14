# Openshift
### Kubernetes with a human face

Vadim Rutkovsky

vrutkovs@redhat.com

---
![Cat pic](imgs/cat.png)

Note:
* Enterprise Kubernetes, developer-focused
* Forked from Kubernetes codebase, regularly synced with upstream (3 month delay)
* Not an overlay over k8s and not an additional layer of abstraction
* Additional k8s objects - ImageStream, BuildConfigs, DeploymentConfigs
* Origin - upstream version, Openshift Container Platform - enterprise version
* OCP versions are supported for 2 years

---
### What's in the box?
![openshift](imgs/openshift.png)

---
### Openshift - batteries included
* OAuth server for authentication
* Docker registry
* HAProxy router
* CI/CD out-of-the-box via Jenkins Pipelines
* Source2Image

![Professor Fortran](imgs/fortran.png)

---
### Getting Started

`oc` - openshift's `kubectl`

Setting up the local containerized cluster:
```shell
$ sudo oc cluster up
Starting OpenShift using openshift/origin:v3.9.0 ...
OpenShift server started.

The server is accessible via web console at:
    https://127.0.0.1:8443

You are logged in as:
    User:     developer
    Password: <any value>

To login as administrator:
    oc login -u system:admin
```

---
Install using Ansible

https://docs.openshift.org/latest/install_config/

Openshift Online

https://manage.openshift.com/

Openshift Dedicated

https://www.openshift.com/dedicated/

Cloud IDE

https://openshift.io

---
# Demo
### Source 2 Image
### Builds, DeploymentConfigs
### Routes

Note:
Cluster: https://cloud.vrutkovs.eu:8443/console/
S2I demo: master branch

```
oc login ...
oc new-project itgm
oc new-app --name=itgm-demo http://github.com/vrutkovs/openshift-demo
```

Route: demo.cloud.vrutkovs.eu

Webhook: https://github.com/vrutkovs/openshift-demo/settings/hooks/23131808

Custom Dockerfile: custom-dockerfile branch

```
oc new-app --name=itgm-custom http://github.com/vrutkovs/openshift-demo#custom-dockerfile
oc create -f route.yaml
```
---
# Demo
### Jenkins Pipelines
### Blue - Green deployments

Note:
Namespace: pipelines

Pipelines branch: jenkins

```
oc new-project pipelines
oc new-app --name=jenkins-pipeline http://github.com/vrutkovs/openshift-demo#jenkins
```
---
# Demo
### Gitlab CI
### Blue-green deployment
### Service catalog

Note:
Namespace: blue-green

Blue-green branch: blue-green

Service Catalog: Amazon RDS
---
#### Monitoring and Metrics
![BMO](imgs/BMO.png)

Note:
Prometheus: https://prometheus.cloud.vrutkovs.eu/

Grafana: https://grafana.cloud.vrutkovs.eu/

---
### See you later, operator

**Operators** - k8s-aware application, which communicate using CRDS (custom resource definitions) and perform actions in the cluster

* **Vault Operator**

  creates and configures Hashicopr's Vault cluster

* **MySQL Operator**

  creates, scales and backs up MySQL containers in kubernetes

---
#### Openshift operators
* **Openshift Metrics Server**

  scales deployments based on custom app metrics

* **Node Problem Detector**

  uses Prometheus metric to disable faulty nodes

* **Autoscaler**

  provision additional nodes
* **Chargeback**
 
  reports AWS billing, node utilization etc.

---
![shifty](imgs/get_shifty.jpg)

https://vrutkovs.github.io/slides-openshift-k8s-human-face/

*<!-- -->* vrutkovs  <!-- .element: class="fab fa-twitter-square" --> *<!-- -->* vrutkovs@redhat.com  <!-- .element: class="fas fa-envelope-square" -->
