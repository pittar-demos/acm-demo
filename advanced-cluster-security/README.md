# Advanced Cluster Security

This folder includes two policies as well as a manual step.  These are:

1. ACS Central Policy - Deploy ACS Operator and Central instance.
2. Manual step (using `roxctl` cli) to create the initial certificate bundle.
3. ACS SecuredCluster Policy - Join all clusters to ACS Central automatically.

## Deploy ACS Operator and Central on Hub Cluster

Apply the contents of the `acs-central-policy` directory to create an ACM policy that will deploy the ACS operator and an instance of Central in the hub cluster.

```
oc apply -f acs-central-policy
```

In a minute or two, the policy will appear in the **Governance** area of ACM.  It will then roll the operator and instance out to the hub cluster.  Once it has been deployed in the `stackrox` namespace on the hub cluster, move on to the next step.

## Generate the ACS Init Bundle

This step will require to you be logged into your hub cluster with the `oc` command line tool as a cluster admin with access to the `roxctl` command line tool.

### Download roxctl

To login to the ACS Central, you will need the generated admin password.  You can find this in the `stackrox` namespace in the `central-htpasswd` secret.  Login to Central with the username "admin" and the password from the secret.

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

## Deploy ACS SecuredCluster to All Clusters

Run the following command to create the "secured cluster" policy that will join all spoke clusters to Central.

```
oc apply -f acs-securedcluster-policy
```

Once the policy is deployed, ACM will start rolling out "SecuredCluster" configuration to all spoke clusters.  A few minutes after that is done, you should notice all of your clusters reporting in the main ACS Central dashboard.