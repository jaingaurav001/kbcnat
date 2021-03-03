## Project Overview

The project demonstrates a simple kubernetes controller using the kubebuilder and test the same using KinD.

This controller is basically a 'Cloud Native At' (cnat) for short – a cloud native version of the Unix's at command. We provide a CRD manifest containing 2 main things:
1. A command to execute
2. Time to execute the command

The controller will basically wait and watch till the appropriate time, then would spawn a Pod and run the given command in that Pod. All this will it will also keep updating the Status of that CRD which we can view with kubectl.

## Install Prerequisite
- Docker
- Kind
- kubectl
- kustomize
- kubebuilder
- Go 1.13

NOTE: Tested with Docker=18.09, Kind=0.8.1, kubernetes=v1.19.0, kubectl=v1.19.0, kustomize=3.7.0, kubebuilder=2.3.1

## Create the setup

Create the kubernetes cluster using KinD

`$ ./scripts/kind-with-registry.sh`

Access the kubernetes cluster

```
$ kubectl cluster-info --context kind-kind
$ kubectl get po --all-namespaces
```

Install the kubernetes controller CRD into kubernetes cluster

`$ make install`

Run the kubernetes controller

```
$ make run
	or
$ make docker-build docker-push IMG=localhost:5000/kbcnat:0.0.1
$ make deploy IMG=localhost:5000/kbcnat:0.0.1
```

Verify the kubernetes controller

```
$ kubectl get po -l control-plane=controller-manager -n kbcnat-system
NAMESPACE            NAME                                         READY   STATUS    RESTARTS   AGE
kbcnat-system        kbcnat-controller-manager-594967764c-d59vr   2/2     Running   0          13s

$ kubectl logs -l control-plane=controller-manager -n kbcnat-system -c manager -f
```

## Test the controller
Schedule a pod using the deployed kubernetes controller.

Create the YAML

```
$ cat config/samples/cnat_v1alpha1_at.yaml
apiVersion: cnat.kbdlr.rocks/v1alpha1
kind: At
metadata:
  name: at-sample
spec:
  schedule: "2019-04-12T10:12:00Z"
  command: "echo Hello World !!"
```

Apply the YAML

```
$ kubectl apply -f config/samples/cnat_v1alpha1_at.yaml
at.cnat.kbdlr.rocks/at-sample created
```

Verify the created CRD resource

`$ kubectl describe at.cnat.kbdlr.rocks/at-sample`

Verify the schedule pod

```
$ kubectl get po
NAME        READY   STATUS      RESTARTS   AGE
at-sample   0/1     Completed   0          23s
```

Verify the pod logs
```
$ kubectl logs at-sample
Hello World !!
```

## Teardown the setup
Remove the kubernetes controller CRD from the kubernetes cluster

`$ make uninstall`

Remove the kubernetes controller

```
$ make kill-deploy IMG=localhost:5000/kbcnat:0.0.1

$ kubectl get po -n kbcnat-system
No resources found in kbcnat-system namespace.
```

Delete the Cluster with Kind

`$ ./scripts/teardown-kind-with-registry.sh`
