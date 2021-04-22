# Dual-Network IBM Cloud Pak for Integration Configuration via OpenShift Multus
###### This documentation was created to provide validated instructions for connecting two networks to CP4I via OpenShift Multus

### Requirements

- OpenShift 4.6.8 or later [OpenShift 4.6 Installation Docs](https://docs.openshift.com/container-platform/4.6/welcome/index.html)
- Cloud Pak for Integration 2021.1 [Cloud Pak for Integration Installation Docs](https://www.ibm.com/docs/en/cloud-paks/cp-integration/2021.1?topic=installing)

### Connecting Secondary Network to OpenShift Nodes
1. Power off all compute nodes
![](/assets/vm-off.png)

2. Edit the VM settings of each compute node and add an additional network adapter
![](/assets/net-adapters.png)

3. Power on all compute nodes
![](/assets/vm-on.png)

### Ensure the OpenShift cluster is stable
1. Login to via the *oc* command-line tool
2. Check for *certificate signing requests* (csr)
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

### Configure the Cluster Network Operator
Configuring the CNO with an "additional network" will cause OpenShift Multus to connect the 2nd network, and standby for connections to pods as needed.
1. Use the *oc edit* command to modify the configuration
```bash
oc edit networks.operator.openshift.io cluster
```

###### This command will open a vi, vim or other text editor where you will make your edits. Be sure to save/write before closing or quitting out of the editor.

2. See the example *additionalNetworks* spec below:
```yaml
apiVersion: operator.openshift.io/v1
kind: Network
metadata:
  name: cluster
spec:
  additionalNetworks:
  - name: ipvlan-main
  namespace: cp4i # Your CP4I project / namespace
  rawCNIConfig: ’{
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
  }’
  type: Raw
...
```
###### Please remove the comments following the *namespace* and *master* fields - they form improper JSON in the rawCNIConfig.

### Connect the 2nd network to CP4I Deployments
Connecting CP4I to Multus requires the addition of network annotations to all deployments that need access to the second network.
Repeat steps 3-5 for each deployment.

1. Make sure you're within your CP4I Project Conext
```bash
oc project <cp4i-project>
```

2. Validate that a Network Attachment Definition was created. (If you don't have an NAD, you will need to edit your CNO again)
```bash
oc get net-attach-def
```
![](/assets/get-nad.png)

3. Identify the CP4I Component (Deployment or StatefulSet) you'd like to connect to the 2nd network
```bash
oc get deploy
```
or
```bash
oc get sts
```
![](/assets/get-deploy.png)

4. Edit the target component
```bash
oc edit deploy <cp4i-component>
```

5. Add the annotation to the *spec.template.metadata.annotations* field. See the example snippet below for help.
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

### Validating the 2nd network is connected
1. Identify a pod that is running the component from before
```bash
oc get po
```

2. Inspect the events on that pod to see whether the network was attached successfully
The events are at the bottom of the description output
```bash
oc describe po <cp4i-pod-name>
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
