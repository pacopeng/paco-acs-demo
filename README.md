# Table of Contents

1. [Intro](#intro)
2. [Role - Info Security Engineer](#role-info-security-engineer)
3. [Third Example](#third-example)
4. [Fourth Example](#fourth-examplehttpwwwfourthexamplecom)

## Intro
Demo is an adaptation from original [google docs](https://docs.google.com/document/d/1q_7PE0ecYTFfrAAgBLoYya9V5VmzPP4ZsA9tUHKit28/edit?pli=1#). Perform following tasks:

* Provision the pre-deployed ACS cluster from [RHPDS](https://rhpds.redhat.com/catalog/explorer) -> Multi Product Demo -> Openshift 4 Advanced Cluster Security 3.

> Tips:
>
> 1. Please note the provisioing can take upto an hour. If the cluster is already provisioned then "Start" the service. Typically it takes 5 to 10 mins.
> 2. Post provisioning make sure the lifetime and runtime are extended. This is to avoid ACS going down in the middle of the demo.
>

* Upgrade the ACS cluster to latest (3.72.1). Perform the upgrade from 3.70 -> 3.71 -> 3.72
* Check scanner and scanner-db pods are running in `stackrox` namespace. If not then delete the scanner-db pod and then scanner. You need to do this everytime the service is started on RHPDS.
* Optional: Upgrade the OCP cluster to latest (4.10.z)

## Role - Info Security Engineer

### Use Case - Policy Management

### Use Case - Compliance Reporting and Remediation

Refer to the [document from BU](https://docs.google.com/document/d/1I88LUsDwviixecXSFL4yq7oZOgUnlOESqf6Qmt5karM/edit#heading=h.8xj75c9dlssy)

* Install compliance operator on the secured OCP 4 cluster
* Create a [ScanSettingBinding](https://github.com/srcporter/acs-examples/blob/main/sscan.yaml) to run a Compliance Scan on OCP 4 cluster

```yml
#cat sscan.yaml
apiVersion: compliance.openshift.io/v1alpha1
kind: ScanSettingBinding
metadata:
  name: cis-compliance
profiles:
  - name: ocp4-cis-node
    kind: Profile
    apiGroup: compliance.openshift.io/v1alpha1
  - name: ocp4-cis
    kind: Profile
    apiGroup: compliance.openshift.io/v1alpha1
settingsRef:
  name: default
  kind: ScanSetting
  apiGroup: compliance.openshift.io/v1alpha1

```

```bash
oc -n openshift-compliance create -f sscan.yaml
```

* Restart the sensor by deleting the sensor pod.

```bash
oc -n stackrox delete pod -lapp=sensor
```

* Kick off a compliance scan in ACS
* Click on "Compliance" from left side menu. "ocp4-cis" and "ocp4-cis-node" should appear in the standards.
* click ocp4-cis to see all the controls.
* 

## Role - DevSecOps Engineer

## Role - Developer


1. SECURITY - Show a policy and edit the policy and show how it can be customized. Platform Configuration -> Policy Management. Use "Policy": "Fixable Severity at least Important"
1. DEVOPS - Teckton Pipeline trigger integration with stackrox. Show the image scan and image check against mongodb image. 
    1. No vul -> quay.io/mongodb/mongodb-enterprise-database:2.0.2
    1. Old image with vul -> quay.io/mongodb/mongodb-enterprise-database:0.1
    1. Then switch off the policy "Policy": "Fixable Severity at least Important" and re-run the previous step and check the logs.
    1. Switch on the policy back again.
1. SECURITY - security team can translate there control in the product. Like add the email id as mandatory annotation in the deployment.
    1. Platform Configuration -> Policy Management. Use "Policy": "email"
1. SECURITY - See the 
    1. filter "Images": "mongo"
1. SECURITY/DEVOPS - Integrate stackrox with slack
1. 
1. DEVELOPER - Integrate kube-linter plugin for stackrox. [Github Link](https://github.com/stackrox/kube-linter)
1. DEVOPS - 
1. 

```bash
# base64 encoded yaml to be used to pass to the pipeline demo. It's a busybox image.
YXBpVmVyc2lvbjogYXBwcy92MQpraW5kOiBEZXBsb3ltZW50Cm1ldGFkYXRhOgogIGxhYmVsczoKICAgIGFwcDogc2ltcGxlLWRlcGxveW1lbnQKICAgIGFwcC5rdWJlcm5ldGVzLmlvL2NvbXBvbmVudDogc2ltcGxlLWRlcGxveW1lbnQKICAgIGFwcC5rdWJlcm5ldGVzLmlvL2luc3RhbmNlOiBzaW1wbGUtZGVwbG95bWVudAogICAgYXBwLmt1YmVybmV0ZXMuaW8vcGFydC1vZjogc2ltcGxlLWRlcGxveW1lbnQKICAgIGFwcC5vcGVuc2hpZnQuaW8vcnVudGltZTogcmVkaGF0CiAgbmFtZTogc2ltcGxlLWRlcGxveW1lbnQKc3BlYzoKICByZXBsaWNhczogMQogIHNlbGVjdG9yOgogICAgbWF0Y2hMYWJlbHM6CiAgICAgIGFwcDogc2ltcGxlLWRlcGxveW1lbnQKICAgIHR5cGU6IFJlY3JlYXRlCiAgdGVtcGxhdGU6CiAgICBtZXRhZGF0YToKICAgICAgbGFiZWxzOgogICAgICAgIGFwcDogc2ltcGxlLWRlcGxveW1lbnQKICAgICAgICBkZXBsb3ltZW50Y29uZmlnOiBzaW1wbGUtZGVwbG95bWVudAogICAgc3BlYzoKICAgICAgY29udGFpbmVyczoKICAgICAgLSBpbWFnZTogcmVnaXN0cnkuYWNjZXNzLnJlZGhhdC5jb20vdWJpOC91Ymk6bGF0ZXN0CiAgICAgICAgaW1hZ2VQdWxsUG9saWN5OiBBbHdheXMKICAgICAgICBuYW1lOiBzaW1wbGUtZGVwbG95bWVudAogICAgICAgIGNvbW1hbmQ6CiAgICAgICAgLSAvYmluL3NoCiAgICAgICAgLSAtYwogICAgICAgIC0gfAogICAgICAgICAgc2xlZXAgaW5maW5pdHkKICAgICAgICByZXNvdXJjZXM6IHt9Cg==
```

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: simple-deployment
    app.kubernetes.io/component: simple-deployment
    app.kubernetes.io/instance: simple-deployment
    app.kubernetes.io/part-of: simple-deployment
    app.openshift.io/runtime: redhat
  name: simple-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: simple-deployment
    type: Recreate
  template:
    metadata:
      labels:
        app: simple-deployment
        deploymentconfig: simple-deployment
    spec:
      containers:
      - image: registry.access.redhat.com/ubi8/ubi:latest
        imagePullPolicy: Always
        name: simple-deployment
        command:
        - /bin/sh
        - -c
        - |
          sleep infinity
        resources: {}
```

```bash
# It's a java based app image.
LS0tCmFwaVZlcnNpb246IGFwcHMvdjEKa2luZDogRGVwbG95bWVudAptZXRhZGF0YToKICBuYW1lOiBqYXItbXVsdGktc3RhZ2UtZXhhbXBsZQpzcGVjOgogIHJlcGxpY2FzOiAxCiAgc3RyYXRlZ3k6CiAgICB0eXBlOiBSZWNyZWF0ZQogIHNlbGVjdG9yOgogICAgbWF0Y2hMYWJlbHM6CiAgICAgIGFwcDogamFyLW11bHRpLXN0YWdlCiAgdGVtcGxhdGU6CiAgICBtZXRhZGF0YToKICAgICAgbGFiZWxzOgogICAgICAgIGFwcDogamFyLW11bHRpLXN0YWdlCiAgICBzcGVjOgogICAgICBjb250YWluZXJzOgogICAgICAtIG5hbWU6IGFwcAogICAgICAgIGltYWdlOiBxdWF5LmlvL29wZW5zaGlmdC1leGFtcGxlcy9qYXItZGVwbG95LWV4YW1wbGU6bXVsdGktc3RhZ2UKICB0cmlnZ2VyczoKICAtIHR5cGU6IENvbmZpZ0NoYW5nZQotLS0KYXBpVmVyc2lvbjogdjEKa2luZDogU2VydmljZQptZXRhZGF0YToKICBsYWJlbHM6CiAgICBhcHA6IGphci1tdWx0aS1zdGFnZQogIG5hbWU6IGphci1tdWx0aS1zdGFnZQpzcGVjOgogIHBvcnRzOgogIC0gbmFtZTogODA4MC04MDgwCiAgICBwb3J0OiA4MDgwCiAgICBwcm90b2NvbDogVENQCiAgICB0YXJnZXRQb3J0OiA4MDgwCiAgc2VsZWN0b3I6CiAgICBhcHA6IGphci1tdWx0aS1zdGFnZQogIHNlc3Npb25BZmZpbml0eTogTm9uZQogIHR5cGU6IENsdXN0ZXJJUAotLS0KYXBpVmVyc2lvbjogcm91dGUub3BlbnNoaWZ0LmlvL3YxCmtpbmQ6IFJvdXRlCm1ldGFkYXRhOgogIGxhYmVsczoKICAgIGFwcDogamFyLW11bHRpLXN0YWdlCiAgbmFtZTogamFyLW11bHRpLXN0YWdlCnNwZWM6CiAgcG9ydDoKICAgIHRhcmdldFBvcnQ6IDgwODAKICB0bzoKICAgIGtpbmQ6ICJTZXJ2aWNlIgogICAgbmFtZTogamFyLW11bHRpLXN0YWdlCiAgICB3ZWlnaHQ6IG51bGwKCg==
```

```yml
# link https://examples.openshift.pub/deploy/jar/
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jar-multi-stage-example
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: jar-multi-stage
  template:
    metadata:
      labels:
        app: jar-multi-stage
    spec:
      containers:
      - name: app
        image: quay.io/openshift-examples/jar-deploy-example:multi-stage
  triggers:
  - type: ConfigChange
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: jar-multi-stage
  name: jar-multi-stage
spec:
  ports:
  - name: 8080-8080
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: jar-multi-stage
  sessionAffinity: None
  type: ClusterIP
---
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  labels:
    app: jar-multi-stage
  name: jar-multi-stage
spec:
  port:
    targetPort: 8080
  to:
    kind: "Service"
    name: jar-multi-stage
    weight: null
```
