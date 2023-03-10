# Table of Contents

1. [Intro](#intro)
1. [Role - Info Security Engineer](#role---info-security-engineer)
    1. [Use Case - Policy Management](#use-case---policy-management)
    1. [Use Case - Compliance Reporting and Remediation](#use-case---compliance-reporting-and-remediation)
    1. [Use Case - Create a new policy](#use-case---create-a-new-policy)
    1. [Use Case - Vulnerability Management](#use-case---vulnerability-management)
    1. [Use Case - Report generation of the vulnerablilities](#use-case---report-generation-of-the-vulnerablilities)
    1. [Use Case - Runtime violations](#use-case---runtime-violations)
    1. [Use Case - Risk Profiling](#use-case---risk-profiling)
    1. [Use Case - Add a custom policy based on Risk Identified](#use-case---add-a-custom-policy-based-on-risk-identified)
    1. [Use Case - Global Search](#use-case---global-search)
    1. [Use Case - Network Segmentation and Graph](#use-case---network-segmentation-and-graph)
1. [Role - DevSecOps Engineer](#role---devsecops-engineer)
    1. [Use Case - Pipeline for Image Scan & Image Check](#use-case---pipeline-for-image-scan--image-check)
    1. [Use Case - Pipeline to check the deployment](#use-case---pipeline-to-check-the-deployment)
    1. [Use Case - Integrate with client CI system](#use-case---integrate-with-client-ci-system)
    1. [Use Case: admission.stackrox.io/break-glass:jira-3423](#use-case-admissionstackroxiobreak-glassjira-3423)
1. [Role - Developer](#role---developer)
    1. [Use Case - Scan the code in the IDE](#use-case---scan-the-code-in-the-ide)
    1. [Use Case - Scan the code with KubeLinter](#use-case---scan-the-code-with-kubelinter)
1. [Role - OpenShift/Kubernetes Platform Engineer](#role---openshiftkubernetes-platform-engineer)
    1. [Use Case - Add cluster to the ACS](#use-case---add-cluster-to-the-acs)
    1. [Use Case - Integrate enterprise registries + scanner](#use-case---integrate-enterprise-registries--scanner)
    1. [Use Case - Integrate with notification systems](#use-case---integrate-with-notification-systems)
        1. [Use Case - Integrate ACS with slack](#use-case---integrate-acs-with-slack)
    1. [Use Case - Backup integration](#use-case---backup-integration)
    1. [Use Case - Third Party automation platform integration via stackrox API](#use-case---third-party-automation-platform-integration-via-stackrox-api)
    1. [Use Case - Signature Integration](#use-case---signature-integration)
    1. [Use Case - Support for online and offline modes](#use-case---support-for-online-and-offline-modes)

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
* Optional: Upgrade the OCP cluster to latest (4.11.z)
* Install web terminal operator on both OCP 4 clusters

## Role - Info Security Engineer

### Use Case - Policy Management

* Examine CIS Docker Benchmark Policies in ACS
  * Navigate to `Platform Configuration` -> "Policy Management". Add filter "Policy:" "Docker CIS" to see the policies contributing to Docker CIS standards.
  * Walk through the interface. Edit the policy and show all the steps in detail.
  * Navigate to the `Violations` page. Apply the filter "Policy": "CIS Docker" OR "Policy": "Latest".
  * Exclude the violation from the policy.
  * Navigate to `Platform Configuration` -> "Policy Management" and find the policy. Edit and show how easily the policy can be included back.
* Configure Policy in ACS to Invoke Compliance related Controls
  * Inspect the NIST 800-190 Guidance for `Control 4.2.2`
  * Observe `Control 4.2.2` being `0%`. Explain why!
  * Enforce Policies that Meet Guidance for `NIST Control 4.2.2`
    * Activate two separate default system policies that, together, meet this control's guidance
    * `90-day Image Age` and `Latest tag`
    * Come back to the compliance page to see the enforcements reflecting in the %age.
* Examine Compliance Reports for Non-OpenShift Kubernetes Clusters. __*[TODO] Works for Kubernetes clusters like GKE, AKS, and EKS. DEMO DOESN'T COVER.*__
  * Navigate back to the ACS Compliance page
  * In the section labeled "PASSING STANDARDS ACROSS CLUSTERS", click on CIS Docker.
  * Many of the controls in CIS Docker refer to the configuration of the Docker engine on each Kubernetes nodes, but a significant number of CIS Docker controls are best practices for building and using containers, and ACS has policies to enforce their use.

[Top](#table-of-contents)

----------

### Use Case - Compliance Reporting and Remediation

Refer to the [document from BU](https://docs.google.com/document/d/1I88LUsDwviixecXSFL4yq7oZOgUnlOESqf6Qmt5karM/edit#heading=h.8xj75c9dlssy). Summary of steps are below:

* Install compliance operator on the secured OCP 4 cluster
* Create a [ScanSettingBinding](https://github.com/srcporter/acs-examples/blob/main/sscan.yaml) to run a Compliance Scan on OCP 4 cluster

```yml
#cat sscan.yaml
apiVersion: compliance.openshift.io/v1alpha1
kind: ScanSettingBinding
metadata:
  name: cis-compliance
  namespace: openshift-compliance
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
* Examine Compliance Results for Workloads in ACS
  * Click on "Compliance" from left side menu. "ocp4-cis" and "ocp4-cis-node" should appear in the standards.
  * click ocp4-cis to see all the controls.
* In case the reports are not generated then edit the CronJob `cis-compliance-rerunner` and make the schedule as `*/5 */1 * * *` which translate to `At every 5th minute past every hour`. Alternatively you can add your [own expression](https://crontab.guru/#*/5_*/1_*_*_*).

[Top](#table-of-contents)

----------

### Use Case - Create a new policy

Create a new policy to mandate the deployment objects to have Change Request Id.

* Navigate to `Policy Management` -> `Policies`. Filter based on "Policy:" "deployment"
* Select "Deployment should have at least one ingress Network Policy" and clone it.
* Give it a name `Deployments should have Change Request annotation` add a new category as `Operations Best Practices`.
* add Rationale as `The release management and security team needs to have the traceability of the application's change management in a multi-tenant environemnt.`
* add Guidance as `Add cr-id annotation to the deployment manifest. E.g. "cr-id": "CR-12345"`.
* Choose `Deploy` as lifecycle stage. Keep "Response method" as "Inform"
* After saving the policy, enable the policy.
* Goto the violations page and apply the filter "namespace:" "microservices-demo" and "deployment:" "adservice"
* observe the vilation related to the email.
* noe edit the deployment manifest and add the label and see the violation being gone.

[Top](#table-of-contents)

----------

### Use Case - Vulnerability Management

* Navigate to Vulnerability Management home page. Walk through the `Dashboard` of VM page.
* There are different ways in which security team may need to discover vulnerablities. Walk through the Top bar. Different perspective can be selected by clicking `All Entities`.
* __`CVES` focused__:  Click in the CVEs and discover the details on it. There is a Red Hat tool as well to check and CVE detail i.e. [CVE Checker Labs](https://access.redhat.com/labs/cvechecker/).
* __`Images` focused__: Shows the image source, top cvss, image status etc as key fields. Click on `gcr.io/rox-se/struts-violations/mastercard-processor:latest` image and discover that OS is publishing any security patches anymore. Look at the `Dockerfile` and `Image Findings`. certain CVEs can be deferred or tagged as False Positive. Filter the CVE for `CVE-2013-0248` and click on the component. It should which `war` file has the `vulnerable jar` file.
* __`Policies` focused__: Shows the policies that track the vulnerablilities. E.g. `Fixable CVSS >= 7`.
* __`Cluster` focused__:
* __`Namespace` focused__:
* __`Deployment` focused__: Choose Deployment `mastercard-processor` -> apply filter "CVES:" "CVE-2013-0248" -> Components -> "common_fileupload". Check the same CVE on [CVE Checker Labs](https://access.redhat.com/labs/cvechecker/).
* __`Compoenents` focused__:

[Top](#table-of-contents)

----------

### Use Case - Report generation of the vulnerablilities

* Walk through the report generation process.

[Top](#table-of-contents)

----------

### Use Case - Runtime violations

* Navigate to Violations page.
* Choose a Deployment with filters "deployment:" "visa". Look for policy violation of `Kubernetes Actions: Exec into Pod` and `Unauthorized Process Execution`
* if you do not see any pod running then please edit the deployment object and delete "securityContent" section

```yml
securityContext:
  runAsUser: 1000
  runAsGroup: 1000
  runAsNonRoot: true
  fsGroup: 1000

```

* Choose `Mark as resolved` for `Kubernetes Actions: Exec into Pod` and `Unauthorized Process Execution`.
* Wait for a while the pod is coded to trigger few commands that will raise `Unauthorized Process Execution` violation. Once the violation occurs; navigate to "Risk -> visa-processor -> Risk Indicator -> Suspicious Process Execution" and "Process Discovery"
* To simulate `Kubernetes Actions: Exec into Pod`; Create a dummy file via oc exec command

```bash
oc exec visa-processor-586d8b989d-l5lhx -n payments -- df

oc exec visa-processor-586d8b989d-l5lhx -c visa-processor -n payments -- curl -O https://github.com/redhat-apac-stp/acs-demo/blob/main/README.md 

oc exec visa-processor-586d8b989d-l5lhx -c visa-processor -n payments -- head README.md

```

* Click on the violation and walk through the `Violation Events`

[Top](#table-of-contents)

----------

### Use Case - Risk Profiling

* Navigate to Risk view where we go beyond the basics of vulnerabilities to understand how `deployment` configuration and `runtime activity` impact the likelihood of an exploit occurring and how successful those exploits will be.
* Observe the priority. These priority is not static. Let's see an example.
* Apply the filter "Deployment:" "currency". See the `priority rank 150`. Click and explore further. You will observfe that there no process active. If there are suspicious processes running that should change the Risk associated with the deployment.
* Make the Pods running by deleting the SCC from the deployment manifest. Take a terminal to the pod and run few admin priviledged commands like chmod.
* Check the new `priority rank 97`
* Now run the following command in the same pod

```bash
while true ;do  netstat; done
```

* Check the `new priority rank of 53` and a `red dot` at the start of the row.

[Top](#table-of-contents)

----------

### Use Case - Add a custom policy based on Risk Identified

Security engineer doesn't want users to execute sudo in the pod/container

* Navigate to Risk tab. Apply the filter as `processName:` `sudo`.
* Observe the deployment `visa-processor` already having the violation.
* press on the `create policy` on the top right corner.
* fill the form. Note that run time behaviour is already filled. The policy can also be imported from the file `StackRox_Exported_Policies-11_23_2022.json` from the resources folder.
* To simulate the violation; get a terminal with the visa processor pod and run below commands:

```bash
sudo -i
apt-get update
```

* Note the violation being generated. Navigate to Vilation tab with filter "policy:" "sudo".
* Walk through the "violation events".

[Top](#table-of-contents)

----------

### Use Case - Global Search

Global search allows the security engineer to search any security related artefact. E.g. You want to search for a common project `test` across multiple clusters.

* Create the test project and some vulnerable deployments.

```bash
oc new-project test
oc run shell --labels=app=shellshock,team=test-team --image=vulnerables/cve-2014-6271 -n test 
oc run samba --labels=app=rce --image=vulnerables/cve-2017-7494 -n test
```

* Click on the seach icon on the top right corner and demo different filters.
* Apply filter "Violation State:" "ACTIVE", "Namespace:" "test", "Policy:" "Latets". Walk through the result.

[Top](#table-of-contents)

----------

### Use Case - Network Segmentation and Graph

The Network Graph is a `flow diagram`, `firewall diagram`, and `firewall rule builder` in one.

* Zoom in on Payments Namespace. As we zoom in, the namespace boxes show the individual deployment names.
* Click on visa-processor deployment. Clicking on a deployment brings up details of the types of traffic observed including source or destination and ports.
* Show the `Flows` and `Network Policies` on the ribbon. Expand the ribbon.
* Walk through the `Network Flow` and `Baseline Settings`

[Top](#table-of-contents)

----------

## Role - DevSecOps Engineer

Pipelines uses `roxctl` cli to perfrom it's tasks. Documentation is available [here](https://docs.openshift.com/acs/3.72/cli/getting-started-cli.html).

### Use Case - Pipeline for Image Scan & Image Check

>
> 1. Edit Pipeline `rox-pipeline` and make tasks -> params -> value=json (for name=output_format).
>
> 2. Make sure the `scanner` and `scanner_db` are running.
>
1. Show the image scan and image check against mongodb image.
    1. No vul -> quay.io/mongodb/mongodb-enterprise-database:2.0.2
    1. Old image with vul -> quay.io/mongodb/mongodb-enterprise-database:0.1
    1. Then switch off the policy "Policy": "Fixable Severity at least Important" and re-run the previous step and check the logs.
    1. Switch on the policy back again.
1. Show the image scan with `Java` and `Node` packages.
    1. Scan `quay.io/rajivranjan/tasks-app:latest` and navigate to component `ansi-regex 3.0.0`.
    1. Navigate to Vulnerability Management -> Images -> filter for `quay.io/rajivranjan/tasks-app:latest`. Look for the `Source` column. It has `OS`, `Java` and `Node` vulnerablities.

[Top](#table-of-contents)

----------

### Use Case - Pipeline to check the deployment

1. Show the existing pipeline to scan the deployment manifest.

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

[Top](#table-of-contents)

----------

### Use Case - Integrate with client CI system

Client CI system can integrate with stackrox via roxctl.

[Top](#table-of-contents)

----------

### Use Case: admission.stackrox.io/break-glass:jira-3423

[Top](#table-of-contents)

----------

## Role - Developer

### Use Case - Scan the code in the IDE

* Install plugin from the [marketplace](https://marketplace.visualstudio.com/items?itemName=redhat.fabric8-analytics)
* Walk through the sample code and it's analysis.

[Top](#table-of-contents)

----------

### Use Case - Scan the code with KubeLinter

* Integrate kube-linter plugin for stackrox. [Github Link](https://github.com/stackrox/kube-linter) and [documentation link](https://docs.kubelinter.io/#/).
* There are 47 rules already baked in.

```bash
$ kube-linter checks list | grep -i "Name: "

Name: access-to-create-pods
Name: access-to-secrets
Name: cluster-admin-role-binding
Name: dangling-horizontalpodautoscaler
Name: dangling-ingress
Name: dangling-networkpolicy
Name: dangling-networkpolicypeer-podselector
Name: dangling-service
Name: default-service-account
Name: deprecated-service-account-field
Name: dnsconfig-options
Name: docker-sock
Name: drop-net-raw-capability
Name: env-var-secret
Name: exposed-services
Name: host-ipc
Name: host-network
Name: host-pid
Name: hpa-minimum-three-replicas
Name: invalid-target-ports
Name: latest-tag
Name: minimum-three-replicas
Name: mismatching-selector
Name: no-anti-affinity
Name: no-extensions-v1beta
Name: no-liveness-probe
Name: no-node-affinity
Name: no-read-only-root-fs
Name: no-readiness-probe
Name: no-rolling-update-strategy
Name: non-existent-service-account
Name: non-isolated-pod
Name: privilege-escalation-container
Name: privileged-container
Name: privileged-ports
Name: read-secret-from-env-var
Name: required-annotation-email
Name: required-label-owner
Name: run-as-non-root
Name: sensitive-host-mounts
Name: ssh-port
Name: unsafe-proc-mount
Name: unsafe-sysctls
Name: unset-cpu-requirements
Name: unset-memory-requirements
Name: use-namespace
Name: wildcard-in-rules
Name: writable-host-mount
```

* run the lint on a sample k8s manifest.

```bash
# run from the folder ~/github/rajiv-ranjan/acs-demo
kube-linter lint resource/pod.yaml
```

[Top](#table-of-contents)

----------

## Role - OpenShift/Kubernetes Platform Engineer

### Use Case - Add cluster to the ACS

* Follow the steps documented [here](https://docs.openshift.com/acs/3.72/installing/install-ocp-operator.html#install-secured-cluster-operator_install-ocp-operator).
* After added adding the cluster; add the below deployment on the cluster and see if ACS scanns and picks the vulnerabilities.

```bash
oc new-project test

oc run shell --labels=app=shellshock,team=test-team --image=vulnerables/cve-2014-6271 -n test

oc run samba --labels=app=rce --image=vulnerables/cve-2017-7494 -n test

```

[Top](#table-of-contents)

----------

### Use Case - Integrate enterprise registries + scanner

You can integrate RHACS with major image registries, including:

1. Amazon Elastic Container Registry (ECR)
1. Docker Hub
1. Google Container Registry (GCR)
1. Google Artifact Registry
1. IBM Cloud Container Registry (ICR)
1. JFrog Artifactory
1. Microsoft Azure Container Registry (ACR)
1. Red Hat Quay
1. Red Hat container registries
1. Sonatype Nexus
1. Any other registry that uses the Docker Registry HTTP API

ACS can leverage Scanners like:
1. Clair
1. Google Container Analysis
1. RH Quay

Demo Integrate with docker registries
```bash
#####Deploy docker registry service###########
BASE_PATH=/opt/acs-demo
mkdir -p ${BASE_PATH}/data && mkdir -p ${BASE_PATH}/{auth,certs}
tree ${BASE_PATH}

yum -y install httpd-tools

htpasswd -bBc ${BASE_PATH}/auth/htpasswd openshift redhat

cat ${BASE_PATH}/auth/htpasswd

REG_NAME=acs-demo
REG_PORT=50002
REG_IMAGE_NAME=hub-mirror.c.163.com/library/registry:latest

docker run -d \
    -p ${REG_PORT}:5000 \
    --name reg-${REG_NAME} \
    --restart=always \
    -v ${BASE_PATH}/data:/var/lib/registry \
    -e REGISTRY_STORAGE_DELETE_ENABLED=true \
    -e REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true \
    -v ${BASE_PATH}/auth:/auth \
    -e REGISTRY_AUTH=htpasswd \
    -e REGISTRY_AUTH_HTPASSWD_REALM=basic-realm \
    -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
    ${REG_IMAGE_NAME}


curl -u openshift:redhat http://localhost:${REG_PORT}/v2/_catalog

```
![Registry-1](resource/integrate-registry-1.png)




[Top](#table-of-contents)

----------

### Use Case - Integrate with notification systems

You can integrate RHACS with notification systems like:

1. Slack: send `alerts`.
1. Microsoft Teams:
1. PagerDuty: send `alerts`.
1. Sumo Logic: send `alerts`.
1. Jira Software: send `alerts` and creates issues.
1. Email: Can send email to a `centralised team email id` or read `email` annotation from `deployment` manifest.
1. Splunk: send `alerts`.
1. Syslog: send `alerts` and `audit events` using the syslog protocol. This can feed into SIEM solutions.
1. AWS Security Hub:
1. GCP Security Command Center: send `alerts`
1. Web Hook: send `alerts` to a webhook URL over HTTP protocol and JSON message.

#### Use Case - Integrate ACS with slack

[Top](#table-of-contents)

----------

### Use Case - Backup integration

ACS needs to backup configurations, resources, events, and certificates.

____Automatic Backup____

You can integrate RHACS with notification systems like:

1. AWS S3
1. Google Cloud Storage:

____On-demand Backup____

1. Use `roxctl` to take backup

* run roxctl command. Below method is using `admin` user credentials. Same can be achieved via API token too.

```bash
roxctl -p "$ROX_ADMIN_PASSWORD"  -e "$ROX_CENTRAL_ADDRESS" central backup

WARN:	The remote endpoint failed TLS validation. This will be a fatal error in future releases.
Please do one of the following at your earliest convenience:
  1. Obtain a valid certificate for your Central instance/Load Balancer.
  2. Use the --ca option to specify a custom CA certificate (PEM format). This Certificate can be obtained by
     running "roxctl central cert".
  3. Update all your roxctl usages to pass the --insecure-skip-tls-verify option, in order to
     suppress this warning and retain the old behavior of not validating TLS certificates in
     the future (NOT RECOMMENDED).


ERROR:	Certificate validation error: x509: certificate is valid for central.stackrox, central.stackrox.svc, not central-stackrox.apps.cluster-gmlfx.gmlfx.sandbox2119.opentlc.com
14.1 MiB [---------->>>>>---------->>>>>---------->>>>>---------->>>>>---------->>>>>---------->>>>>---------->>>>>-------------] 1.1 MiB/s stackrox_db_2022_11_25_10_01_05.zip
Wrote backup file to "stackrox_db_2022_11_25_10_01_05.zip"
```

* Use above method to get the backup and can push zip file to `ODF S3` bucket.

[Top](#table-of-contents)

----------

### Use Case - Third Party automation platform integration via stackrox API

[Top](#table-of-contents)

----------

### Use Case - Signature Integration
Currently ACS only support integrate with cosign
Here, we use cosign to demo the signature integration

Intall cosign with binary
```bash
wget "https://github.com/sigstore/cosign/releases/download/v1.6.0/cosign-linux-amd64"
mv cosign-linux-amd64 /usr/local/bin/cosign
chmod +x /usr/local/bin/cosign
```
Generate a keypair
```bash
[root@registry acs-demo]# cosign generate-key-pair
Enter password for private key: 
Enter password for private key again: 
Private key written to cosign.key
Public key written to cosign.pub
```

Sign a container and store the signature in the registry
```bash
```




[Top](#table-of-contents)

----------

### Use Case - Support for online and offline modes

[Documentation](https://docs.openshift.com/acs/3.72/configuration/enable-offline-mode.html)

[Top](#table-of-contents)

----------
