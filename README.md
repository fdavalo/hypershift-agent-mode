# Create an Hypershift cluster with Agent mode for Workers

## Provision nodes on an Infrastructure Env (use Discovery ISO or PXE Boot mode)

   You need at least 2 worker nodes for your cluster
   
   They have to be ready in Available status

[![See the Hosts Available](https://github.com/fdavalo/mce-agent-provision-vms/blob/main/agent-vsphere.png?raw=true)](agent-vsphere.png)

## Create an Hypershift cluster with the Advanced Cluster Management UI

   Go on Tab Infrastructure/Clusters on Advanced Cluster Management UI
   
   Click on Create Button

[![Create Button](https://github.com/fdavalo/hypershift-agent-mode/blob/main/hypershift-create-button.png?raw=true)](hypershift-create-button.png)

   Choose Host Inventory as cluster provider (for agent mode)

[![Host Inventory](https://github.com/fdavalo/hypershift-agent-mode/blob/main/hypershift-create-choose-hostinventory.png?raw=true)](hypershift-create-choose-hostinventory.png)

   Choose Hosted cluster (means control plane is hosted on the hub cluster)

[![Hosted Cluster](https://github.com/fdavalo/hypershift-agent-mode/blob/main/hypershift-create-choose-hosted.png?raw=true)](hypershift-create-choose-hosted.png)

   Fill form page for main cluster information

* Credential Provider : where you provide a pull secret (credentials to access redhat registries)

* Name for the cluster

* Cluster-Set : where you can automate day2 customization, the cluster-set can be linked to installation and configuration of components in your cluster

* Base Domain : used for dns and certificates

* Openshift Version : recommendation to use the same or lower version than the hub Openshift -> here choose 4.11.xxx

[![Main Page](https://github.com/fdavalo/hypershift-agent-mode/blob/main/hypershift-create-ui-details.png?raw=true)](hypershift-create-ui-details.png)

   Find at least two nodes for your cluster

* Namespace : name of Infrastructure Environment where you should find Agent CRD

* You will able to select en amount of nodes up to available nodes in the namespace

[![Nodes Page](https://github.com/fdavalo/hypershift-agent-mode/blob/main/hypershift-create-ui-nodepools.png?raw=true)](hypershift-create-ui-nodepools.png)

   Fill network information

* API server publishing strategy : 

  ** not for testing, use a LoadBalancer (cloud provider, metalLB, front VIP)
        
  ** not for testing, use nodeport and provide the IP of a front VIP that will loadbalance on cluster nodes and the chosen nodeport

  ** for testing, use nodeport and just provide the ip of a node from the hub cluster and let the installer choose an available nodeport
        
* Machine CIDR : choose the subnet where worker nodes main IP should be used 

[![Network Page](https://github.com/fdavalo/hypershift-agent-mode/blob/main/hypershift-create-ui-network.png?raw=true)](hypershift-create-ui-network.png)

   Review page before starting the cluster creation

[![Review Page](https://github.com/fdavalo/hypershift-agent-mode/blob/main/hypershift-create-ui-review.png?raw=true)](hypershift-create-ui-review.png)

   First step, creation of control plane pods on the Hub cluster
   
   Nodes are bound to the cluster and installation is just starting

[![Start Page](https://github.com/fdavalo/hypershift-agent-mode/blob/main/hypershift-create-starting-nodes.png?raw=true)](hypershift-create-starting-nodes.png)

   Control plane pods are almost all ready
   
   Worker nodes are installing and updating

[![Installed Page](https://github.com/fdavalo/hypershift-agent-mode/blob/main/hypershift-create-installed.png?raw=true)](hypershift-create-installed.png)

   The last control plane pods appear lately (ovnkube master when worker nodes are ready) : around 10 minutes duration
   
   The control plane can be changed to "HiglyAvailable" instead of "SingleReplica" in the HostedCluster CRD or during the creation in the CRD edit tab
   
[![Control Plane Pods](https://github.com/fdavalo/hypershift-agent-mode/blob/main/hypershift-control-plane-pods.png?raw=true)](hypershift-control-plane-pods.png)

   The cluster is ready
   
[![Final Page](https://github.com/fdavalo/hypershift-agent-mode/blob/main/hypershift-create-final.png?raw=true)](hypershift-create-final.png)

   Add the hypershift cluster wildcard DNS to be able to access the cluster console
   
   You can add the wildcard for each worker node IP
   
             *.apps.hycl3 IN A 10.6.82.226
             *.apps.hycl3 IN A 10.6.82.232

   Or you can create a front loadbalancer with the widlcard DNS pointing to it and that loadbalance to the worker nodes

   Now, you can see in the hypershift console that only worker nodes appear, there is no more control plane nodes
   
[![Worker nodes](https://github.com/fdavalo/hypershift-agent-mode/blob/main/hypershift-cluster-nodes.png?raw=true)](hypershift-cluster-nodes.png)

 ## CRD examples
 
                     apiVersion: hypershift.openshift.io/v1alpha1
                     kind: HostedCluster
                     metadata:
                       name: 'hycl1'
                       namespace: 'hycl1'
                       labels:
                     spec:
                       release:
                         image: quay.io/openshift-release-dev/ocp-release:4.11.27-x86_64
                       pullSecret:
                         name: cluster-hycl1-pullsecret
                       sshKey:
                         name: cluster-hycl1-sshkey
                       networking:
                         podCIDR: 10.132.0.0/14
                         serviceCIDR: 172.31.0.0/16
                         machineCIDR: 10.6.82.0/24
                       platform:
                         type: Agent
                         agent:
                           agentNamespace: 'baremetal'
                       infraID: 'hycl1'
                       dns:
                         baseDomain: 'redhat.hpecic.net'
                       services:
                       - service: APIServer
                         servicePublishingStrategy:
                           type: NodePort
                           nodePort:
                             address: 10.6.82.55
                             port: 0
                       - service: OAuthServer
                         servicePublishingStrategy:
                           type: Route
                       - service: OIDC
                         servicePublishingStrategy:
                           type: Route
                       - service: Konnectivity
                         servicePublishingStrategy:
                           type: Route
                       - service: Ignition
                         servicePublishingStrategy:
                           type: Route


