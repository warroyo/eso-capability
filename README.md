# Installing External Secrets Operator as a capability


This is a workaround until custom capabilties are officially supported. Some of the steps here are not officially supported and may not work over time. 





## Setup the TMC plugins to work with Tanzu Platform

This assumes you already have the Tanzu cli and the TMC plugins installed. if not, follow the [docs](https://docs.vmware.com/en/VMware-Tanzu-CLI/index.html) to get that installed.

1. Install [mitmproxy](https://mitmproxy.org/) you will also need to add the mitm proxy [root ca](https://docs.mitmproxy.org/stable/concepts-certificates/) to your system. 
2. get your Tanzu platform project ID -  this can be foudn with `tanzu current context` while connected to a tanzu platform context

3. create a context for the tanzu platform tmc api

```bash
tanzu context create tpk8s-tmc -t tmc --endpoint tmc.tanzu.cloud.vmware.com
```


4. run the mitmproxy with a setting to also re-write headers

```bash
mitmdump --modify-headers "/X-Project-Id/your-project-id"
```


5. in another terminal window set your proxy and use the cli

```bash
export HTTPS_PROXY=localhost:8080

tanzu tmc cluster list

```


## Enable Flux

To manage the ESO capabiltiy and ensure it installs on all clusters in a cluster group we will use the clustergroup level kustomization functionality. In this case since we are going to use helm also, either enable the helm capability on the cluster group or use the TMC continuous delivery APIs to enable helm. in my case I used the helm capability since it's built in.


1. enable TMC CD

```bash
tanzu tmc continuousdelivery enable -g your-cg -s clustergroup
```

2. create a git repo

```bash

 ytt -f tpk8s-resources/gitrepo.yml -v cg=your-cg -v gitrepo=https://github.com/warroyo/eso-capabiltiy | tanzu tmc continuousdelivery gitrepository create -s clustergroup -f-
```


3. create a kustomization

```bash
 ytt -f tpk8s-resources/kustomization.yml -v cg=your-cg  | tanzu tmc continuousdelivery kustomization create -s clustergroup -f-
 ```


 Now you should see the ESO controller installed and a capabiltiy show up for it when creating a profile.

 