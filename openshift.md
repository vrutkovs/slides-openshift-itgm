# Openshift
### Kubernetes with a human face

Vadim Rutkovsky

vrutkovs@redhat.com

---
![Cat pic](imgs/cat.png)

Note:
* Enterprise Kubernetes, developer-focused
* Forked from Kubernetes codebase, regularly synced with upstream (3 month delay)
* Origin - upstream version, Openshift Container Platform - enterprise version
* OCP versions are supported for 2 years
* Additional k8s objects - ImageStream, BuildConfigs, DeploymentConfigs

---
### What's in the box?
![openshift](imgs/openshift.png)

Note:
* Base layer - your cloud provider / VMs / bare-metal
* OS with container runtime - Docker / CRI-O
* SDN / Storage / core services
* Developer services - CI/CD, Service Catalog, Web Console

---
### Openshift - batteries included
* OAuth server for authentication
* Container registry
* HAProxy router
* CI/CD out-of-the-box via Jenkins Pipelines
* Source2Image

![Professor Fortran](imgs/fortran.png)

Note:
* All things ready to get started
* Encourages CI / CD
* Devs don't require low-level Docker knowledge with S2I

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
### Source 2 Image
### Builds, DeploymentConfigs
### Routes

---

Look mom, no Dockerfile!

```shell
oc login https://cloud.vrutkovs.eu -t ...
oc new-project beer
oc new-app  \
   --name=beer-demo \
   https://github.com/vrutkovs/openshift-demo
oc expose svc/demo --host=demo.cloud.vrutkovs.eu
```

Setup a github webhook to trigger builds on new commits

---

![Web Console](imgs/build_logs.png)
![Build logs](imgs/web_console.png)

---

Dockerfile + route settings in YAML

```shell
oc new-project beer-custom
oc new-app --name=beer-custom \
   http://github.com/vrutkovs/openshift-demo#custom-dockerfile
oc create -f route.yaml
```

---
### Jenkins Pipelines

---

```shell
oc new-project pipelines
oc new-app --name=jenkins-pipeline \
   http://github.com/vrutkovs/openshift-demo#jenkins
```

![Build logs](imgs/jenkins_pipeline.png)

---
```groovy
stage("Build") {
    openshiftBuild
      buildConfig: "pipeline-app", showBuildLogs: "true"
}

stage("Deploy to dev") {
    openshiftDeploy deploymentConfig: "pipeline-app"
}

stage("Smoketest") {
  sh "curl -kLvs
      http://pipeline-app.pipelines.svc:8080/Minsk |
      grep 'Hello, Minsk'"
}

stage("Deploy to tested") {
  openshiftCOLOR
    srcStream: "pipeline-app", srcCOLOR: 'latest',
    destinationStream: "pipeline-app",
    destinationCOLOR: "smoketested"
  openshiftDeploy deploymentConfig: "pipeline-app-tested"
}
```
---
### Gitlab CI
### Blue-green deployment
---
```yaml
before_script:
  - oc login --insecure-skip-tls-verify=true
      kubernetes.default.svc --token=$OPENSHIFT_TOKEN
  - oc project blue-green
  - export COLOR="blue" ALTCOLOR="green"
  - export ACTIVE=$(
      oc get route prod-route -o
      jsonpath='{ .spec.to.name }' --loglevel=4)
  - if [ $ACTIVE == "blue" ]; then
      export COLOR="green" ALTCOLOR="blue"
    fi
  - export COLORHOST="$COLOR.blue-green.svc"

build:
  stage: build
  script:
    - oc start-build blue-green --wait
```
---
```yaml
deploy:
  stage: deploy
  script:
    - oc COLOR blue-green:latest blue-green:${COLOR}
    - oc rollout latest dc/${COLOR}
    - oc rollout status dc/${COLOR} -w

autotest:
  stage: autotest
  script:
    - curl -kLs http://${COLORHOST}:8080/ | tee | grep 'Anonymous'
    - curl -kLs http://${COLORHOST}:8080/beer | tee | grep 'beer'

canary:
  stage: canary
  script:
    - for i in `seq 10 10 100`; do
        oc set route-backends prod-route \
           ${COLOR}=$i ${ALTCOLOR}=$((100-i))
        && sleep 3;
      done
```
---

![Gitlab pipelines](imgs/gitlab_pipelines.png)
![Blue-green console](imgs/webconsole_bluegreen.png)


---
#### Monitoring and Metrics
![BMO](imgs/BMO.png)

---

![Hawkular metrics](imgs/hawkular.png)
![Grafana](imgs/grafana.png)

---
### See you later, operator

**Operators** - k8s-aware application, which communicate using CRDS (custom resource definitions) and perform actions in the cluster

* **Vault Operator**

  creates and configures Hashicorp's Vault cluster

* **MySQL Operator**

  creates, scales and backs up MySQL containers in kubernetes

Note:
Cloud-native apps - the apps which are aware of running in k8s and can react to k8s events
Operators take care of running complicated apps, e.g. databases

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

Note:
* metrics server scales your apps with demand
* problem detector finds and isolates problemtic nodes
* autoscaler adds more machines to cluster if pods don't have place to fit
* chargeback find less utilized nodes and may calculate cloud-provider bills

---
Install using Ansible

https://docs.openshift.org/latest/install_config/

Openshift Online

https://manage.openshift.com/

Openshift Dedicated

https://www.openshift.com/dedicated/

Cloud IDE

https://openshift.io

Note:

* openshift-ansible to install on any infrastructure
* Online to try-before-you-buy
* Dedicated - managed by Red Hat on AWS
* openshift.io to develop Java microservices online using Eclipse Che
---
![shifty](imgs/get_shifty.jpg)

https://vrutkovs.github.io/slides-openshift-k8s-human-face/

*<!-- -->* vrutkovs  <!-- .element: class="fab fa-twitter-square" --> *<!-- -->* vrutkovs@redhat.com  <!-- .element: class="fas fa-envelope-square" -->
