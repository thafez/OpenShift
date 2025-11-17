# Openshift Online Installation (VMWare UPI Installation)

This document for Openshift online installation on ESXI VMWare 

## Prepare Bastion Host

**Disable firewall**

`systemctl disable --now firewalld`

**Disable SELinux**

`sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config`

**System Reboot**

`systemctl reboot`

**Install Required packages** 

`yum install -y httpd bash-completion httpd-tools curl jq`

**create workspace directory**

`mkdir -p “workspace directory”`

**Set httpd server to listen on port 8080 and 8443 on the private IP of the bastion host**

`vi /etc/httpd/conf/httpd.conf`

Modify port listen from 80 to 8080 then restart the service

```
systemctl restart httpd 
systemctl enable --now httpd 
```
**Install Openshift Client and Installer**

```
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/stable-4.12/openshift-install-linux.tar.gz
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/stable-4.12/openshift-client-linux.tar.gz
```
**Unpack the archive:**

```
tar xvzf openshift-install-linux.tar.gz
tar xvfz openshift-client-linux.tar.gz
```


**Place `oc` `kubectl` binary in a directory that is on your PATH execute the following command:**
`cp kubectl oc /usr/local/bin/ `


**Configure "oc" bash auto-completion:**
execute the following then reopen your shell

```
oc completion bash >>/etc/bash_completion.d/oc_completion

kubectl completion bash >>/etc/bash_completion.d/kubectl_completion 
```

**Generate ssh key file**

`ssh-keygen -t rsa -b 4096 -N '' -f /root/workspace/sources/.ssh/id_rsa`

**Start ssh-agent and load key file (This allows auto-login with the SSH key)**

`eval "$(ssh-agent -s)"`

**Convert the pull secret to a more readable format:**

`cat pull-secret-raw.txt | jq . > pull-secret.json`

**Create install-config.yaml file**
Specify more details about OpenShift Container Platform cluster’s platform or modify the values of the required parameters.

