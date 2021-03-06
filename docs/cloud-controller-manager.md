# Kubernetes Cloud Controller Manager for Tencent Cloud

**Note**: Chinese documentation please refer [here](https://github.com/tencentcloud/tencentcloud-cloud-controller-manager/blob/master/README_zhCN.md).

`tencentcloud-cloud-controller-manager` is the Kubernetes cloud controller manager implementation for Tencent Cloud Container Service. Read more about cloud controller managers [here](https://kubernetes.io/docs/tasks/administer-cluster/running-cloud-controller/).

**WARNING**: this project is a work in progress and may not be production ready. Currently, only Kubernetes 1.10.x is supported.

## Implementation Details

Currently `tencentcloud-cloud-controller-manager` implements:

* nodecontroller - updates nodes with cloud provider specific labels and addresses.
* routecontroller - responsible for creating routes in vpc
* servicecontroller - responsible for creating LoadBalancers when a service of `Type: LoadBalancer` is created in Kubernetes.

## Requirements

At the current state of Kubernetes, running cloud controller manager requires a few things. Please read through the requirements carefully as they are critical to running cloud controller manager on a Kubernetes cluster on Tencent Cloud.

### --cloud-provider=external
All `kubelet`s in your cluster **MUST** set the flag `--cloud-provider=external`. `kube-apiserver` and `kube-controller-manager` must **NOT** set the flag `--cloud-provider` which will default them to use no cloud provider natively.

**WARNING**: setting `--cloud-provider=external` will taint all nodes in a cluster with `node.cloudprovider.kubernetes.io/uninitialized`, it is the responsibility of cloud controller managers to untaint those nodes once it has finished initializing them. This means that most pods will be left unscheduable until the cloud controller manager is running.

In the future, `--cloud-provider=external` will be the default. Learn more about the future of cloud providers in Kubernetes [here](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/cloud-provider/cloud-provider-refactoring.md).

### Kubernetes node names must match the instance private ipv4 ip

By default, the kubelet will name nodes based on the node's hostname. On TencentCloud Container Service, node hostnames are set based on the private ipv4 ip of the instance. If you decide to override the hostname on kubelets with --hostname-override, this will also override the node name in Kubernetes. It is important that the node name on Kubernetes matches private ipv4 ip of the instance, otherwise cloud controller manager cannot find the corresponding instance to nodes.
