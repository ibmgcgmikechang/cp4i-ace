# Test deployment

Example of ACE deployment using a bar file located in a git.
ACE needs to be configured to retrieve this bar file since the repo is TLS and might needs authentication.

HTTP(S) server used to host bar files are accessed by integration server using a "barauth" configuration, which is a ACE CR.

## BarAuth configuration

Create a json file with:
```json
{"authType":"BASIC_AUTH","credentials":{"username":"","password":"","insecureSsl":"true"}}
```
If basic authentication is required, the user and password can be provided.
For the pilot we will accept insecureSSL but it is possible to add ca cert as well.

Create a secret using the file:
```sh
oc create secret generic $secret-name --from-file=configuration=$json-file --namespace=$ace-namespace
```
To create the barauth configuration, update the following ace configuration cr and apply it on your namespace

```yaml
apiVersion: appconnect.ibm.com/v1beta1
kind: Configuration
metadata:
  name: $barauth-name
  namespace: $ace-namespace
spec:
  description: configuration to access git repo where bars are located
  secretName: $secret-name
  type: barauth
  version: 12.0.5.0-r2
```
## Deploy IntegrationServer

create a ACE IntegrationServer CR with the following template:

```yaml
apiVersion: appconnect.ibm.com/v1beta1
kind: IntegrationServer
metadata:
  name: ace-countryapi-sample
  namespace: $ace-namespace
spec:
  adminServerSecure: true
  barURL: https://eu-de.git.cloud.ibm.com/integration-ts-pub/arch-repository/-/raw/main/ace/restapi-sample.bar
  configurations:
  - $barauth-name
  createDashboardUsers: false
  designerFlowsOperationMode: disabled
  enableMetrics: true
  license:
    accept: true
    license: L-APEH-CCHL5W
    use: CloudPakForIntegrationNonProduction
  pod:
    containers:
      runtime:
        resources:
          limits:
            cpu: 300m
            memory: 368Mi
          requests:
            cpu: 300m
            memory: 368Mi
  replicas: 1
  router:
    timeout: 120s
  service:
    endpointType: http
  version: "12.0"
```