```yaml
additionalTrustBundlePolicy: Proxyonly
apiVersion: v1
baseDomain: devfasah.sa
compute:
- architecture: amd64
  name: worker
    #platform: vsphere
  replicas: 0
controlPlane:
  architecture: amd64
  name: master
    # platform: vsphere
  replicas: 3
metadata:
  creationTimestamp: null
  name: ocp-z
platform:
  none: {}
fips: false
pullSecret: '{"auths": {
    "cloud.openshift.com": {
      "auth": "b3BlbnNoaWZ0LXJlbGVhc2UtZGV2K29jbV9hY2Nlc3NfN2Y0OTEwYzhhYjdlNDk4ZDhmZDJhMjU2YjBhMDgyYTI6VzhHQkUzTlNXQkVSOEZMVkNFVjRHTTA2TjJUUUJYVFFPUlNZUENCSlVSRURQOVo0Sks4RTNFWUhRUE5FTEFWWg==",
      "email": "mababneh@tabadul.sa"
    },
    "quay.io": {
      "auth": "b3BlbnNoaWZ0LXJlbGVhc2UtZGV2K29jbV9hY2Nlc3NfN2Y0OTEwYzhhYjdlNDk4ZDhmZDJhMjU2YjBhMDgyYTI6VzhHQkUzTlNXQkVSOEZMVkNFVjRHTTA2TjJUUUJYVFFPUlNZUENCSlVSRURQOVo0Sks4RTNFWUhRUE5FTEFWWg==",
      "email": "mababneh@tabadul.sa"
    },
    "registry.connect.redhat.com": {
      "auth": "fHVoYy1wb29sLTNmOWNhY2JmLTEzMDUtNDdlZC1iMzhkLTUyOWRmMzcyYTFlMjpleUpoYkdjaU9pSlNVelV4TWlKOS5leUp6ZFdJaU9pSmpNVFE1WlRnNVpHVTJPR00wTmpneE9XSmpZakJsTVRZM01XTXlPVFkyT1NKOS5YR3BoNlVNSFVQM3RiUWg2T0M2THpfTmdZZzRLY25CZmtxM211ZEd5bHZsV0NES29qdFRnOVdreWtCU0MwbGFNa0J5V1dOZjNza0dvSk9NbmJFLXAtR0h4dkF5UWkxeVVZNGVBa2NUNlZiOXdsNDFNaXVfQktmYU9ZT252bmZnQ1R4T3dsbzlTdU5LNzh6Q0tlRUJRZWtYcExBbjFNZV9nRnV5M3o3emdYZ0R1NnIyMHhOSl9nTFJSYld4c293Vmlpb1Yxd3pwNDdzTmQtTXVVVjY4VWN1Q2p3TXlraEdFcF9weFZOQlJwZDVKUzhGUkNLUUFYV09qbnE5MkVNRWxhWWFOZnUwUlZSNURwTDZlQUxwR1E3Y0E2WXpnbllyS3B4YnV1endEdFVTZWRLY1dTXzdlcGNwTi1hSTZuZ2l2WDgzeDA3cXRLc21oN2RsZkVqNERPajVtZzhQNjVHZWNHY0VHRnBuTTFhMGI0d3VTdnZQSE9ldE1tOVVzS1dtWllDU0lfa1VoU3IxQzFYRGUzUjQ0cUpvdmJCdWh4THdwUTlNbmp5a3RrdmlnN2tyQnpkZzBPWEtyNHRJTkhWYXNqNXd1SjlwOVJPLVNPcnhzNkNKUTNET256WFRPRnFNcXpBRmNvX2xDT3JJeGZwajhBdkdtY0VyanowSjJZNGQ2T3ZITnhjRS0ySjZjRDZuaXRsN2dGOURPYmFRQVpDemxEUkJyR2NqRzIzZHd3enlQMlNZLUFxbURDV083Z01lM3FXdWR5Tl9UbnhoNjl2MTllektUdHl1X05qZVdjaUtsaWxyenlTNXkwbEtqV01aOXBQeHRvaWlmc3d5SlF4YmVrNmdENl9LeVhQdHZ6NnUtMXRxQ045T1Q1NGpqSThDUUV4aGs5Y1IxTFhxRQ==",
      "email": "mababneh@tabadul.sa"
    },
    "registry.redhat.io": {
      "auth": "fHVoYy1wb29sLTNmOWNhY2JmLTEzMDUtNDdlZC1iMzhkLTUyOWRmMzcyYTFlMjpleUpoYkdjaU9pSlNVelV4TWlKOS5leUp6ZFdJaU9pSmpNVFE1WlRnNVpHVTJPR00wTmpneE9XSmpZakJsTVRZM01XTXlPVFkyT1NKOS5YR3BoNlVNSFVQM3RiUWg2T0M2THpfTmdZZzRLY25CZmtxM211ZEd5bHZsV0NES29qdFRnOVdreWtCU0MwbGFNa0J5V1dOZjNza0dvSk9NbmJFLXAtR0h4dkF5UWkxeVVZNGVBa2NUNlZiOXdsNDFNaXVfQktmYU9ZT252bmZnQ1R4T3dsbzlTdU5LNzh6Q0tlRUJRZWtYcExBbjFNZV9nRnV5M3o3emdYZ0R1NnIyMHhOSl9nTFJSYld4c293Vmlpb1Yxd3pwNDdzTmQtTXVVVjY4VWN1Q2p3TXlraEdFcF9weFZOQlJwZDVKUzhGUkNLUUFYV09qbnE5MkVNRWxhWWFOZnUwUlZSNURwTDZlQUxwR1E3Y0E2WXpnbllyS3B4YnV1endEdFVTZWRLY1dTXzdlcGNwTi1hSTZuZ2l2WDgzeDA3cXRLc21oN2RsZkVqNERPajVtZzhQNjVHZWNHY0VHRnBuTTFhMGI0d3VTdnZQSE9ldE1tOVVzS1dtWllDU0lfa1VoU3IxQzFYRGUzUjQ0cUpvdmJCdWh4THdwUTlNbmp5a3RrdmlnN2tyQnpkZzBPWEtyNHRJTkhWYXNqNXd1SjlwOVJPLVNPcnhzNkNKUTNET256WFRPRnFNcXpBRmNvX2xDT3JJeGZwajhBdkdtY0VyanowSjJZNGQ2T3ZITnhjRS0ySjZjRDZuaXRsN2dGOURPYmFRQVpDemxEUkJyR2NqRzIzZHd3enlQMlNZLUFxbURDV083Z01lM3FXdWR5Tl9UbnhoNjl2MTllektUdHl1X05qZVdjaUtsaWxyenlTNXkwbEtqV01aOXBQeHRvaWlmc3d5SlF4YmVrNmdENl9LeVhQdHZ6NnUtMXRxQ045T1Q1NGpqSThDUUV4aGs5Y1IxTFhxRQ==",
      "email": "mababneh@tabadul.sa"}}}'
sshKey: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCUHT2hSr81CRJm4/IeEuu2++0aZ/tKJm7etq27//5ZAQ8a5te1L2w4FfgL682Ih3CMjd3pBnqddPzuKNTFBF6t+OfxmzUA12JK5seNpWj3BVZ0ThtZQuCOQj3NhqCEMSkP2Pc+UnJnwFdhYowA2Hx0KOkesmJKMbWLQDxQHWEqPlcoEiRpfQXpH7TsDJoB1nUHjNzJ4QwPMZwsus8uLvoVzISj+DBRNxmRp/PCXgMLE8rjkQHcBmI9v1NUjP4FRM66k+Prs6YF8do6/ypZufymltZS2GkC074kbdaq8PCd1RhhT3/Hd1GDNeCuXyRm4gYcxapVfHrXaFbrGw9yi0kxTZn76GFPFAPUwZNfFJgTf3lrXPpRF9msihZ0OAdwH4tWkXAgX5dd1RkIH4cLphkW8OVC6160/RdYtavTa+47bBgzfi+nlXN/6GzR2haZoQ2j3g+Iji345e1VyV+/NK6IbrNOqHEUwLKTzAmeV6XT4aAdGbmtAHYzaSWwmxlss1NmSj4ZGwdKBiqjDJV4zbjF+mUuHwauQzOO2/kKi/FtdA3hMDTCz/Mj6mtyEI1PUpIchAFrmQDh9+q4cia9zj7jQhk7Sbclj2icwOTQU4YpHknyYTFw/EGK0RSTulK3nzEv8GeJQoGrWbx+kEYc8vml6mCXwEg0dIK9N+OzMLkXBQ== root@bstn.devfasah.sa'
```

