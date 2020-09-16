# operator-example-with-everything
## Datadog Operator Example that Includes Kubernetes, Docker, Logs, Process, NPM, APM, Control Plane, and OpenShift Metrics

This guide explains the installation process behind configuring every possible integration you may want to configure with the Operator. Included in this repository are instructions on how to deploy a local OpenShift cluster (v4+) as well as configure the Datadog agent along with all relevant integrations. 

Inspiration came from [Ryan Hennessy's guide that you can find here](https://github.com/ryhennessy/datadog-operator-openshift-example). 

### Configuring `crc` to work on your local machine

1. Navigate to [Red Hat's CodeReady Containers product page](https://cloud.redhat.com/openshift/install/crc/installer-provisioned?intcmp=7013a000002CtetAAC). You will need to create a Red Hat account in order to access the page. 

2. Once logged in, download the CodeReady Containers tarball archive. Keep this page open, because you'll need the pull secret later on. 

![download_crc](images/crc_download.png)

3. Extract the archive and place the contents in a location that you can easily add to your `$PATH`. I extracted to `/usr/local/share/` using the following command. 

```
[morgan.lupton@mycomputer:~/Downloads]$ tar -xvf crc-macos-amd64.tar.xz -C /usr/local/share/
```

4. Add the path to the `crc` executable to your `$PATH` env var. Please keep in mind that the path name will depend on the version of crc you downloaded (in this case 1.14.0). 

```
[morgan.lupton@mycomputer:~]$ export PATH="/usr/local/share/crc-macos-1.14.0-amd64/:$PATH"
```

5. Make sure to add this statement to your bash profile such that it persists after you exit out of the terminal. 

```
[morgan.lupton@mycomputer:~]$ vim ~/.bash_profile
```

Simply add this line to the bottom of the file. 
```
...
export PATH="/usr/local/share/crc-macos-1.14.0-amd64/:$PATH"
```

6. Run the `crc setup` command. You will be prompted for your password. 

```
[morgan.lupton@COMP10906:~]$ crc setup
INFO Checking if oc binary is cached
INFO Caching oc binary
INFO Checking if podman remote binary is cached
INFO Checking if goodhosts binary is cached
INFO Caching goodhosts binary
INFO Will use root access: change ownership of /Users/morgan.lupton/.crc/bin/goodhosts
Password:
INFO Will use root access: set suid for /Users/morgan.lupton/.crc/bin/goodhosts
INFO Checking if CRC bundle is cached in '$HOME/.crc'
INFO Unpacking bundle from the CRC binary
INFO Checking minimum RAM requirements
INFO Checking if running as non-root
INFO Checking if HyperKit is installed
INFO Setting up virtualization with HyperKit
INFO Will use root access: change ownership of /Users/morgan.lupton/.crc/bin/hyperkit
INFO Will use root access: set suid for /Users/morgan.lupton/.crc/bin/hyperkit
INFO Checking if crc-driver-hyperkit is installed
INFO Installing crc-machine-hyperkit
INFO Will use root access: change ownership of /Users/morgan.lupton/.crc/bin/crc-driver-hyperkit
INFO Will use root access: set suid for /Users/morgan.lupton/.crc/bin/crc-driver-hyperkit
INFO Checking file permissions for /etc/hosts
INFO Checking file permissions for /etc/resolver/testing
Setup is complete, you can now run 'crc start' to start the OpenShift cluster
```

7. Run the `crc start` command. When prompted for a pull secret, copy and paste the pull secret from the CodeReady Containers product page that you downloaded the tarball from in step 2. 

>**Important Note**
>VirtualBox must be installed for crc to run properly. If crc is not natively using VirtualBox as its driver, you can configure that by appending the `--driver=virtualbox` flag to the `crc start` command. 



```
[morgan.lupton@COMP10906:~]$ crc start
WARN A new version (1.15.0) has been published on https://cloud.redhat.com/openshift/install/crc/installer-provisioned
INFO Checking if oc binary is cached
INFO Checking if podman remote binary is cached
INFO Checking if goodhosts binary is cached
INFO Checking minimum RAM requirements
INFO Checking if running as non-root
INFO Checking if HyperKit is installed
INFO Checking if crc-driver-hyperkit is installed
INFO Checking file permissions for /etc/hosts
INFO Checking file permissions for /etc/resolver/testing
? Image pull secret [? for help]
```

Once copy-pasted it will take 5-10 minutes to install. It's a 9.9GB file so be patient!

```
INFO Extracting bundle: crc_hyperkit_4.5.4.crcbundle ... ******************************************crc.qcow2: 1.14 GiB / 9.90 GiB [------>____________________________________________________] 11.52
```

You'll know the install is done when you see the following messages. You may see a WARN statement, but feel free to ignore it. 

```
INFO Checking if oc binary is cached
INFO Checking if podman remote binary is cached
INFO Checking if goodhosts binary is cached
INFO Checking minimum RAM requirements
INFO Checking if running as non-root
INFO Checking if HyperKit is installed
INFO Checking if crc-driver-hyperkit is installed
INFO Checking file permissions for /etc/hosts
INFO Checking file permissions for /etc/resolver/testing
INFO A CodeReady Containers VM for OpenShift 4.5.4 is already running
Started the OpenShift cluster
WARN The cluster might report a degraded or error state. This is expected since several operators have been disabled to lower the resource usage. For more information, please consult the documentation

```

8. Take note of the `kubeadmin` password that is output. **You will need this later!** You should see a message that looks like the following in the logs: 

```
...
INFO To login as an admin, run 'oc login -u kubeadmin -p <KUBE-ADMIN-PASSWORD> https://api.crc.testing:6443'
...
```

9. Validate the install completed correctly by running `crc status`.

```
[morgan.lupton@COMP10906:~]$ crc status
CRC VM:          Running
OpenShift:       Running (v4.5.4)
Disk Usage:      13.09GB of 32.72GB (Inside the CRC VM)
Cache Usage:     13.1GB
Cache Directory: /Users/morgan.lupton/.crc/cache
```

10. Finally configure the `oc-env` by running the following command. Without this, you will not be able to run `oc` commands against your cluster. 

```
[morgan.lupton@COMP10906:~]$ ]eval $(crc oc-env)
```

#### Troubleshooting `crc`

If `crc start` fails for whatever reason. I've found these steps to work almost every time. 

1. Delete the `~/.crc` directory and all its contents

```
[morgan.lupton@COMP10906:~]$ rm -r ~/.crc
```

2. Restart your computer

3. Re-run `crc setup` followed by `crc start`


### Install the Datadog Operator from the Operator Marketplace

1. Once your cluster is up and running, run `crc console` and a page should automatically open in your browser that will take you to the OpenShift Management Console

2. Once at the login page, login with the user `kubeadmin` using the password that was automatically generated when you first ran `crc start`. If you have to blow the cluster away and restart from scratch; don't worry, this password remains the same even if you completely delete it locally. 

![Login Page](images/login_page.png)

![Login Page 2](images/login_page_2.png)

3. Navigate to the OperatorHub by using the left panel. Go to `Operators > OperatorHub`.

4. Search for "Datadog" and select the first result. Make sure you don't select the Community Operator.

![Operator Page 1](images/operator_page_1.png)

5. Install the Datadog Operator in all namespaces. 

![Operator Page 2](images/operator_page_2.png)

### Install Kube State Metrics

Kube State Metrics is important to include as many Datadog metrics are reliant upon it. Despite the fact that OpenShift already has a Kube State Metrics server, this does not appear to be compatible with Datadog. 

1. Navigate to the `kube-state-metrics` folder in this repository. 

```
[morgan.lupton@COMP10906:~]$ cd /PATH/TO/operator-example-with-everything/kube-state-metrics
```

2. Login as `kubeadmin` with `oc login`. You will be prompted to provide a username and password.

```
[morgan.lupton@COMP10906:/PATH/TO/kube-state-metrics]$ oc login
Authentication required for https://api.crc.testing:6443 (openshift)
Username: kubeadmin
Password:
Login successful.

You have access to 57 projects, the list has been suppressed. You can list all projects with 'oc projects'
```

3. Run the following command to deploy kube-state-metrics 

```
[morgan.lupton@COMP10906:/PATH/TO/kube-state-metrics]$ oc apply -f examples/standard/
Warning: oc apply should be used on resource created by either oc create --save-config or oc apply
clusterrolebinding.rbac.authorization.k8s.io/kube-state-metrics configured
Warning: oc apply should be used on resource created by either oc create --save-config or oc apply
clusterrole.rbac.authorization.k8s.io/kube-state-metrics configured
deployment.apps/kube-state-metrics created
serviceaccount/kube-state-metrics created
service/kube-state-metrics created

```

4. Validate Kube State Metrics has been successfully deployed. 

```
[morgan.lupton@COMP10906:/PATH/TO/kube-state-metrics]$ oc get pods --all-namespaces | grep state
kube-system                                  kube-state-metrics-5c5cb55b4-vgw5d                           1/1     Running     0          2m59s
```

### Create Datadog Project & Apply Customized Manifest

1. Create a new project (same as a namespace in regular Kubernetes) to use for the Datadog Agent deployment. 

```
[morgan.lupton@COMP10906:~]$ oc new-project datadog
```

2. It should automatically switch you into your new project; but in case you aren't in the `datadog` project, switch to the project context using the following command. 

```
[morgan.lupton@COMP10906:~]$ oc project datadog
```

3. Navigate to the manifests directory and apply the SCC. 

```
[morgan.lupton@COMP10906:~]$ cd /PATH/TO/operator-example-with-everything/manifests
[morgan.lupton@COMP10906:/PATH/TO/manifests]$ oc apply -f scc.yaml
```

4. Modify the Datadog Operator manifest (`~/manifests/datadog-agent-all.yaml`) so that it includes your Datadog API & App Keys. The section will look like this:

```
spec:
  credentials:
    apiKey: "<INSERT-API-KEY-HERE>"
    appKey: "<INSERT-APP-KEY-HERE>"
```

5. Apply the Datadog Operator manifest to your cluster. 

```
[morgan.lupton@COMP10906:/PATH/TO/manifests]$ oc apply -f datadog-agent-all.yaml
```

6. Validate everything is deployed by running the following command. It may take a few minutes for the cluster agent and Datadog Agent to come online. 

```
[morgan.lupton@COMP10906:~]$ oc get pods
NAME                                     READY   STATUS    RESTARTS   AGE
datadog-agent-24bnc                      4/4     Running   0          110s
datadog-cluster-agent-6cd6d5c596-ptf95   1/1     Running   0          2m1s
```

### Collect Control Plane Metrics & Enable Further Integrations

1. Add annotations to the Operator's API server to collect Kube API Server metrics.

Edit the annotations by first running this command
```
[morgan.lupton@COMP10906:~]$ oc edit service metrics -n openshift-apiserver-operator

```

Then add the annotations 
```
...
  annotations:
    ad.datadoghq.com/endpoints.check_names: '["kube_apiserver_metrics"]'
    ad.datadoghq.com/endpoints.init_configs: '[{}]'
    ad.datadoghq.com/endpoints.instances: '[{ "prometheus_url": "https://%%host%%:%%port%%/metrics",
      "bearer_token_auth": "true" }]'
...
```

2. Do the same thing for the Operator's Kube API server.

Edit the annotations by running this command
```
[morgan.lupton@COMP10906:~]$ oc edit service metrics -n openshift-kube-apiserver-operator
```

Then add the annotations
```
...
  annotations:
    ad.datadoghq.com/endpoints.check_names: '["kube_apiserver_metrics"]'
    ad.datadoghq.com/endpoints.init_configs: '[{}]'
    ad.datadoghq.com/endpoints.instances: '[{ "prometheus_url": "https://%%host%%:%%port%%/metrics",
      "bearer_token_auth": "true" }]'
...
```

3. Validate that the Kube API Server metrics are flowing in by either running the `agent status` comamnd, or checking for the metrics in Metrics Explorer. 

