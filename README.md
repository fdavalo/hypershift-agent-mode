# hypershit-agent-mode

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



## Create a Virtual Machine on Vsphere to be used as template for nodes

   The virtual machine will not be started, only created to be cloned as template
   
[![Create a virtual machine on vsphere](https://github.com/fdavalo/mce-agent-provision-vms/blob/main/vsphere-vm-template.png?raw=true)](vsphere-vm-template.png)

## Clone the Virtual Machine as a template
   
[![Clone the virtual machine as a template on vsphere](https://github.com/fdavalo/mce-agent-provision-vms/blob/main/vsphere-template.png?raw=true)](vsphere-template.png)

## Use the Tekton Pipeline to create the Virtual Machine
   
   The Pipeline is started and asking for a Workspace (persistent volume to share files between tasks), choose a volume claim template

   The Tekton pipeline consists of 3 steps :

* Clone this git repository (fetch terraform template and bash scripts)

* Use terraform to create the Virtual Machine (random name is used, a cdrom is used to boot the discovery iso)

* A bash script will fetch Virtual Machine name and IP to update the Agent (approved=true, hostname=vm name)