**Creating the Kubernetes manifest and Ignition config files**

Generate the Kubernetes manifest and Ignition config files that the cluster needs to configure the machines.
The installation configuration file transforms into the Kubernetes manifests. The manifests wrap into the Ignition configuration files, which are later used to configure the cluster machines.

**Backup the install config file before starting the process, as it will be deleted after created the ignitions**

`cp install-files/install-config.yaml sources/install-config.yaml`

**Create the manifest files**

`openshift-install create manifests --dir /root/workspace/install-files`

**Remove the Kubernetes manifest files that define the control plane machines, compute machine sets, and control plane machine sets** 

`rm -f openshift/99_openshift-cluster-api_master-machines-*.yaml openshift/99_openshift-cluster-api_worker-machineset-*.yaml openshift/99_openshift-machine-api_master-control-plane-machine-set.yaml`

**Change the mastersSchedulable parameter in the <installation_directory>/manifests/cluster-scheduler-02-config.yml Kubernetes manifest file is set to false.**

`vim /manifests/cluster-scheduler-02-config.yml`

Locate the `mastersSchedulable parameter and ensure that it is set to false`

**To create the Ignition configuration files, run the following command from the directory that contains the installation program:**

`openshift-install create ignition-configs --dir /root/workspace/install-files/ `

set permissions and copy ignitions to https server 

`chmod 544 install-files/*.ign`
`cp -p /install-files/*.ign /var/www/html/`

**Create a pointer for ign files**

* Bootstrap

```
cat << EOF > install-files/append-bootstrap.ign
 {
 "ignition": {
 "config": {
 "merge": [
  {
"source": "http://192.168.22.14:8080/bootstrap.ign", 
"verification": {}
}
]
},
"timeouts": {},
"version": "3.2.0"
},
"networkd": {},
"passwd": {},
"storage": {},
"systemd": {}
}
EOF
```
* Workers

```
cat << EOF > install-files/append-worker.ign
 {
 "ignition": {
 "config": {
 "merge": [
  {
"source": "http://192.168.22.14:8080/worker.ign", 
"verification": {}
}
]
},
"timeouts": {},
"version": "3.2.0"
},
"networkd": {},
"passwd": {},
"storage": {},
"systemd": {}
}
EOF
```
* Masters

```
cat << EOF > install-files/append-master.ign
 {
 "ignition": {
 "config": {
 "merge": [
  {
"source": "http://192.168.22.14:8080/master.ign", 
"verification": {}
}
]
},
"timeouts": {},
"version": "3.2.0"
},
"networkd": {},
"passwd": {},
"storage": {},
"systemd": {}
}
EOF
```
**Convert Ignition files to Base64 encoding**

