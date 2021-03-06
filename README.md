# Canonical Kubernetes with F5 Big-IP Load Balancers

This document describes how to integrate Canonical Kubernetes (CDK) with F5 Networks Big-IP load balancer devices.

We will deploy Canonical Kubernetes, the F5 Networks Big-IP Device and the F5 k8s-bigip-ctlr to control the loadbalancer.

### Index

- [Deploying Canonical Kubernetes](https://github.com/CalvinHartwell/canonical-kubernetes-f5-big-ip#deploying-canonical-kubernetes-cdk)
- [Deploying F5 Big-IP Load-Balancer](https://github.com/CalvinHartwell/canonical-kubernetes-f5-big-ip#deploying-the-f5-big-ip-load-balancer)
  - [Configuring the F5 Big-IP Load-Balancer](https://github.com/CalvinHartwell/canonical-kubernetes-f5-big-ip#configuring-the-f5-big-ip-load-balancer)
  - [Removing the F5 Big-IP Load-Balancer](https://github.com/CalvinHartwell/canonical-kubernetes-f5-big-ip#removing-the-f5-big-ip-load-balancer)
  - [Deploying the F5 Big-IP Load-Balancer Controller](https://github.com/CalvinHartwell/canonical-kubernetes-f5-big-ip#deploying-the-f5-big-ip-load-balancer-controller-on-cdk)
- [Utilising the F5 Big-IP Load Balancer with Kubernetes](https://github.com/CalvinHartwell/canonical-kubernetes-f5-big-ip#utilising-the-f5-big-ip-load-balancer-with-kubernetes)
  - [Removing the F5 Big-IP Controller Container](https://github.com/CalvinHartwell/canonical-kubernetes-f5-big-ip#removing-the-f5-big-ip-controller-container)
  - [Troubleshooting](https://github.com/CalvinHartwell/canonical-kubernetes-f5-big-ip#troubleshooting-the-f5-big-ip-controller)
- [Conclusion](https://github.com/CalvinHartwell/canonical-kubernetes-f5-big-ip#conclusion)
  - [Software versions](https://github.com/CalvinHartwell/canonical-kubernetes-f5-big-ip#software-versions)
  - [Getting Help & Support](https://github.com/CalvinHartwell/canonical-kubernetes-f5-big-ip#getting-help--support)
- [Useful Links](https://github.com/CalvinHartwell/canonical-kubernetes-f5-big-ip#useful-links)

## Deploying Canonical Kubernetes (CDK)

Deploying Canonical Kubernetes is really easy, I assume you are running on an Ubuntu machine and you have an AWS account.  Grab a new API key from AWS and put that into:

```
~/.aws/credentials
```

In this format:

```
[default]
aws_access_key_id=<access_id>
aws_secret_access_key=<secret_key>
```

Next install juju on your machine, we will use the snap:

```
sudo apt-get install snap
snap install juju
```

Next we bootstrap juju so it's ready to use aws:

```
juju bootstrap
```

We follow the steps here for bootstrapping our cloud for AWS:

```
calvinh@ubuntu-ws:~/.aws$ juju bootstrap
Clouds
aws
aws-china
aws-gov
azure
azure-china
cloudsigma
google
joyent
localhost
oracle
rackspace
salesmaas

Select a cloud [localhost]: aws

Regions in aws:
ap-northeast-1
ap-northeast-2
ap-south-1
ap-southeast-1
ap-southeast-2
ca-central-1
eu-central-1
eu-west-1
eu-west-2
sa-east-1
us-east-1
us-east-2
us-west-1
us-west-2

Select a region in aws [us-east-1]: eu-west-1

Enter a name for the Controller [aws-eu-west-1]: f5-k8s

Creating Juju controller "f5-k8s" on aws/eu-west-1
Looking for packaged Juju agent version 2.3.2 for amd64
Launching controller instance(s) on aws/eu-west-1...
 - i-08ce69142f943b5a4 (arch=amd64 mem=4G cores=2)eu-west-1a"
Installing Juju agent on bootstrap instance
Fetching Juju GUI 2.11.3
Waiting for address
Attempting to connect to 172.31.20.176:22
Attempting to connect to 34.244.155.220:22
Connected to 34.244.155.220
Running machine configuration script...


Bootstrap agent now started
Contacting Juju controller at 34.244.155.220 to verify accessibility...
Bootstrap complete, "f5-k8s" controller now available
Controller machines are in the "controller" model
Initial model "default" added
```

Once the controller has been configured, we now deploy CDK:

```
calvinh@ubuntu-ws:~/.aws$ juju deploy canonical-kubernetes
Located bundle "cs:bundle/canonical-kubernetes-150"
Resolving charm: cs:~containers/easyrsa-27
Resolving charm: cs:~containers/etcd-63
Resolving charm: cs:~containers/flannel-40
Resolving charm: cs:~containers/kubeapi-load-balancer-43
Resolving charm: cs:~containers/kubernetes-master-78
Resolving charm: cs:~containers/kubernetes-worker-81
Executing changes:
- upload charm cs:~containers/easyrsa-27 for series xenial
- deploy application easyrsa on xenial using cs:~containers/easyrsa-27
  added resource easyrsa
- set annotations for easyrsa
- upload charm cs:~containers/etcd-63 for series xenial
- deploy application etcd on xenial using cs:~containers/etcd-63
  added resource etcd
  added resource snapshot
- set annotations for etcd
- upload charm cs:~containers/flannel-40 for series xenial
- deploy application flannel on xenial using cs:~containers/flannel-40
  added resource flannel-amd64
  added resource flannel-s390x
- set annotations for flannel
- upload charm cs:~containers/kubeapi-load-balancer-43 for series xenial
- deploy application kubeapi-load-balancer on xenial using cs:~containers/kubeapi-load-balancer-43
- expose kubeapi-load-balancer
- set annotations for kubeapi-load-balancer
- upload charm cs:~containers/kubernetes-master-78 for series xenial
- deploy application kubernetes-master on xenial using cs:~containers/kubernetes-master-78
  added resource cdk-addons
  added resource kube-apiserver
  added resource kube-controller-manager
  added resource kube-scheduler
  added resource kubectl
- set annotations for kubernetes-master
- upload charm cs:~containers/kubernetes-worker-81 for series xenial
- deploy application kubernetes-worker on xenial using cs:~containers/kubernetes-worker-81
  added resource cni-amd64
  added resource cni-s390x
  added resource kube-proxy
  added resource kubectl
  added resource kubelet
- expose kubernetes-worker
- set annotations for kubernetes-worker
- add relation kubernetes-master:kube-api-endpoint - kubeapi-load-balancer:apiserver
- add relation kubernetes-master:loadbalancer - kubeapi-load-balancer:loadbalancer
- add relation kubernetes-master:kube-control - kubernetes-worker:kube-control
- add relation kubernetes-master:certificates - easyrsa:client
- add relation etcd:certificates - easyrsa:client
- add relation kubernetes-master:etcd - etcd:db
- add relation kubernetes-worker:certificates - easyrsa:client
- add relation kubernetes-worker:kube-api-endpoint - kubeapi-load-balancer:website
- add relation kubeapi-load-balancer:certificates - easyrsa:client
- add relation flannel:etcd - etcd:db
- add relation flannel:cni - kubernetes-master:cni
- add relation flannel:cni - kubernetes-worker:cni
- add unit easyrsa/0 to new machine 0
- add unit etcd/0 to new machine 1
- add unit etcd/1 to new machine 2
- add unit etcd/2 to new machine 3
- add unit kubeapi-load-balancer/0 to new machine 4
- add unit kubernetes-master/0 to new machine 5
- add unit kubernetes-worker/0 to new machine 6
- add unit kubernetes-worker/1 to new machine 7
- add unit kubernetes-worker/2 to new machine 8
Deploy of bundle completed.
```

You can check the deployment status using the following command:

```
 watch --color juju status --color
```

Note that this will give you the default bundle for CDK which is made up of 9 machines, flannel networking and no RBAC. This is based on the default bundle found here: [https://jujucharms.com/canonical-kubernetes/](https://jujucharms.com/canonical-kubernetes/).

For a more tailored build with Canal or Calico, you can use the bundle builder: [https://github.com/juju-solutions/bundle-canonical-kubernetes](https://github.com/juju-solutions/bundle-canonical-kubernetes). This will generate a bundle file, which is just a big piece of yaml which describes the configuration for the entire cluster, similar to an Ansible Playbook or Puppet Manifest.

If you have a custom bundle, you would deploy that using a command like this instead:

```
 juju deploy bundle.yaml
```

Eventually the colours will all turn green and your cluster is good to go. To access the cluster, we need to install the kubectl command line client and copy the kubernetes configuration file over for it to use:

```
 # If this does not work, try adding the --classic option on the end.
 snap install kubectl --classic
```

Next we copy over the configuration file:

```
  juju scp kubernetes-master/0:/home/ubuntu/config ~/.kube/config
```

Finally, using kubectl we can check that kubernetes cluster interaction is possible:

```
Kubernetes master is running at https://34.253.164.197:443

Heapster is running at https://34.253.164.197:443/api/v1/namespaces/kube-system/services/heapster/proxy
KubeDNS is running at https://34.253.164.197:443/api/v1/namespaces/kube-system/services/kube-dns/proxy
kubernetes-dashboard is running at https://34.253.164.197:443/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy
Grafana is running at https://34.253.164.197:443/api/v1/namespaces/kube-system/services/monitoring-grafana/proxy
InfluxDB is running at https://34.253.164.197:443/api/v1/namespaces/kube-system/services/monitoring-influxdb/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'
```

**__Note: There are two other ways CDK would be deployed, either using [Conjure-up](https://tutorials.ubuntu.com/tutorial/install-kubernetes-with-conjure-up#0) or using the graphical juju-as-a-service tool provided at [https://jujucharms.com](https://jujucharms.com).__**

## Deploying the F5 Big-IP Load-Balancer

F5 Network's BigIP Product is shipped either as a physical or virtual device. They also provide an [AWS AMI](https://aws.amazon.com/marketplace/seller-profile?id=74d946f0-fa54-4d9f-99e8-ff3bd8eb2745) on the AWS Market Place which can be used for testing.

As we've just deployed Kubernetes onto AWS, we will spin-up the load-balancer on that platform as well. The AMI I chose is called 'F5 BIG-IP Virtual Edition - GOOD - (Hourly, 200Mbps, v13)'  ([http://aws.amazon.com/marketplace/pp/B079C3WS75?ref=cns_srchrow](http://aws.amazon.com/marketplace/pp/B079C3WS75?ref=cns_srchrow)). Note that F5's licensing model revolves around flavours, Good being the cheapest licence, and best being the most feature-rich offering ([https://www.f5.com/pdf/licensing/good-better-best-licensing-overview.pdf](https://www.f5.com/pdf/licensing/good-better-best-licensing-overview.pdf)).

Depending on which type of licence you have, your load-balancer options may be limited. For example, if you want to directly integrate the F5 Big-IP devices into your Kubernetes network, you will need to have a 'BEST' type licence. More information on this can be found here: [http://clouddocs.f5.com/containers/v2/kubernetes/kctlr-modes.html](http://clouddocs.f5.com/containers/v2/kubernetes/kctlr-modes.html).

The next steps have not yet been automated but could easily be automated with a script and the aws-cli tool or by using Juju, puppet or ansible.

To Launch an instance on AWS, perform the following steps:

- Go to the AWS Console and login: [https://aws.amazon.com](aws.amazon.com)
- Select the correct region for where you deployed your cluster with Juju, in my case it was eu-west-1
- Go to EC2, make sure you have a private key setup and Hit 'Launch Instance'
- Go to AWS Marketplace, Type F5 BIG-IP and pick a flavor. Hit the Select button.
- Use the defaults for the AMI, but make sure you pick one with version latest version (13 or higher) and make sure you pick the right SSH keypair.

Once the load balancer has been spun up, we need to change the admin password:

- SSH to the loadbalancer using the SSH key you setup, you can find the public IP address for the machine by looking at the machine in the EC2 GUI:

```
 ssh admin@34.241.93.33
```

- Next we change the password:

```
admin@(ip-172-31-47-72)(cfg-sync Standalone)(Active)(/Common)(tmos)# modify auth user admin password admin
admin@(ip-172-31-47-72)(cfg-sync Standalone)(Active)(/Common)(tmos)# save sys config
Saving running configuration...
  /config/bigip.conf
  /config/bigip_base.conf
  /config/bigip_user.conf
Saving Ethernet mapping...done
```

**__Note: The default UI will run on port 8443 if only give the F5 device one interface, however, the container expects 443 so we must change it.__**

- We can change the port with a few extra steps:


```
admin@(localhost)(cfg-sync )(INOPERATIVE)(/Common)(tmos)#
admin@(localhost)(cfg-sync )(INOPERATIVE)(/Common)(tmos)# modify auth user admin password admin
admin@(localhost)(cfg-sync )(INOPERATIVE)(/Common)(tmos)# modify sys httpd ssl-port 443
admin@(ip-172-31-41-63)(cfg-sync )(INOPERATIVE)(/Common)(tmos)# save sys config
Saving running configuration...
  /config/bigip.conf
  /config/bigip_base.conf
  /config/bigip_user.conf
Saving Ethernet mapping...done

```

- Type quit after you're done.

Finally, we can now use the web interface. If you didn't add two interfaces to your device and if you didn't change the default port, the load balancer will be available on https://<public-ip-of-F5>:8443. If you have more than one port, it is exposed through 443.

It seems that the container doesn't handle anything else but 443, so it should be changed using the steps above first. You can test connectivity to the web gui using your web browser or wget.

```
  # This should be resolvable now publically, try it in firefox.
  wget https://<your F5 public ip or vip>:443/ --no-check-certificate
```

![f5 big-ip login](https://raw.githubusercontent.com/CalvinHartwell/canonical-kubernetes-f5-bigip/master/images/login.png "F5 Big-IP Login Screen")

Enter your credentials, login and you should see this screen:

![f5 big-ip gui](https://raw.githubusercontent.com/CalvinHartwell/canonical-kubernetes-f5-bigip/master/images/f5-gui.png "F5 Big-IP GUI")

There appears to be several different versions of the Big-IP appliance on AWS for different bandwidth requirements and price ranges. This example uses the cheapest option but it is possible to sign-up for a [free trial of the virtual edition](https://f5.com/products/deployment-methods/virtual-editions) if you're running on your own estate or you can use existing physical F5 hardware running Big-IP.

If you deploy your own load balancer the default credentials may be different. Note that the container work-load will have quite privileged access to your loadbalancer so using a model which is running in production is not recommended until you are more familiar with its operation.

###  Configuring the F5 Big-IP Load-Balancer

Before we configure the F5 Big-IP Load-balancer controller container, we must configure some additional things on the load-balancer.

Note in the top-right handside of the F5 Load-Balancer interface it says Partition: 'common'. Partitions are used to essentially carve-up an F5 device for multiple users or projects. Each partition has its own set of associated F5 objects. The container we deploy on-top of Kubernetes to control the Big-IP device is unable to manage objects within the common partition and a new one must be created called k8s.

From the default GUI page, go to System -> Users -> Partition List

![f5 big-ip new partition](https://raw.githubusercontent.com/CalvinHartwell/canonical-kubernetes-f5-bigip/master/images/f5-gui-partitions.png "F5 Big-IP Partition List")

On the partition list, you should see 'Common', hit the Create button on the right hand side. Fill in the details for the new Partition and call it k8s:

![f5 big-ip create partition](https://raw.githubusercontent.com/CalvinHartwell/canonical-kubernetes-f5-bigip/master/images/f5-gui-new-k8s-partition.png "F5 Big-IP Create Partition")

Finally hit the Finish button and we're ready to go. You should see two partitions now in the partition list. Depending on where you browse around, you can set the Partition in the top-right to k8s, which will filter your view to objects which belong to the k8s partition.

![f5 big-ip partition-list](https://raw.githubusercontent.com/CalvinHartwell/canonical-kubernetes-f5-bigip/master/images/f5-gui-partition-list.png "F5 Big-IP Partition List")

### Removing the F5 Big-IP Load-Balancer

Removing the load-balancer is just as easy as deploying it, just destroy the VM in the AWS EC2 web interface or using the cli-tool. Remember to clean-up any volumes you attached to the VM, any security groups you created and any SSH key-pairs you no longer need. If you've run the rest of the steps, make sure you remove the F5 Big-IP load balancer container from your cluster as well.

### Deploying the F5 Big-IP Load-Balancer Controller on CDK

Most third-party product integrations for Kubernetes are pretty transparent to the cluster itself. They usually function like so:

- Company produces a container for their product which is able to interact with the API(s) provided by their product.
- You deploy the third-party product (in our case, F5 Big-IP Load Balancer) and configure some credentials for the API.
- You deploy the third-party container on your kubernetes cluster for the specific device with the API endpoint and credentials specified.
- The container picks up constructs which are created on Kubernetes and replicates them onto the Load-balancer, such as ingress rule creation.

The F5 Big-IP Controller container functions like this as well. It is an open-source workload ([https://github.com/F5Networks/k8s-bigip-ctlr](https://github.com/F5Networks/k8s-bigip-ctlr)) which interacts with the Kubernetes API and the API of the Big-IP load balancer to automatically configure the load-balancer based on objects created on kubernetes:

![f5 big-ip gui](https://raw.githubusercontent.com/CalvinHartwell/canonical-kubernetes-f5-bigip/master/images/f5-big-ip-k8s.png "F5 Big-IP GUI Kubernetes")

The yaml file cdk-f5-big-ip.yaml included inside this repository describes the deployment of the F5 Big-IP Controller Container on-top of CDK. The example file has been created based on documentation on F5's website: [http://clouddocs.f5.com/products/connectors/k8s-bigip-ctlr/v1.4/](http://clouddocs.f5.com/products/connectors/k8s-bigip-ctlr/v1.4/).

Let's breakdown and examine the yaml:

```
---
apiVersion: v1
kind: Secret
metadata:
  name: bigip-credentials
type: Opaque
data:
  url: NTIuMzAuMjUuMjEz
  username: YWRtaW4=
  password: YWRtaW4=
---
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: bigip-ctlr-serviceaccount
    namespace: default
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: k8s-bigip-ctlr
  namespace: default
spec:
  replicas: 1
  template:
    metadata:
      name: k8s-bigip-ctlr
      labels:
        app: k8s-bigip-ctlr
    spec:
      serviceAccountName: bigip-ctlr-serviceaccount
      containers:
        - name: k8s-bigip-ctlr
          image: "f5networks/k8s-bigip-ctlr"
          env:
            - name: BIGIP_USERNAME
              valueFrom:
               secretKeyRef:
                name: bigip-credentials
                key: username
            - name: BIGIP_PASSWORD
              valueFrom:
               secretKeyRef:
                name: bigip-credentials
                key: password
            - name: BIGIP_URL
              valueFrom:
               secretKeyRef:
                name: bigip-credentials
                key: url
          command: ["/app/bin/k8s-bigip-ctlr"]
          args: ["--running-in-cluster=true",
            "--bigip-url=$(BIGIP_URL)",
            "--bigip-username=$(BIGIP_USERNAME)",
            "--bigip-password=$(BIGIP_PASSWORD)",
            "--bigip-partition=k8s",
            "--namespace=default",
            "--python-basedir=/app/python",
            "--log-level=INFO",
            "--verify-interval=30",
            "--use-node-internal=true",
            "--pool-member-type=nodeport",
            "--kubeconfig=./config"
          ]

```

The first section of this yaml file creates a secret, which is used to store the API end-point and credentials the load-balancer controller container will use to access the API of the Load-balancer device itself. These credentials are simple base64 encoded strings. Let's replace these values:

```
# First generate some new base64 encoded strings
calvinh@ubuntu-ws:~/.ssh$ echo -n "admin" | base64
YWRtaW4=
calvinh@ubuntu-ws:~/.ssh$ echo -n "https://34.241.93.33:8443" | base64
aHR0cHM6Ly8zNC4yNDEuOTMuMzM6ODQ0Mw==
```

You can also decode strings on the command line:

```
  # Decoding the URL from the original example
  calvinh@ubuntu-ws:~/Source/canonical-kubernetes-f5-bigip$ echo -n "aHR0cHM6Ly8xMC4xOTAuMjEuMTQ4Cg==" | base64 --decode
https://10.190.21.148
```

Next we modify the secret yaml and replace the existing values:

```
 # replace username, password and URL in file cdk-f5-big-ip.yaml:
sed -i 's/YWRtaW4=/<BASE64-USERNAME>/g' cdk-f5-big-ip.yaml
sed -i 's/base64password/<BASE64-PASSWORD>/g' cdk-f5-big-ip.yaml
sed -i 's/aHR0cHM6Ly8xMC4xOTAuMjEuMTQ4Cg==/<BASE64-F5-URL>/g' cdk-f5-big-ip.yaml
```

Once you've changed the file, the section secret section should now look like this, your base64 values should be different:

```
---
apiVersion: v1
kind: Secret
metadata:
  name: bigip-credentials
type: Opaque
data:
  url: NTIuMzAuMjUuMjEz
  username: YWRtaW4=
  password: YWRtaW4=
```

Now we're ready to deploy the container, run the kubectl apply -f command like so:

```
calvinh@ubuntu-ws:~/Source/canonical-kubernetes-f5-bigip$ kubectl apply -f cdk-f5-big-ip.yaml
secret "bigip-credentials" created
serviceaccount "bigip-ctlr-serviceaccount" created
deployment "k8s-bigip-ctlr" created
```

Next, we check the pod status and watch for the controller to be deployed:

```
calvinh@ubuntu-ws:~/Source/canonical-kubernetes-f5-bigip$ kubectl get po
NAME                                               READY     STATUS              RESTARTS   AGE
default-http-backend-7bk4g                         1/1       Running             0          14h
k8s-bigip-ctlr-68549fc7d5-wnlsc                    0/1       ContainerCreating   0          10s
nginx-ingress-kubernetes-worker-controller-8mbtz   1/1       Running             0          14h
nginx-ingress-kubernetes-worker-controller-mdxpl   1/1       Running             0          14h
nginx-ingress-kubernetes-worker-controller-qs7g7   1/1       Running             0          14h

...eventually:

calvinh@ubuntu-ws:~/Source/canonical-kubernetes-f5-bigip$ kubectl get po
NAME                                               READY     STATUS    RESTARTS   AGE
default-http-backend-7bk4g                         1/1       Running   0          14h
k8s-bigip-ctlr-68549fc7d5-wnlsc                    1/1       Running   0          1m
nginx-ingress-kubernetes-worker-controller-8mbtz   1/1       Running   0          14h
nginx-ingress-kubernetes-worker-controller-mdxpl   1/1       Running   0          14h
nginx-ingress-kubernetes-worker-controller-qs7g7   1/1       Running   0          14h
```

Now we're ready to utilise the load-balancer through Kubernetes. Note that at the bottom section of the F5 Big-IP Container, there is a section of run time arguments for the F5 Big-IP controller:

```
args: ["--running-in-cluster=true",
  "--bigip-url=$(BIGIP_URL)",
  "--bigip-username=$(BIGIP_USERNAME)",
  "--bigip-password=$(BIGIP_PASSWORD)",
  "--bigip-partition=k8s",
  "--namespace=default",
  "--python-basedir=/app/python",
  "--log-level=INFO",
  "--verify-interval=30",
  "--use-node-internal=true",
  "--pool-member-type=nodeport",
  "--kubeconfig=./config"
]
```

These options are incredibly important for controlling the integration of the two products. It is highly recommended that you read the F5 provided manual to understand these options in more detail ([http://clouddocs.f5.com/products/connectors/k8s-bigip-ctlr/v1.4/](http://clouddocs.f5.com/products/connectors/k8s-bigip-ctlr/v1.4/)).

**__Note: as the --pool-member-type is set to nodeport, we need to set type 'nodeport' to each of the services we create on Kubernetes in order for these to be picked up. More information on this can be found here: [http://clouddocs.f5.com/containers/v2/kubernetes/kctlr-modes.html](http://clouddocs.f5.com/containers/v2/kubernetes/kctlr-modes.html).__**

## Utilising the F5 Big-IP Load Balancer with Kubernetes

Included inside this repository is an example workload which will create objects on the load-balancer. This can be found inside the yaml file called cdk-f5-ingress-example.yaml:

```
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: cdk-cats
  name:  cdk-cats
spec:
  replicas: 1
  selector:
    matchLabels:
      app:  cdk-cats
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app:  cdk-cats
        ima: pod
    spec:
      containers:
      - image: calvinhartwell/cdk-cats:latest
        imagePullPolicy: Always
        name: cdk-cats
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          timeoutSeconds: 30
        resources: {}
      restartPolicy: Always
      serviceAccountName: ""
status: {}
---
apiVersion: v1
kind: Service
metadata:
  name: cdk-cats
  labels:
    app: cdk-cats
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
  selector:
    app: cdk-cats
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: cdk-cats
  namespace: default
  annotations:
    # Provide an IP address from the external VLAN on your BIG-IP device
    virtual-server.f5.com/ip: "10.190.25.70"
    # Specify the BIG-IP partition containing the virtual server
    virtual-server.f5.com/partition: "k8s"

spec:
  backend:
    # The name of the Kubernetes Service you want to expose to external traffic
    serviceName: cdk-cats
    servicePort: 80
```

Essentially this yaml describes the deployment of a single container, service and ingress rule. The ingress rule has annotations added to it which are picked up by the F5 Big-IP Controller container and replicated to the load-balancer automatically. Note, because we are using nodeport, the service has been given the type 'NodePort'. The most important part of this example is the ingress rule at the bottom.

The annotations section is used to inform the Big-IP controller to pick-up information regarding the ingress rule, such as the IP address, port, partition name, etc. A full list of annotations and further examples can be found here in the F5 manual: [http://clouddocs.f5.com/products/connectors/k8s-bigip-ctlr/v1.4/](http://clouddocs.f5.com/products/connectors/k8s-bigip-ctlr/v1.4/).

To deploy the workload, all we need to do is run the following command:

```
  # file is included in this repo.
  kubectl apply -f cdk-f5-big-ip.yaml
```

This will cause Kubernetes to pull and run the container by creating a deployment. It will also create the service and the ingress rule.  If you tail the logs of your Big-IP Controller container logs, you should see messages like this:

```
calvinh@ubuntu-ws:~$ kubectl get po
NAME                                               READY     STATUS    RESTARTS   AGE
cdk-cats-8575fffbc-z7jzg                           1/1       Running   0          41m
default-http-backend-7bk4g                         1/1       Running   0          2d
k8s-bigip-ctlr-68549fc7d5-v7pd9                    1/1       Running   0          41m
nginx-ingress-kubernetes-worker-controller-8mbtz   1/1       Running   0          2d
nginx-ingress-kubernetes-worker-controller-mdxpl   1/1       Running   0          2d
nginx-ingress-kubernetes-worker-controller-qs7g7   1/1       Running   0          2d
calvinh@ubuntu-ws:~$ kubectl logs  -f k8s-bigip-ctlr-68549fc7d5-v7pd9
2018/03/14 21:07:26 [INFO] Starting: Version: v1.4.2, BuildInfo: n829-346558842
2018/03/14 21:07:26 [INFO] ConfigWriter started: 0xc420287f80
2018/03/14 21:07:26 [INFO] Started config driver sub-process at pid: 13
2018/03/14 21:07:26 [INFO] NodePoller (0xc42008a7e0) registering new listener: 0x407050
2018/03/14 21:07:26 [INFO] NodePoller started: (0xc42008a7e0)
2018/03/14 21:07:26 [WARNING] Overwriting existing entry for backend {ServiceName:cdk-cats ServicePort:80 Namespace:default}
2018/03/14 21:07:26 [INFO] Wrote 0 Virtual Server and 0 IApp configs
2018/03/14 21:07:26 [INFO] [2018-03-14 21:07:26,769 __main__ INFO] entering inotify loop to watch /tmp/k8s-bigip-ctlr.config017314497/config.json
2018/03/14 21:07:44 [INFO] Port '80' for service 'cdk-cats' was not found.
2018/03/14 21:07:44 [INFO] Service 'cdk-cats' has not been found.
2018/03/14 21:07:44 [INFO] Wrote 0 Virtual Server and 0 IApp configs
2018/03/14 21:07:44 [INFO] Port '80' for service 'cdk-cats' was not found.
2018/03/14 21:07:44 [INFO] Service 'cdk-cats' has not been found.
2018/03/14 21:07:44 [INFO] Wrote 0 Virtual Server and 0 IApp configs
2018/03/14 21:07:53 [INFO] Wrote 0 Virtual Server and 0 IApp configs
```

This indicates that the Big-IP controller container has picked up the ingress rule and replicated onto the load-balancer. We can check that by logging onto the F5 Load-Balancer interface and going to the virtual server list. Make sure your partition is set to k8s in the top-right hand corner, otherwise you may not be able to see the objects:

![f5 big-ip k8s virtual-server](https://raw.githubusercontent.com/CalvinHartwell/canonical-kubernetes-f5-bigip/master/images/f5-virtual-server-created.png "F5 Big-IP k8s virtual server list")

As you can see, the name (ingress_10-190-25-70_80) for the newly created Virtual Server has been generated based on the IP address specified as the 'virtual-server.f5.com/ip' annotation and the servicePort which has been set to 80. If you click into the virtual server you can check that the rule matches your specification.

The controller container has also created a pool list for us automatically. Using the GUI on the left-hand side, click the Pools tab and go to Pool List. You should see a newly created pool based on the name of your ingress rule:

![f5 big-ip k8s pool-list](https://raw.githubusercontent.com/CalvinHartwell/canonical-kubernetes-f5-bigip/master/images/f5-gui-pool-list.png "F5 Big-IP k8s pool list")

### Removing the F5 Big-IP Controller Container

If you wish to remove the F5 Big-IP Controller from the cluster, we can do it using kubectl. Deleting constructs in Kubernetes is as simple as creating them:

```
  # If you used the nodeport example change the yaml filename if you used the ingress example.
  kubectl delete -f cdk-f5-big-ip.yaml
```

### Troubleshooting the F5 Big-IP Controller

F5 have provided a fully fledged Troubleshooting guide here: [http://clouddocs.f5.com/containers/v2/troubleshooting/kubernetes.html](http://clouddocs.f5.com/containers/v2/troubleshooting/kubernetes.html)

## Conclusion

We have covered deploying Canonical Kubernetes and integrating it with the F5 Big-IP Load-Balancer. A simple example has been provided to demonstrate how this integration works.

More examples are planned, including how to use the VXLAN functionality with Flannel and Canal.

### Software Versions

The software versions used throughout this docunmentation are:
 - Kubernetes 1.9.3
 - F5 Big-IP Version 13
 - F5 Big-IP Controller Container version 1.4

### Getting Help & Support

This document was written by [Calvin Hartwell](https://www.linkedin.com/in/calvinhartwell), please feel to drop me an email on [calvin.hartwell@canonical.com](calvin.hartwell@canonical.com).

To open issues with CDK, please open an issue with the repository here [https://github.com/juju-solutions/bundle-canonical-kubernetes](https://github.com/juju-solutions/bundle-canonical-kubernetes). If you want professional support for Kubernetes please contact Canonical Sales [https://www.ubuntu.com/kubernetes](https://www.ubuntu.com/kubernetes).

F5 provides support through [Slack](https://f5cloudsolutions.herokuapp.com/) and through paid support. Please contact F5 directly to get more help with their product. You can also open issues on the F5 Big-IP controller itself [https://github.com/F5Networks/k8s-bigip-ctlr/issues](https://github.com/F5Networks/k8s-bigip-ctlr/issues).

### Useful Links
- [https://github.com/F5Networks/k8s-bigip-ctlr](https://github.com/F5Networks/k8s-bigip-ctlr)
- [http://clouddocs.f5.com/containers/v2/index.html](http://clouddocs.f5.com/containers/v2/index.html)
- [http://clouddocs.f5.com/products/connectors/k8s-bigip-ctlr/v1.4/](http://clouddocs.f5.com/products/connectors/k8s-bigip-ctlr/v1.4/)
- [http://clouddocs.f5.com/containers/v2/kubernetes/kctlr-modes.html](http://clouddocs.f5.com/containers/v2/kubernetes/kctlr-modes.html)
- [http://clouddocs.f5.com/containers/v2/kubernetes/flannel-bigip-info.html](http://clouddocs.f5.com/containers/v2/kubernetes/flannel-bigip-info.html)
- [http://clouddocs.f5.com/containers/v2/kubernetes/kctlr-use-bigip-k8s.html](http://clouddocs.f5.com/containers/v2/kubernetes/kctlr-use-bigip-k8s.html)
- [http://clouddocs.f5.com/containers/v2/kubernetes/kctlr-manage-bigip-objects.html](http://clouddocs.f5.com/containers/v2/kubernetes/kctlr-manage-bigip-objects.html)
- [http://clouddocs.f5.com/containers/v2/kubernetes/kctlr-app-install.html](http://clouddocs.f5.com/containers/v2/kubernetes/kctlr-app-install.html)
- [http://clouddocs.f5.com/containers/v2/kubernetes/kctlr-ingress.html](http://clouddocs.f5.com/containers/v2/kubernetes/kctlr-ingress.html)
- [https://kubernetes.io/docs/getting-started-guides/ubuntu/installation/](https://kubernetes.io/docs/getting-started-guides/ubuntu/installation/)
- [http://clouddocs.f5.com/containers/v2/kubernetes/kctlr-manage-bigip-objects.html](http://clouddocs.f5.com/containers/v2/kubernetes/kctlr-manage-bigip-objects.html)
- [http://clouddocs.f5.com/containers/v2/kubernetes/kctlr-ingress.html](http://clouddocs.f5.com/containers/v2/kubernetes/kctlr-ingress.html)
- [https://github.com/juju-solutions/bundle-canonical-kubernetes](https://github.com/juju-solutions/bundle-canonical-kubernetes)
- [https://github.com/juju-solutions/bundle-canonical-kubernetes/wiki/Authorization-Mode-and-RBAC](https://github.com/juju-solutions/bundle-canonical-kubernetes/wiki/Authorization-Mode-and-RBAC)
- [https://f5.com/products/deployment-methods/hardware](https://f5.com/products/deployment-methods/hardware)
