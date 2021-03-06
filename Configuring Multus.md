# Configuring OpenShift Multus
This documentation was created to provide validated instructions for connecting two networks to CP4I via OpenShift Multus.
In this section you will need infrastructure permissions to modify the underlying OpenShift nodes.

### Connect Secondary Network to OpenShift Nodes
1. Power off all compute nodes
![](/assets/vm-off.png)

2. Edit the VM settings of each compute node and add an additional network adapter
![](/assets/net-adapters.png)

3. Power on all compute nodes
![](/assets/vm-on.png)

### Ensure the OpenShift cluster is stable
1. Login to via the *oc* command-line tool
```
Copy and paste the login command from the top-right dropdown under your username in the OCP dashboard
```

3. Check for *certificate signing requests* (csr)
```bash
oc get csr
```
![](/assets/get-csr-1.png)

3. If there are any pending csr's, approve them, and check again
 ```bash
 yum install jq -y
 oc get csr -ojson | jq -r '.items[] | select(.status == {} ) | .metadata.name' | xargs oc adm certificate approve
 oc get csr
 ```
 ![](/assets/get-csr-2.png)
 
4. Check node status and ensure they are all "Ready"
```bash
oc get no
```
![](/assets/get-no.png)

### Identify the name of the second network adapter on each compute node
```diff
- ALL SECONDARY ADAPTERS MUST SHARE THE SAME NAME
```
##### Repeat the steps below for each compute node
1. Open a a debug shell to each compute node
```bash
oc debug no/<node-name>
```
![](/assets/debug.png)

2. Use the node's host binaries
```bash
chroot /host
```

3. Check the name of the second network adapter
```bash
ip a
```
![](/assets/ip-a.png)

### Configure Cluster Network Operator
Configuration of Multus is accomplished by configuring the ``Cluster Network Operator`` (CNO)
Configuring the CNO with an "additional network" will cause OpenShift Multus to connect the 2nd network, and standby for connections to pods as needed.
1. Use the ``oc edit`` command below to modify the configuration of the CNO
```bash
oc edit networks.operator.openshift.io cluster
```

###### This command will open a vi, vim or other text editor where you will make your edits. Be sure to save/write before closing or quitting out of the editor.

