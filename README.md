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

## 3. Manually create ACS init-bundle.

### Download roxctl

To login to the ACS Central you will need the generated admin password.  You can find this in the `stackrox` namespace in the `central-htpasswd` secret.  Login to Central with the username "admin" and the password from the secret.

Once logged in, you can download the `roxctl` cli by clicking on the "CLI" icon at the top-right of the screen.

### Create an Admin Token

While logged into Central, navigate to "Platform Configuration" in the side navigation and select "Integrations".  Next, scroll to the bottom of the screen and select the "StackRox API Token" tile.

Select "Generate Token", then:
* Give your token a name (e.g. init-bundle-token)
* Select the "Admin" role.
* Generate the token.

Copy the token and assign it to an enviornment variable on your machne called "ROX_API_TOKEN".  For example:

```
export ROX_API_TOKEN=<token>
```

### Generate the Init Bundle

Generate the init-bundle by executing the following command.  It will use the `ROX_API_TOKEN` env var for authentication.  You should also be logged in to the cluster as a `cluster-admin` with the oc cli.

```
curl https://raw.githubusercontent.com/open-cluster-management/advanced-cluster-security/main/scripts/deploy-bundle.sh -o deploy-bundle.sh

bash deploy-bundle.sh -i bundle.yaml | oc apply -f -
```

## 4. Deploy ACS "SecuredCluster" Sensors to All Clusters

Run the following command to create the "SecuredCluster" policy that will join all spoke clusters to Central.

```
oc apply -k acm/advanced-cluster-security/acs-securedcluster-policy
```

Once the policy is deployed, ACM will start rolling out "SecuredCluster" configuration to all spoke clusters.  A few minutes after that is done, you should notice all of your clusters reporting in the main ACS Central dashboard.

## 5. Deploy All Other Policies / Apps

```
oc apply -k acm/all
```