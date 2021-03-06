# Lab 2 - Going HA

In this lab we will go through both GUI and CLI methods for upgrading a Kubernetes cluster on DC/OS to be Highly Available

By default, the HA option in DC/OS will configure:
- 3x etcd instances
- 3x kube-apiserver instances
- 3x kube-scheduler instances
- 3x kube-controller-manager instances

### Making Kubernetes Highly Available through the GUI

You can choose to make Kubernetes on DC/OS high availability (HA) at the time of deployment. However, you can also make non-HA clusters into HA by editing and saving the configuration. 

In the DC/OS UI under Services, click on your Kubernetes cluster (Not kube-proxy). 

In the top left, click Edit.

![](https://i.imgur.com/2dYmVLp.png)

Under Kubernetes in left hand menu, choose the high availability box

![](https://i.imgur.com/PkGHHlJ.png)

Save configuration and watch new components come online with the following command or in the GUI.
```
dcos kubernetes plan status deploy
``` 

Output should look like below:

```
$ dcos kubernetes plan status deploy
deploy (serial strategy) (COMPLETE)
├─ etcd (serial strategy) (COMPLETE)
│  ├─ etcd-0:[peer] (COMPLETE)
│  ├─ etcd-1:[peer] (COMPLETE)
│  └─ etcd-2:[peer] (COMPLETE)
├─ apiserver (serial strategy) (COMPLETE)
│  ├─ kube-apiserver-0:[instance] (COMPLETE)
│  ├─ kube-apiserver-1:[instance] (COMPLETE)
│  └─ kube-apiserver-2:[instance] (COMPLETE)
├─ kubernetes-api-proxy (serial strategy) (COMPLETE)
│  └─ kubernetes-api-proxy-0:[install] (COMPLETE)
├─ controller-manager (serial strategy) (COMPLETE)
│  ├─ kube-controller-manager-0:[instance] (COMPLETE)
│  ├─ kube-controller-manager-1:[instance] (COMPLETE)
│  └─ kube-controller-manager-2:[instance] (COMPLETE)
├─ scheduler (serial strategy) (COMPLETE)
│  ├─ kube-scheduler-0:[instance] (COMPLETE)
│  ├─ kube-scheduler-1:[instance] (COMPLETE)
│  └─ kube-scheduler-2:[instance] (COMPLETE)
├─ node (serial strategy) (COMPLETE)
│  ├─ kube-node-0:[kube-proxy] (COMPLETE)
│  ├─ kube-node-0:[coredns] (COMPLETE)
│  └─ kube-node-0:[kubelet] (COMPLETE)
├─ public-node (serial strategy) (COMPLETE)
└─ mandatory-addons (serial strategy) (COMPLETE)
   ├─ mandatory-addons-0:[kube-dns] (COMPLETE)
   ├─ mandatory-addons-0:[metrics-server] (COMPLETE)
   ├─ mandatory-addons-0:[dashboard] (COMPLETE)
   └─ mandatory-addons-0:[ark] (COMPLETE)
```

### Making Kubernetes Highly Available through the CLI

Grab the Kubernetes options.json file:

```
dcos kubernetes describe > options.json
```

Edit the options.json file to set "high_availability": true,

```
$ cat options.json
{
  "apiserver": {
    "cpus": 0.5,
    "mem": 1024
  },
  "controller_manager": {
    "cpus": 0.5,
    "mem": 512
  },
  "etcd": {
    "cpus": 0.5,
    "data_disk": 3072,
    "disk_type": "ROOT",
    "mem": 1024,
    "wal_disk": 512
  },
  "kube_proxy": {
    "cpus": 0.1,
    "mem": 512
  },
  "kubernetes": {
    "cloud_provider": "(none)",
    "high_availability": true,
    "network_provider": "dcos",
    "node_count": 1,
    "public_node_count": 0,
    "reserved_resources": {
      "kube_cpus": 2,
      "kube_disk": 10240,
      "kube_mem": 2048,
      "system_cpus": 1,
      "system_mem": 1024
    },
    "service_cidr": "10.100.0.0/16"
  },
  "scheduler": {
    "cpus": 0.5,
    "mem": 512
  },
  "service": {
    "log_level": "INFO",
    "name": "kubernetes",
    "service_account": "",
    "service_account_secret": "",
    "sleep": 1000
  }
}
```

Update the Kubernetes Framework:
```
dcos kubernetes update --options=options.json
```

Output should resemble below:

```
The components of the cluster will be updated according to the changes in the
options file [options.json].

Updating these components means the Kubernetes cluster may experience some
downtime or, in the worst-case scenario, cease to function properly.
Before updating proceed cautiously and always backup your data.

This operation is long-running and has to run to completion.
Are you sure you want to continue? [yes/no] yes

2018/07/09 10:09:29 starting update process...
2018/07/09 10:09:29 waiting for update to finish...
2018/07/09 10:12:50 update complete!
```

Validate update against the plan:
```
dcos kubernetes plan status deploy
```

Output should look similar to below when complete:
```
$ dcos kubernetes plan status deploy
deploy (serial strategy) (COMPLETE)
├─ etcd (serial strategy) (COMPLETE)
│  ├─ etcd-0:[peer] (COMPLETE)
│  ├─ etcd-1:[peer] (COMPLETE)
│  └─ etcd-2:[peer] (COMPLETE)
├─ apiserver (serial strategy) (COMPLETE)
│  ├─ kube-apiserver-0:[instance] (COMPLETE)
│  ├─ kube-apiserver-1:[instance] (COMPLETE)
│  └─ kube-apiserver-2:[instance] (COMPLETE)
├─ kubernetes-api-proxy (serial strategy) (COMPLETE)
│  └─ kubernetes-api-proxy-0:[install] (COMPLETE)
├─ controller-manager (serial strategy) (COMPLETE)
│  ├─ kube-controller-manager-0:[instance] (COMPLETE)
│  ├─ kube-controller-manager-1:[instance] (COMPLETE)
│  └─ kube-controller-manager-2:[instance] (COMPLETE)
├─ scheduler (serial strategy) (COMPLETE)
│  ├─ kube-scheduler-0:[instance] (COMPLETE)
│  ├─ kube-scheduler-1:[instance] (COMPLETE)
│  └─ kube-scheduler-2:[instance] (COMPLETE)
├─ node (serial strategy) (COMPLETE)
│  ├─ kube-node-0:[kube-proxy] (COMPLETE)
│  ├─ kube-node-0:[coredns] (COMPLETE)
│  └─ kube-node-0:[kubelet] (COMPLETE)
├─ public-node (serial strategy) (COMPLETE)
└─ mandatory-addons (serial strategy) (COMPLETE)
   ├─ mandatory-addons-0:[kube-dns] (COMPLETE)
   ├─ mandatory-addons-0:[metrics-server] (COMPLETE)
   ├─ mandatory-addons-0:[dashboard] (COMPLETE)
   └─ mandatory-addons-0:[ark] (COMPLETE)
```

## Done with Lab 2
Updating to High Availability is simple with Kubernetes on DC/OS. Small projects can start as non-HA to save resources and update to HA when moving projects to production.

[Labs home](https://github.com/c-mcinerney/kubernetes-labs)

[Lab 1 - Installing Kubernetes](https://github.com/c-mcinerney/kubernetes-labs/blob/master/Lab%201%20-%20Installing%20Kubernetes.md)


[Lab 2 - Upgrading Cluster](https://github.com/c-mcinerney/kubernetes-labs/blob/master/Lab%202%20-%20Upgrading%20Cluster.md)

[Lab 3 - Going HA](https://github.com/c-mcinerney/kubernetes-labs/blob/master/Lab%203%20-%20Going%20HA.md)

[Lab 4 - Killing Node](https://github.com/c-mcinerney/kubernetes-labs/blob/master/Lab%204%20-%20Killing%20Node.md)

[Lab 5 - Running Apps](https://github.com/c-mcinerney/kubernetes-labs/blob/master/Lab%205%20-%20Running%20Apps.md)

[Lab 6 - Scaling Kubernetes](https://github.com/c-mcinerney/kubernetes-labs/blob/master/Lab%206%20-%20Scaling%20Kubernetes.md)

[Lab 7 - Add External Ingress](https://github.com/c-mcinerney/kubernetes-labs/blob/master/Lab%207%20-%20Add%20External%20Ingress.md)

[Lab 8 - Kafka Streams](https://github.com/c-mcinerney/kubernetes-labs/blob/master/Lab%208%20-%20Kafka%20Streams.md)