2. See an example of the ``additionalNetworks`` spec that must be configured below:
Note there are many different ipam plugins to choose from. Our example uses [whereabouts](https://github.com/k8snetworkplumbingwg/whereabouts)
```yaml
apiVersion: operator.openshift.io/v1
kind: Network
metadata:
  name: cluster
spec:
  additionalNetworks:
  - name: ipvlan-main
    namespace: cp4i # target namespace / project
    rawCNIConfig: '{
      "cniVersion": "0.3.1",
      "name": "ipvlan-main",
      "type": "ipvlan",
      "mode": "l2",
      "master": "ens224", # name of second adapter
      "ipam": {
        "type": "whereabouts",
        "range": "192.168.12.0/24",
        "range_start": "192.168.12.50",
        "range_end": "192.168.12.200"
      }
    }'
    type: Raw
...
```
###### Please remove the comments following the *namespace* and *master* fields - they form improper JSON in the rawCNIConfig.

3. Make sure you're within your CP4I Project Context
```bash
oc project <cp4i-project>
```

4. Validate that a ``Network Attachment Definition`` was created. (If you don't have an NAD, you will need to edit your CNO again)
```bash
oc get net-attach-def
```
![](/assets/get-nad.png)

# Attaching additional networks to CP4I workloads
Connecting additional networks to workloads is always done via ``annotations``. In the following sections, you will learn how to assign/apply a special container network interface annotation to each workload. This is a simple process, however, different workloads require the annotation to be inserted at a different level of the custom resource tree. Some capabilities and runtimes may simply be modified at their resulting OCP Deployment or StatefulSet resource (ACE), yet others ignore these modifications and look to their maintaining custom resource one level higher in the tree (MQ).

### Connect the 2nd network to App Connect Enterprise (ACE)
Connecting CP4I to Multus requires the addition of network annotations to all components that need access to the second network.
ACE pods are replicated and controlled by a ``Deployment`` in OCP, so we add the annotation there at the top level.

1. Identify the ACE Deployment or StatefulSet you'd like to connect to the 2nd network
```bash
oc get deploy
```
![](/assets/get-deploy.png)

2. Edit the target component
```bash
oc edit deploy <ace-component>
```

3. Add the annotation to the ``spec.template.metadata.annotations`` field. See the example snippet below for help.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  ... # more metadata
  name: db-01-quickstart-dash
  namespace: cp4i
  ... # more metadata
spec:
... # deployment spec
  template:
    metadata:
      annotations:
        ... # other annotations
        k8s.v1.cni.cncf.io/networks: ipvlan-main
        ... # other annotations
      namespace: cp4i
    spec:
    ... # container spec
```

4. Validating the 2nd network is connected
Identify an ACE pod
```bash
oc get po
```

Inspect the events on that pod to see whether the network was attached successfully
The events are at the bottom of the description output
```bash
oc describe po <ace-pod-name>
```

Here's an example of what App Connect Enterprise pod events look like when Multus is configured correctly:
```bash
[root@dns ~]# oc describe po db-01-quickstart-dash-6c59fdb875-xxlf7
Name:         db-01-quickstart-dash-6c59fdb875-xxlf7
Namespace:    cp4i
... (snippet)
Events:
  Type     Reason          Age                   From               Message
  ----     ------          ----                  ----               -------
  Normal   Scheduled       2m32s                 default-scheduler  Successfully assigned cp4i/db-01-quickstart-dash-6c59fdb875-xxlf7 to worker3
  Normal   AddedInterface  2m29s                 multus             Add eth0 [10.129.2.33/23]
  Normal   AddedInterface  2m29s                 multus             Add net1 [192.168.12.52/24] from cp4i/ipvlan-main
  Normal   Pulled          2m29s                 kubelet            Container image "cp.icr.io/cp/appc/acecc-dashboard-prod@sha256:0fce25498220937f697056684c9d9afd46cb6ef1c9a39875631ecbf1d84f280c" already present on machine
  Normal   Pulled          2m28s                 kubelet            Container image "cp.icr.io/cp/appc/acecc-dashboard-prod@sha256:0fce25498220937f697056684c9d9afd46cb6ef1c9a39875631ecbf1d84f280c" already present on machine
  Normal   Started         2m28s                 kubelet            Started container content-server-init
  Normal   Created         2m28s                 kubelet            Created container content-server-init
  Normal   Created         2m28s                 kubelet            Created container control-ui
  Normal   Started         2m28s                 kubelet            Started container control-ui
  Normal   Pulled          2m28s                 kubelet            Container image "cp.icr.io/cp/appc/acecc-content-server-prod@sha256:db67b9c263ca90deafbd5be6e53baf45bd9723ee7cc049bdd2ef79823c896c7b" already present on machine
  Normal   Created         2m27s                 kubelet            Created container content-server
  Normal   Started         2m27s                 kubelet            Started container content-server
```

### Connect the 2nd network to IBM MQ
Connecting CP4I to Multus requires the addition of network annotations to all components that need access to the second network.
MQ pods are replicated and controlled by a ``StatefulSet``, however, a ``QueueManager`` controls and maintains the ``StatefulSet``.
Because the ``QueueManager`` is the top level resource, the annotation must be added there.

1. Identify the MQ QueueManager you'd like to connect to the 2nd network:
```bash
oc get queuemanager
```

2. Add the annotation to the ``metadata.annotations`` field. See the example snippet below for help.
```yaml
apiVersion: mq.ibm.com/v1beta1
kind: QueueManager
metadata:
  annotations:
    k8s.v1.cni.cncf.io/networks: ipvlan-main
  ... # other metadata
spec:
  license:
    accept: true
  ... # more spec
```

3. Describe the MQ pods to verify that 2 network interfaces have been added in the event list.
Ex:
```
  Normal   AddedInterface  2m29s                 multus             Add eth0 [10.129.2.33/23]
  Normal   AddedInterface  2m29s                 multus             Add net1 [192.168.12.52/24] from cp4i/ipvlan-main
```
