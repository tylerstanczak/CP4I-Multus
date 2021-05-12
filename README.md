# Table of Contents
1. [Requirements](#requirements "Requirements")
2. [Platform Navigator](Platform%20Navigator.md)
3. [Configure Multus](#configure-multus "Configure Multus")
4. [App Connect Enterprise](#connect-the-2nd-network-to-app-connect-enterprise-ace "App Connect Enterprise")
5. [MQ](#connect-the-2nd-network-to-ibm-mq "IBM MQ")

### Requirements

- OpenShift 4.6.12 or later [OpenShift 4.6 Installation Docs](https://docs.openshift.com/container-platform/4.6/welcome/index.html)
- Cloud Pak for Integration 2021.1 [Cloud Pak for Integration Installation Docs](https://www.ibm.com/docs/en/cloud-paks/cp-integration/2021.1?topic=installing)
  ```diff
  - NOTICE During Installation, name your IBM Entitled Registry Key Secret:
  + "ibm-entitlement-key"
  - as some CP4I capabilities such as MQ require an IBM entitled registry key with this name
  ```
- Highly Recommended: Deploy [Platform Navigator](Platform%20Navigator.md) & [Operations Dashboard](IBM%20docs%20Installing%20MQ.md) before deploying additional CP4I capabilities
