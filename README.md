# Kubernetes Lab

Deploys a Kubernetes Lab on Nectar Research Cloud

## Setup

There is no need to clone this repo. All necessary resources can be referred to over HTTP.

### Install Requirements

Install the OpenStack and Heat clients. For Ubuntu, this will be:

```
apt-get install python-openstackclient python-heatclient
```

## Process

### Create Lab

If you created a virtual environment above, ensure you have activated that virtual environment, and that the OpenStack and Heat clients are installed. Create your Kubernetes Lab by running the following command:

```
$ openstack stack create --wait --template https://raw.githubusercontent.com/NeCTAR-RC/lab-kubernetes/master/heat/stack.yaml lab-kubernetes-01
```

This will generate a Heat (Orchestration) Stack containing all the necessary resources for your Kubernes Lab, bootstrap the Master Node, and join any Worker Nodes if specified.

#### Parameters

The `worker_node_count` parameter allows specifying the number of Kubernetes Worker Nodes you would like deployed. If omitted, it defaults to `worker_node_count=2`. If `worker_node_count=0` is specified, then no Kubernetes Worker Nodes will be deployed, and the Kubernetes Master Node will be untainted to allow scheduling of Pods to itself.

The `lab_repo` parameter allows specifying an alternative repo to clone from, perhaps a fork you maintain or are testing changes in. If omitted, it defaults to this (`https://github.com/NeCTAR-RC/lab-kubernetes.git`) repo.

The `lab_repo_version` parameter allows specifying an alternative version to checkout, such as a specific tag or branch. If omitted, it defaults to the `master` branch.

### Check Lab Status

Check on the status of your Lab build by running the following command:

```
$ openstack stack show lab-kubernetes-01
```

You will know it has successfully completed when the `stack_status` goes into `CREATE_COMPLETE`. If something when wrong, the status will show as `CREATE_FAILED`, and the `stack_status_reason` will provide details of what went wrong.

### Access Lab

After the Lab has successfully completed building, you will need to fetch the IP Addresses and Credentials of your Lab.  You can do that with the following commands:

```
$ openstack stack output list lab-kubernetes-01
$ openstack stack output show -c output_value -f value lab-kubernetes-01 master_node_ip_address
$ openstack stack output show -c output_value -f value lab-kubernetes-01 worker_nodes_ip_address
$ openstack stack output show -c output_value -f value lab-kubernetes-01 password
$ openstack stack output show -c output_value -f value lab-kubernetes-01 private_key > private-key.pem

$ chmod 600 private-key.pem
```

There is both a `password` and a `private_key`, either of which you can use to access the Lab. Below is an example using the `private_key` to access one of the Lab Nodes.

```
$ ssh -i private-key.pem ubuntu@<worker-or-master-node-ip-address>
```

### Verify Status of Kubernetes

After logging into the Master Node, verify the status of Kubernetes by running the following commands:

```
$ kubectl get nodes
```

#### Sample Output

```
NAME            STATUS    ROLES     AGE       VERSION
test-master     Ready     master    53m       v1.10.2
test-worker-0   Ready     <none>    52m       v1.10.2
test-worker-1   Ready     <none>    52m       v1.10.2
test-worker-2   Ready     <none>    52m       v1.10.2
```

```
$ kubectl get all --all-namespaces
```

#### Sample Output

```
NAMESPACE     NAME                                      READY     STATUS    RESTARTS   AGE
kube-system   pod/etcd-test-master                      1/1       Running   0          52m
kube-system   pod/kube-apiserver-test-master            1/1       Running   0          52m
kube-system   pod/kube-controller-manager-test-master   1/1       Running   0          52m
kube-system   pod/kube-dns-86f4d74b45-kbg5c             3/3       Running   0          52m
kube-system   pod/kube-flannel-ds-649cx                 1/1       Running   0          52m
kube-system   pod/kube-flannel-ds-cdq7k                 1/1       Running   2          52m
kube-system   pod/kube-flannel-ds-drh7q                 1/1       Running   2          52m
kube-system   pod/kube-flannel-ds-vtdnf                 1/1       Running   0          52m
kube-system   pod/kube-proxy-7x4bj                      1/1       Running   0          52m
kube-system   pod/kube-proxy-d6rzw                      1/1       Running   0          52m
kube-system   pod/kube-proxy-gprt5                      1/1       Running   0          52m
kube-system   pod/kube-proxy-rwn9w                      1/1       Running   0          52m
kube-system   pod/kube-scheduler-test-master            1/1       Running   0          51m

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP         53m
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP   53m

NAMESPACE     NAME                             DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                   AGE
kube-system   daemonset.apps/kube-flannel-ds   4         4         4         4            4           beta.kubernetes.io/arch=amd64   53m
kube-system   daemonset.apps/kube-proxy        4         4         4         4            4           <none>                          53m

NAMESPACE     NAME                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/kube-dns   1         1         1            1           53m

NAMESPACE     NAME                                  DESIRED   CURRENT   READY     AGE
kube-system   replicaset.apps/kube-dns-86f4d74b45   1         1         1         52m
```

### Delete Lab

When you are done with your Lab, you can delete the Stack and all of its resources by running the following command:

```
$ openstack stack delete --yes --wait lab-kubernetes-01
```

The `--wait` option will cause the command to not return until the Stack had been fully deleted. If you want the delete to be asynchronous, then you may omit this option.