```
base64 -w0 install-files/append-master.ign > install-files/append-master.64 
base64 -w0 install-files/append-worker.ign > install-files/append-worker.64 
base64 -w0 install-files/append-bootstrap.ign > install-files/append-bootstrap.64 
```
### Creating Virtual Machines

* Download coreos OVA image from redhat download site
* From vmware vCenter, create a new folder. give it a name that matches the cluster name that you specified in the install-config.yaml file:
* Deploy OVF as a template
* Create VMs from the deployed template
* For all nodes add the following configuration parameters:
   Select VM Options --> Advanced --> add Configuration parameters
* Bootstrap Values
   - Name: guestinfo.ignition.config.data.encoding | | Value : base64
   - Name: disk.EnableUUID || Value : TRUE
   - Name: guestinfo.ignition.config.data || Value : Base64 Code 
   - Name: guestinfo.afterburn.initrd.network-kargs || Value : ip=192.168.22.15::192.168.22.1:255.255.255.0:btsrp.ocp-z.devfasah.sa:ens192:none nameserver=192.168.34.4
   - Name: stealclock.enable || Value : TRUE

* Bootstrap Code : `IHsKICJpZ25pdGlvbiI6IHsKICJjb25maWciOiB7CiAibWVyZ2UiOiBbCiAgewoic291cmNlIjogImh0dHA6Ly8xOTIuMTY4LjIyLjE0OjgwODAvYm9vdHN0cmFwLmlnbiIsIAoidmVyaWZpY2F0aW9uIjoge30KfQpdCn0sCiJ0aW1lb3V0cyI6IHt9LAoidmVyc2lvbiI6ICIzLjIuMCIKfSwKIm5ldHdvcmtkIjoge30sCiJwYXNzd2QiOiB7fSwKInN0b3JhZ2UiOiB7fSwKInN5c3RlbWQiOiB7fQp9Cg==`
* Masters Code
`IHsKICJpZ25pdGlvbiI6IHsKICJjb25maWciOiB7CiAibWVyZ2UiOiBbCiAgewoic291cmNlIjogImh0dHA6Ly8xOTIuMTY4LjIyLjE0OjgwODAvbWFzdGVyLmlnbiIsIAoidmVyaWZpY2F0aW9uIjoge30KfQpdCn0sCiJ0aW1lb3V0cyI6IHt9LAoidmVyc2lvbiI6ICIzLjIuMCIKfSwKIm5ldHdvcmtkIjoge30sCiJwYXNzd2QiOiB7fSwKInN0b3JhZ2UiOiB7fSwKInN5c3RlbWQiOiB7fQp9Cg==`
* Workers Code
`IHsKICJpZ25pdGlvbiI6IHsKICJjb25maWciOiB7CiAibWVyZ2UiOiBbCiAgewoic291cmNlIjogImh0dHA6Ly8xOTIuMTY4LjIyLjE0OjgwODAvd29ya2VyLmlnbiIsIAoidmVyaWZpY2F0aW9uIjoge30KfQpdCn0sCiJ0aW1lb3V0cyI6IHt9LAoidmVyc2lvbiI6ICIzLjIuMCIKfSwKIm5ldHdvcmtkIjoge30sCiJwYXNzd2QiOiB7fSwKInN0b3JhZ2UiOiB7fSwKInN5c3RlbWQiOiB7fQp9Cg==`

**Monitor Progress**
`openshift-install --dir ./ wait-for bootstrap-complete --log-level=debug`

**Approve Certificates**
login to the cluster, export KUBECONFIG  

`export KUBECONFIG=/root/workspace/install-files/auth/kubeconfig`

Get the pending Certificates and approve 

`oc get csr`

`oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs --no-run-if-empty oc adm certificate approve`

**Confirm that all the cluster components are online with the following command:**

`oc get clusteroperators`

**Alternatively, the following command notifies you when all of the clusters are available. It also retrieves and displays credentials:**

`openshift-install --dir install-files/ wait-for install-complete`

**Creating an infrastructure node**
Add a label to the worker node

`oc label node  infra01.ocp-z.devfasah.sa node-role.kubernetes.io/app=""`

`oc label node infra01.ocp-z.devfasah.sa node-role.kubernetes.io/infra=""`

