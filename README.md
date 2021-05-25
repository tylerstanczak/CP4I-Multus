# CP4I Install with Multus

Follow the steps below

1. [Requirements](#requirements "Requirements")
2. [Express Installation - Cloud Pak for Integration 2021.1](#express-installation---cloud-pak-for-integration-20211 "Cloud Pak for Integration")
3. [Deploying Operations Dashboard](Deploy%20Operations%20Dashboard.md "Operations Dashboard")
4. [Deploying App Connect Enterprise](Deploy%20ACE.md "App Connect Enterprise")
5. Deploying API Connect - Coming Soon
6. [Deploying MQ](Deploy%20MQ.md "Deploy MQ")
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

On this page, scroll down to the 3rd section titled "Installing" and begin following the instructions there.
