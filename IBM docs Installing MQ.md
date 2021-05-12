# Deploying MQ with CP4I
Tracing is required to deploy MQ with CP4I. To enable tracing, first deploy an ``Operations Dashboard`` (OD) capability

[Installing Operations Dashboard](https://www.ibm.com/docs/en/cloud-paks/cp-integration/2021.1?topic=configuration-installation)


### Deploy an MQ Queue Manager using CP4I Platform Navigator
https://www.ibm.com/docs/en/ibm-mq/9.2?topic=dqmorhocpc-deploying-queue-manager-using-cloud-pak-integration-platform-navigator
Once deployed, an ``Operations Dashboard Service Binding`` is needed to register the Queue Manager for tracing with the OD

### Registering a QueueManager for Tracing with Operations Dashboard
Similar steps can be followed for registering API Connect, App Connect Enterprise, or other capabilities with the Operations Dashboard
https://www.ibm.com/docs/en/cloud-paks/cp-integration/2021.1?topic=configuration-capability-registration
