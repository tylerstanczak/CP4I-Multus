# Dual-Network IBM Cloud Pak for Integration Installation and Configuration with OpenShift Multus
This documentation was created to provide validated instructions for connecting two networks to CP4I capabilities and runtimes via OpenShift Multus.
**This guide often provides a hyperlink to IBM documentation.** These hyperlinks often redirect to the middle of a documentation tree. Please **do not navigate away from the target page unless instructed to do so.** All of the instructions you need will be on the hyperlinked page!

1. [Requirements](#requirements "Requirements")
2. [Express Installation - Cloud Pak for Integration 2021.1](#express-installation---cloud-pak-for-integration-20211 "Cloud Pak for Integration")
3. [Deploying Operations Dashboard](#deploying-operations-dashboard-with-cp4i "Operations Dashboard")
4. [App Connect Enterprise Deployment](#app-connect-enterprise-deployment "App Connect Enterprise")
5. Deploying API Connect - Coming Soon
6. [Deploy an MQ Queue Manager](#deploy-an-mq-queue-manager-using-cp4i-platform-navigator "Deploy MQ")
7. [Configure Multus](Configuring%20Multus.md "Configuring Multus")

### Requirements

- OpenShift 4.6.12 or later 
  - [Click here to follow the OpenShift 4.6 Installation Docs if you don't already have OpenShift installed](https://docs.openshift.com/container-platform/4.6/welcome/index.html)

- License for Cloud Pak for Integration 2021.1

- The capabilities and runtimes in Cloud Pak for Integration have varying storage requirements. At a minimum, you will require a storage class that supports the RWO access mode. For Asset Repository, Operations Dashboard, App Connect dashboard, Aspera HSTS, and MQ, you will also require a storage class that supports the RWX access mode. The recommended storage providers are:

  - OpenShift Container Storage version 4.x (starting with version 4.2 or higher)
  - Portworx Storage, version 2.5.5 or above
  - IBM Cloud Block Storage and IBM Cloud File Storage

  
- Highly Recommended: Deploy [Operations Dashboard](IBM%20docs%20Installing%20MQ.md) before deploying additional CP4I capabilities

# Express Installation - Cloud Pak for Integration 2021.1

```diff
- During CP4I Installation, name your IBM Entitled Registry Key Secret: ibm-entitlement-key

some CP4I capabilities such as MQ require an IBM entitled registry key with this name
```
  
Be sure to install Platform Navigator during the express installation. 
Platform Navigator install instructions are at the bottom of the express install instructions.

[Click here for the CP4I Express installation instructions](https://www.ibm.com/docs/en/cloud-paks/cp-integration/2021.1?topic=installing-express-installation-cloud-pak-integration)

On this page, scroll down to the 3rd section titled "Installing" and begin following the instructions from there to the end of the page. 

# Deploying Operations Dashboard with CP4I
To enable tracing for capabilities and runtimes, first deploy an ``Operations Dashboard`` (OD) capability by following the instructions in the link below.
Scroll down to the section titled: "Installing Operations Dashboard from the Platform Navigator" on the linked IBM Documentation page.
[IBM Documentation - Deploying Operations Dashboard](https://www.ibm.com/docs/en/cloud-paks/cp-integration/2021.1?topic=configuration-installation)

### Registering a Capability/Runtime for Tracing
```diff
- If you decide to enable tracing during a capability/runtime deployment follow the link below to activate the tracing
```
Scroll down to the section titled: "Capability registration with service binding API" on the linked IBM Documentation page.
[IBM Documentation - Capability Registration](https://www.ibm.com/docs/en/cloud-paks/cp-integration/2021.1?topic=configuration-capability-registration)

# App Connect Enterprise Deployment
There are three simple ways to deploy the App Connect dashboard

- CP4I Platform Navigator UI
- OpenShift Installed Operators UI
- OpenShift CLI

Click the link below to find those three options and their respective instructions

[IBM Documentation - App Connect Dashboard reference](https://www.ibm.com/docs/en/app-connect/11.0.0?topic=resources-dashboard-reference#install)

After installing and accessing the dashboard, you can create integration servers with .bar files per usual

# Deploy an MQ Queue Manager using CP4I Platform Navigator

Click the link below and begin at step 1a of the ``Procedure`` section. (If you already created an IBM Entitled Registry Key Secret named: ``ibm-entitlement-key``, you don't need to follow the ``Preparing your OpenShift project for IBM MQ`` hyperlink.)

![IBM Documentation - Deploying a Queue Manager](https://www.ibm.com/docs/en/ibm-mq/9.2?topic=dqmorhocpc-deploying-queue-manager-using-cloud-pak-integration-platform-navigator)

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
