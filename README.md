# autorollout-operator
autorollout-operator is k8s operator which can perform rollout restart of deployments at regular intervals of time

## Description
autorollout-operator is k8s operator created using operator-sdk https://sdk.operatorframework.io/ that performs rolling restart of deployment by patching filtered deployments with annotation ```kubectl.kubernetes.io/restartedAt``` at regular intervals of time. The operator also tries to ensure in case of failure, it is reconciled till success occurs. The operator watches on custom resource : ```apiVersion: crd.ricktech.io/v1alpha1``` and ```kind: Flipper``` which is used to specify both interval and label selectors to filter the deployments 

![alt text](https://raw.githubusercontent.com/rickslick/autorollout-operator/main/docs/basicworkflow.jpeg)



![alt text](https://raw.githubusercontent.com/rickslick/autorollout-operator/main/docs/flow-autorollout.jpg)
Eg :

```
apiVersion: crd.ricktech.io/v1alpha1
kind: Flipper
metadata:
  labels:
    app.kubernetes.io/name: flipper
    app.kubernetes.io/instance: flipper-sample
    app.kubernetes.io/part-of: autorollout-operator
    app.kubernetes.io/managed-by: kustomize
    app.kubernetes.io/created-by: autorollout-operator
  name: flipper-redis
spec:
  interval: 12h
  match:
    namespace: redis
    labels:
      mesh: enabled
```
### Features

* validation webhook to verify presence of atleast one label and interval in proper format
* mutating webhook to set the default interval to 10m if not provided
* auto retry for failed rollout restarts(failed patches)
* automatic rollout restart at every interval
* Restarts only Ready deployments
  
## Getting Started

### Prerequisites
- go version v1.20.0+
- docker version 17.03+.
- kubectl version v1.11.3+.
- Access to a Kubernetes v1.11.3+ cluster.
- Cert-manager installed in cluster - (webhook)
- Prometheus-operator install in cluster (serviceMonitor metrics)
### To Deploy on the cluster
**Build and push your image to the location specified by `IMG`:**

```sh
make docker-build docker-push IMG=<some-registry>/autorollout-operator:tag
```

**NOTE:** This image ought to be published in the personal registry you specified. 
And it is required to have access to pull the image from the working environment. 
Make sure you have the proper permission to the registry if the above commands don’t work.

**Install the CRDs into the cluster:**

```sh
make install
```

**Deploy the Manager to the cluster with the image specified by `IMG`:**

```sh
make deploy IMG=<some-registry>/autorollout-operator:tag
```

Example 
```
make deploy
namespace/autorollout-operator-system created
customresourcedefinition.apiextensions.k8s.io/flippers.crd.ricktech.io created
serviceaccount/autorollout-operator-controller-manager created
role.rbac.authorization.k8s.io/autorollout-operator-leader-election-role created
clusterrole.rbac.authorization.k8s.io/autorollout-operator-manager-role created
clusterrole.rbac.authorization.k8s.io/autorollout-operator-metrics-reader created
clusterrole.rbac.authorization.k8s.io/autorollout-operator-proxy-role created
rolebinding.rbac.authorization.k8s.io/autorollout-operator-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/autorollout-operator-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/autorollout-operator-proxy-rolebinding created
service/autorollout-operator-controller-manager-metrics-service created
service/autorollout-operator-webhook-service created
deployment.apps/autorollout-operator-controller-manager created
certificate.cert-manager.io/autorollout-operator-serving-cert created
issuer.cert-manager.io/autorollout-operator-selfsigned-issuer created
servicemonitor.monitoring.coreos.com/autorollout-operator-controller-manager-metrics-monitor created
mutatingwebhookconfiguration.admissionregistration.k8s.io/autorollout-operator-mutating-webhook-configuration created
validatingwebhookconfiguration.admissionregistration.k8s.io/autorollout-operator-validating-webhook-configuration created
```


> **NOTE**: If you encounter RBAC errors, you may need to grant yourself cluster-admin 
privileges or be logged in as admin.

**Create instances of your solution**
You can apply the samples (examples) from the config/sample (negative test also present):

```sh
kubectl apply -k config/samples/
```

>**NOTE**: Ensure that the samples has default values to test it out.

### To Uninstall
**Delete the instances (CRs) from the cluster:**

```sh
kubectl delete -k config/samples/
```

**Delete the APIs(CRDs) from the cluster:**

```sh
make uninstall
```

**UnDeploy the controller from the cluster:**

```sh
make undeploy
```


### Unit test

```
ok      github.com/rickslick/autorollout-operator/api/v1alpha1  4.373s  coverage: 29.2% of statements
ok      github.com/rickslick/autorollout-operator/internal/controller   6.371s  coverage: 82.9% of statements
ok      github.com/rickslick/autorollout-operator/internal/utils        1.168s  coverage: 96.6% of statements
```

## Contributing


More information can be found via the [Kubebuilder Documentation](https://book.kubebuilder.io/introduction.html)

## License

Copyright 2024.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

