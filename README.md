# kubeoperator

A Kubernetes operator that scales Deployments up or down automatically based on a
configurable time window. It watches for `Scaler` custom resources and reconciles the
replica count of the Deployments listed in the spec whenever the current UTC hour falls
within `start` and `end`.

## Description

`kubeoperator` introduces a `Scaler` CRD (`api.palaniraj.kandasamy/v1alpha1`) that lets you
declare, per set of Deployments, a UTC hour window and a target replica count. The
`ScalerReconciler` compares the current hour against that window on every reconcile loop and
patches the matching Deployments' `spec.replicas` to keep them in sync — a lightweight
alternative to running a CronJob for simple scheduled scaling (e.g. spinning up extra
replicas during business hours and scaling back down overnight).

**Example `Scaler` resource:**

```yaml
apiVersion: api.palaniraj.kandasamy/v1alpha1
kind: Scaler
metadata:
  name: scaler-sample
spec:
  start: 9
  end: 18
  replicas: 5
  deployments:
    - name: my-app
      namespace: default
```

## Getting Started

### Prerequisites
- go version v1.21.0+
- docker version 17.03+
- kubectl version v1.11.3+
- Access to a Kubernetes v1.11.3+ cluster

### To Deploy on the cluster

**Build and push your image to the location specified by `IMG`:**

```sh
make docker-build docker-push IMG=<some-registry>/kubeoperator:tag
```

**NOTE:** This image ought to be published in the personal registry you specified.
And it is required to have access to pull the image from the working environment.
Make sure you have the proper permission to the registry if the above commands don't work.

**Install the CRDs into the cluster:**

```sh
make install
```

**Deploy the Manager to the cluster with the image specified by `IMG`:**

```sh
make deploy IMG=<some-registry>/kubeoperator:tag
```

> **NOTE**: If you encounter RBAC errors, you may need to grant yourself cluster-admin
privileges or be logged in as admin.

**Create instances of your solution**

You can apply the samples (examples) from `config/samples`:

```sh
kubectl apply -k config/samples/
```

>**NOTE**: Ensure that the samples have values that make sense for your cluster before applying.

### To Uninstall

**Delete the instances (CRs) from the cluster:**

```sh
kubectl delete -k config/samples/
```

**Delete the APIs (CRDs) from the cluster:**

```sh
make uninstall
```

**UnDeploy the controller from the cluster:**

```sh
make undeploy
```

## Project Distribution

Following are the steps to build the installer and distribute this project to users.

1. Build the installer for the image built and published in the registry:

```sh
make build-installer IMG=<some-registry>/kubeoperator:tag
```

   This generates an `install.yaml` file in the `dist` directory containing all the
   resources built with Kustomize, necessary to install this project without its
   dependencies.

2. Using the installer, users can run the following to install the project:

```sh
kubectl apply -f https://raw.githubusercontent.com/Palanirajkandasamy63/kubeoperator/<tag or branch>/dist/install.yaml
```

## Contributing

Issues and pull requests are welcome. Run `make help` to see all available `make` targets,
and check `make test` / `make lint` before submitting changes.

More information can be found via the [Kubebuilder Documentation](https://book.kubebuilder.io/introduction.html).

## License

Copyright 2024 raj.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