check now if the nodes are labled or not 

`oc get nodes`

Taint to prevent scheduling user workloads on it

`oc adm taint nodes infra01.ocp-z.devfasah.sa node-role.kubernetes.io/infra=reserved:NoSchedule`

Add the taint with NoExecute Effect along with the above taint with NoSchedule Effect

`oc adm taint nodes infra01.ocp-z.devfasah.sa node-role.kubernetes.io/infra=reserved:NoExecute`

repeat for all infra nodes

Moving the router to Infra Nodes

`oc patch ingresscontroller/default -n  openshift-ingress-operator  --type=merge -p '{"spec":{"nodePlacement": {"nodeSelector": {"matchLabels": {"node-role.kubernetes.io/infra": ""}},"tolerations": [{"effect":"NoSchedule","key": "node-role.kubernetes.io/infra","value": "reserved"},{"effect":"NoExecute","key": "node-role.kubernetes.io/infra","value": "reserved"}]}}}'`

`oc edit ingresscontroller default -n openshift-ingress-operator`

add to spec

```yaml
  spec:
    nodePlacement:
      nodeSelector: 
        matchLabels:
          node-role.kubernetes.io/infra: ""
      tolerations:
      - effect: NoSchedule
        key: node-role.kubernetes.io/infra
        value: reserved
      - effect: NoExecute
        key: node-role.kubernetes.io/infra
        value: reserved
```
Confirm that the router pod is running on the infra node

`oc get pod -n openshift-ingress -o wide`

## Adding admin user to the cluster

**Creating an htpasswd file using Linux**
- Create or update your flat file with a user name and hashed password:

`htpasswd -c -B -b users.htpasswd <username> <password>`

**Creating the htpasswd secret**

`oc create secret generic htpass-secret --from-file=htpasswd=<path_to_users.htpasswd> -n openshift-config`

**create CR for identity provider**

```yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: local_provider 
    mappingMethod: claim 
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-secret
```

**add cluster admin role to user**

`oc adm policy add-cluster-role-to-user cluster-admin <user>`

**Changing the image registry’s management state**

Change managementState Image Registry Operator configuration from Removed to Managed. For example:

`oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"managementState":"Managed"}}'`

### Image registry storage configuration

The Image Registry Operator is not initially available for platforms that do not provide default storage. After installation, you must configure your registry to use storage so that the Registry Operator is made available.

Instructions are shown for configuring a persistent volume, which is required for production clusters. Where applicable, instructions are shown for configuring an empty directory as the storage location, which is available for only non-production clusters.

Additional instructions are provided for allowing the image registry to use block storage types by using the `Recreate` rollout strategy during upgrades

**Verify that you do not have a registry pod**

`oc get pod -n openshift-image-registry -l docker-registry=default`

**Create a PVC to use the cephfs storage class. For example:**

```
cat <<EOF | oc apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: registry-storage-pvc
 namespace: openshift-image-registry
spec:
 accessModes:
 - ReadWriteMany
 resources:
   requests:
     storage: 500Gi
 storageClassName: ocs-storagecluster-cephfs
EOF
    
```

**Configure the image registry to use the CephFS file system storage by entering the following command:**

`oc patch config.image/cluster -p '{"spec":{"managementState":"Managed","replicas":2,"storage":{"managementState":"Unmanaged","pvc":{"claim":"registry-storage-pvc"}}}}' --type=merge`

 * Check the clusteroperator status

`oc get clusteroperator image-registry`

**Install Butane on your installation host by using the following command:**

`sudo dnf -y install butane`

**Create a Butane config, 99-master-chrony-conf-override.bu, including the contents of the chrony.conf file for the control plane nodes**

```yaml
variant: openshift
version: 4.15.0
metadata:
  name: 99-master-chrony-conf-override
  labels:
    machineconfiguration.openshift.io/role: master
storage:
  files:
    - path: /etc/chrony.conf
      mode: 0644
      overwrite: true
      contents:
        inline: |
          # Use public servers from the pool.ntp.org project.
          # Please consider joining the pool (https://www.pool.ntp.org/join.html).

          # The Machine Config Operator manages this file
          # server openshift-master-0.<cluster-name>.<domain> iburst 
          # server openshift-master-1.<cluster-name>.<domain> iburst
          server 192.168.215.175 iburst

          stratumweight 0
          driftfile /var/lib/chrony/drift
          rtcsync
          makestep 10 3
          bindcmdaddress 127.0.0.1
          bindcmdaddress ::1
          keyfile /etc/chrony.keys
          commandkey 1
          generatecommandkey
          noclientlog
          logchange 0.5
          logdir /var/log/chrony

          # Configure the control plane nodes to serve as local NTP servers
          # for all compute nodes, even if they are not in sync with an
          # upstream NTP server.

          # Allow NTP client access from the local network.
          allow all
          # Serve time even if not synchronized to a time source.
          local stratum 3 orphan
```

