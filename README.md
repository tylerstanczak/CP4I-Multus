# IBM Cloud Pak for Integration Multus Configuration
###### This documentation was created to provide 

### Requirements

- OpenShift 4.6.8 or later ![OpenShift 4.6 Installation Docs](https://docs.openshift.com/container-platform/4.6/welcome/index.html)
- Cloud Pak for Integration 2021.1 ![Cloud Pak for Integration Installation Docs](https://www.ibm.com/docs/en/cloud-paks/cp-integration/2021.1?topic=installing)

### Connecting Secondary Network to OpenShift Nodes
1. Power off all compute nodes
2. Edit the VM settings of each compute node and add an additional network adapter
3. Power on all compute nodes

### Ensure the OpenShift cluster is stable
1. Login to via the *oc* command-line tool
2. Check for *certificate signing requests* (csr)
```bash
oc get csr
```
3. If there are any pending csr's, be sure to approve them
 ```bash
 yum install jq -y
 oc get csr -ojson | jq -r '.items[] | select(.status == {} ) | .metadata.name' | xargs oc adm certificate approve
 ```
4. Check node status and be sure they are all "Ready"
```bash
oc get no
```

### Identify the name of the second network adapter on each compute node
```diff
- ALL SECONDARY ADAPTERS MUST SHARE THE SAME NAME
```
##### Repeat the steps below for each compute node
1. Open a a debug shell to each compute node
```bash
oc debug no/<node-name>
```
2. Use the node's host binaries
```bash
chroot /host
```
3. Check the network adapter name
```bash
ip a
```

### Configure the Cluster Network Operator
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
  namespace: cp4i
  rawCNIConfig: ’{
    "cniVersion": "0.3.1",
 ⭐️ "name": "ipvlan-main", ⭐️
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

