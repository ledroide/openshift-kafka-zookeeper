# openshift-kafka-zookeeper
testing kafka and zookeeper as statefullsets over an openshift cluster

## CLI installation

Watch for openshift-origin-client-tools and download the latest stable release from https://github.com/openshift/origin/releases :

```bash
wget https://github.com/openshift/origin/releases/download/v3.9.0-alpha.3/openshift-origin-client-tools-v3.9.0-alpha.3-78ddc10-linux-64bit.tar.gz
tar xzf openshift-origin-client-tools-v3.9.0-alpha.3-78ddc10-linux-64bit.tar.gz
sudo mv openshift-origin-client-tools-v3.7.1-ab0f056-linux-64bit/oc /usr/local/bin/
oc version
```

### openshift inside minishift option

* watch for minishift and download the latest stable release from https://github.com/minishift/minishift/releases :

```bash
wget https://github.com/openshift/origin/releases/download/v3.9.0-alpha.3/openshift-origin-client-tools-v3.9.0-alpha.3-78ddc10-linux-64bit.tar.gz
tar xzf minishift-1.13.1-linux-amd64.tgz
sudo mv minishift-1.13.1-linux-amd64/minishift /usr/local/bin/
minishift version
```

* launch the openshift cluster including metrics enabled : `minishift start --vm-driver virtualbox --metrics`
* WebUI dashboard should be at https://192.168.0.100:8443/ (check with `minishit console --url`)

### openshift inside docker option

#### dockerd configuration

* edit the /etc/docker/daemon.json file and add the following :

```json
{
   "insecure-registries": [
     "172.30.0.0/16"
   ]
}
```

* restart dockerd : `sudo systemctl restart docker` and check configuration with `docker info`.
* launch the openshift cluster including metrics enabled : `oc cluster up --metrics`
* WebUI dashboard should be at https://127.0.0.1:8443/ (check with `oc cluster status`)
* further documentation on https://github.com/openshift/origin/blob/master/docs/cluster_up_down.md

## check openshift CLI and interface

* CLI login : `oc login -u system:admin`
* WebUI dashboard login : _developer/developer_

## kafka and zookeeper as stateful sets

> forked from https://github.com/strimzi/strimzi/blob/0.1.0/kafka-statefulsets/resources/openshift-template.yaml

* add the kafka+zookeeper (named _kafka-zk_ including 3 instances for each component) template to openshift : `oc apply -f openshift-kafka-zk-template.yaml`
* run a new app based on this template : `oc new-app kafka-zk`
* wait for everything running : `watch oc get all`

## prometheus service collecting kafka metrics

> forked from https://github.com/strimzi/strimzi/blob/0.1.0/metrics/prometheus/openshift-template.yaml 

* create a prometheus-server _service account_ resource :

```bash
export NAMESPACE=default
oc create sa prometheus-server
oc adm policy add-cluster-role-to-user cluster-reader system:serviceaccount:${NAMESPACE}:prometheus-server
```

* add prometheus template to openshift : `oc apply -f openshift-prometheus-template.yaml`
* run a new app based on this template : `oc new-app prometheus`
* in order to access prometheus WebUI dashboard : `oc expose svc/prometheus`
* further documentation on https://github.com/strimzi/strimzi/blob/0.1.0/metrics/METRICS.md 

## grafana connection to prometheus

> forked from https://github.com/OpenShiftDemos/grafana-openshift

* add the grafana template to openshift : `oc apply -f openshift-grafana-template.yaml`
* run a new app based on this template : `oc new-app grafana`

## yahoo kafka manager

> * based on https://github.com/yahoo/kafka-manager
> * forked from https://github.com/Yolean/kubernetes-kafka/tree/master/yahoo-kafka-manager

* add prometheus template to openshift : `oc apply --dry-run --validate -f openshift-kafka-manager-template.yaml`
* \(not working yet)

## todo

* [ ] expose a route automatically from prometheus template
* [ ] include kafka-manager for high-level monitoring
* [ ] kafka : add a swtich for EmptyDir volume option

## links

* [Kubernetes 1.7 API reference](https://v1-7.docs.kubernetes.io/docs/api-reference/v1.7/) for OpenShift 3.7 Blueprints
* [Mardown syntax help](https://help.github.com/articles/basic-writing-and-formatting-syntax)

## notes

```
oc apply --dry-run --validate -f openshift-kafka-zk-template.yaml
error: error validating "openshift-kafka-zk-template.yaml": error validating data: couldn't find type: v1.Template; if you choose to ignore these errors, turn validation off with --validate=false
```

```
oc new-app kafka-manager
error: Errors occurred while determining argument types:
kafka-manager as a local directory pointing to a Git repository:  stat kafka-manager: no such file or directory
Errors occurred during resource creation:
error: error processing template "myproject/kafka-manager": only encoded map or array can be decoded into a struct
```
