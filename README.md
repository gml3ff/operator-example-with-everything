# operator-example-with-everything
## Datadog Operator Example that Includes Kubernetes, Docker, Logs, Process, NPM, APM, Control Plane, and OpenShift Metrics

This guide explains the installation process behind configuring every possible integration you may want to configure with the Operator. Included in this repository are instructions on how to deploy a local OpenShift cluster (v4+) as well as configure the Datadog agent along with all relevant integrations. 

Inspiration came from [Ryan Hennessy's guide that you can find here](https://github.com/ryhennessy/datadog-operator-openshift-example). 

### Configuring `crc` to work on your local machine

1. Navigate to [Red Hat's CodeReady Containers product page](https://cloud.redhat.com/openshift/install/crc/installer-provisioned?intcmp=7013a000002CtetAAC). You will need to create a Red Hat account in order to access the page. 

2. Once logged in, download the CodeReady Containers tarball archive. 

![download_crc](images/crc_download.png)

3. Extract the archive and place the contents in a location that you can easily add to your `$PATH`. I extracted to `/usr/local/share/` using the following command. 

```
[morgan.lupton@mycomputer:~/Downloads]$ tar -xvf crc-macos-amd64.tar.xz -C /usr/local/share/
```

4. Add the path to the `crc` executable to your `$PATH` env var. Please keep in mind that the path name will depend on the version of crc you downloaded (in this case 1.14.0). 

```
[morgan.lupton@mycomputer:~/Downloads]$ export PATH="/usr/local/share/crc-macos-1.14.0-amd64/:$PATH"
```

Make sure to add this statement to your bash profile such that it persists after you exit out of the terminal. 

```
[morgan.lupton@mycomputer:~/Downloads]$ export PATH="/usr/local/share/crc-macos-1.14.0-amd64/:$PATH"
```
