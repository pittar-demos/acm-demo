# ACM Demo

## Preparation

### 1. Install ACM

Log in to OpenShift as a cluster admin and navigate to OperatorHub.  Find and install Advanced Cluster Management for Kubernetes, accepting all defaults.

Once the ACM Operator is installed, create a "MultiClusterHub" instance in the `open-cluster-management` project.  Again, accept all defaults.

### 2. Create all the Policies and Applications

You can apply each Policy and Application individually if you like, but it's easiest just to deploy them all in one shot:

```
oc apply -k acm/all
```

## What Gets Installed?

The following Policies and Appications are created in your ACM hub.

### Advanced Cluster Security for Kubernetes

**policy-advanced-cluster-security-operator:** This policy will deploy [Advanced Cluster Security for Kubernetes](https://www.redhat.com/en/technologies/cloud-computing/openshift/advanced-cluster-security-kubernetes) Central to your Hub cluster.  It also generates an *init-bundle* along with ACM *channel/subscription/placementrule* resources to copy the init-bundle to all spoke clusters automatically.

**policy-acs-securedcluster:** Deploys the *SecuredCluster* resources to all clusters - automatically joining them to the ACS Central that is deployed on the Hub cluster.

### Compliance Operator and CIS Scan

**policy-compliance-operator:** Deploys the [Compliance Operator](https://docs.openshift.com/container-platform/4.9/security/compliance_operator/compliance-operator-understanding.html) to all clusters and runs OpenShift CIS scans.

Advanced Cluster Security will pick up the scan results and report them in ACS Central.

### DevOps Tools

**policy-gitops-operator:** This policy will install the [OpenShift GitOps](https://docs.openshift.com/container-platform/4.9/cicd/gitops/understanding-openshift-gitops.html) operator *without* the default Argo CD instance in the `openshift-gitops` namespace.  The reason for this is ACM will be managing the cluster state, so the "admin" instance of Argo CD is not required.

This policy is applied to any cluster with the `devops-tools=true` label.

**developer-gitops (Application):** This "Application" will deploy a "developer" instance of Argo CD to a namespapce called `developer-gitops`.  It will be automatically configured to use OpenShift OAuth for authentication.

This Applicaiton is deployed to any cluster with both `developer-gitops=true` and `devops-tools=true` labels.

**policy-pipelines-operator:** This policy deploys [OpenShift Pipelines](https://docs.openshift.com/container-platform/4.9/cicd/pipelines/understanding-openshift-pipelines.html) to any cluster with the `devops-tools=true` label.

**policy-codeready-workspaces:** 

