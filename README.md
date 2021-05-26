# Dual-Network IBM Cloud Pak for Integration Installation and Configuration with OpenShift Multus
This documentation was created to provide validated instructions for connecting two networks to CP4I capabilities and runtimes via OpenShift Multus.
**This guide often provides a hyperlink to IBM documentation.** These hyperlinks often redirect to the middle of a documentation tree. Please **do not navigate away from the target page unless instructed to do so.** All of the instructions you need will be on the hyperlinked page!

1. [Requirements](#requirements "Requirements")
2. [Express Installation - Cloud Pak for Integration 2021.1](#express-installation---cloud-pak-for-integration-20211 "Cloud Pak for Integration")
3. [Deploying Operations Dashboard](#deploying-operations-dashboard-with-cp4i "Operations Dashboard")
4. [App Connect Enterprise Deployment](#app-connect-enterprise-deployment "App Connect Enterprise")
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
