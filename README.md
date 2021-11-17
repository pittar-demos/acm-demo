# ACM Demo

## 1. Install ACM

Log in to OpenShift as a cluster admin and navigate to OperatorHub.  Find and install Advanced Cluster Management for Kubernetes, accepting all defaults.

Once the ACM Operator is installed, create a "MultiClusterHub" instance in the `open-cluster-management` project.  Again, accept all defaults.

## 2. Install Advanced Cluster Security for Kubernetes (ACS)

Log in to OpenShift as a cluster admin using the `oc` cli.  Apply a policy that will instruct ACM to install ACS Central in the "hub" cluster.  YOu should see this policy in the "Governance" portion of ACM.  The policy might report as "Not compliant" until the operator has finished installing so that the `Central` CRD will be available in the cluster.

```
oc apply -k  acm/advanced-cluster-security/acs-central-policy
```

After a few minutes, you should see that ACS is installed in the hub cluster.

## 3. Deploy ACS "SecuredCluster" Sensors to All Clusters

Run the following command to create the "SecuredCluster" policy that will join all spoke clusters to Central.

```
oc apply -k acm/advanced-cluster-security/acs-securedcluster-policy
```

Once the policy is deployed, ACM will start rolling out "SecuredCluster" configuration to all spoke clusters.  A few minutes after that is done, you should notice all of your clusters reporting in the main ACS Central dashboard.

## 5. Deploy All Other Policies / Apps

```
oc apply -k acm/all
```