**Use Butane to generate a MachineConfig object file, 99-master-chrony-conf-override.yaml, containing the configuration to be delivered to the control plane nodes**

`butane 99-master-chrony-conf-override.bu -o 99-master-chrony-conf-override.yaml`

**Create a Butane config, 99-worker-chrony-conf-override.bu, including the contents of the chrony.conf file for the compute nodes that references the NTP servers on the control plane nodes**

```yaml
variant: openshift
version: 4.15.0
metadata:
  name: 99-worker-chrony-conf-override
  labels:
    machineconfiguration.openshift.io/role: worker
storage:
  files:
    - path: /etc/chrony.conf
      mode: 0644
      overwrite: true
      contents:
        inline: |
          # The Machine Config Operator manages this file.
          # server openshift-master-0.<cluster-name>.<domain> iburst 
          # server openshift-master-1.<cluster-name>.<domain> iburst
          server 192.168.215.175 iburst

          stratumweight 0
          driftfile /var/lib/chrony/drift
          rtcsync
          makestep 10 3
          bindcmdaddress 127.0.0.1
          bindcmdaddress ::1
          keyfile /etc/chrony.keys
          commandkey 1
          generatecommandkey
          noclientlog
          logchange 0.5
          logdir /var/log/chrony
```

**Use Butane to generate a MachineConfig object file, 99-worker-chrony-conf-override.yaml, containing the configuration to be delivered to the worker nodes:**

`butane 99-worker-chrony-conf-override.bu -o 99-worker-chrony-conf-override.yaml`

**Apply the 99-master-chrony-conf-override.yaml policy to the control plane nodes.**

`oc apply -f 99-master-chrony-conf-override.yaml`

**Apply the 99-worker-chrony-conf-override.yaml policy to the compute nodes.**

`oc apply -f 99-worker-chrony-conf-override.yaml`

**Check the status sof NTP applied settings**

`oc describe machineconfigpool`


## Install Openshift Data Foundation Storage 

**Installing Local Storage Operator**

Install the Local Storage Operator from the Operator Hub before creating Red Hat OpenShift Data Foundation clusters on local storage devices.
Procedure

- Log in to the OpenShift Web Console.
- Click Operators OperatorHub.
- Type local storage in the Filter by keyword​ box to find the Local Storage Operator from the list of operators, and click on it.
- Set the following options on the Install Operator page:
   - Update channel as stable.
   - Installation mode as A specific namespace on the cluster.
   - Installed Namespace as Operator recommended namespace openshift-local-storage.
   - Update approval as Automatic.
   - Click Install.
     
**Installing Red Hat OpenShift Data Foundation Operator**
You can install Red Hat OpenShift Data Foundation Operator using the Red Hat OpenShift Container Platform Operator Hub.

- label nodes as infra and storage nodes
  
```
oc label node strg01.ocp-z.devfasah.sa node-role.kubernetes.io/infra=""
oc label node strg01.ocp-z.devfasah.sa cluster.ocs.openshift.io/openshift-storage=""

```

**Adding a NoSchedule OpenShift Data Foundation taint is also required so that the infra node will only schedule OpenShift Data Foundation resources and repel any other non-OpenShift Data Foundation workloads.**

`oc adm taint node strg01.ocp-z.devfasah.sa node.ocs.openshift.io/storage="true":NoSchedule`

**repeate for all storage nodes**

**Create storagesystem**

- Click Storage --> Data Foundation --> Storage systems
- Create storage system
- Create a new StorageClass using local storage devices
- Select the storage nodes, SSD/NVMe Disk type
- Select Default SDN

### Notes
to login export KUBECONFIG 

`export KUBECONFIG=/root/workspace/install-files/auth/kubeconfig`


### check below links
https://examples.openshift.pub/cluster-installation/vmware/example/
https://github.com/openshift/installer/blob/master/docs/user/vsphere/install_upi.md
