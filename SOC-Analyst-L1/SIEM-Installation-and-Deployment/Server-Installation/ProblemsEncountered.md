**Problem 0: No connection on my Ubuntu VM after installation**


Since the `ipconfig` command wasn't working here, I had to:

1- Update my VM

2- Run the command “apt install net-tools” to install all the network packages

3- I verified that everything was working properly with the “ifconfig” command


**Problem 1: Loss of Internet connectivity on the monitoring VM (Ubuntu)**

Symptom: After switching the Ubuntu VM to the VMnet8 virtual switch to segment my network and assign it a static IP address, the machine lost all Internet access, making it impossible to update packages or download security agents.

Analysis: VMware's VMnet8 network uses NAT mode, which includes a virtual gateway by default to route traffic to the outside. The blockage was caused by an internal configuration error in the VM's network file (***/etc/netplan/01-network-manager-all.yaml***): I had instinctively set the gateway to the .1 address, whereas the VMware hypervisor assigns the .2 address to its virtual NAT gateway by default. As a result, packets destined for the Internet could not find an exit path.

Solution: To maintain logical isolation from the other machines in the lab while retaining access to the update repositories, I corrected the network configuration of the Ubuntu VM. I aligned its static IP address with the VMnet8 range and corrected the routing directive by replacing the incorrect gateway with the exact IP address of the VMware NAT component (192.168.X.2). After applying the changes (***netplan apply***), external connectivity was immediately restored


**Problem 2 : Failed to connect to the indexer**

Symptom: After installing the indexer and initializing the cluster, the initialization test fails and returns the error message: "*Connectivity check failed on node 192.168.X.X port 9200. Possible causes: The Wazuh indexer is not installed on the node, the Wazuh indexer service is not running, or you are experiencing connectivity issues with that node. Please check this before trying again*."

Analysis: The indexer's default port (9200) may not be open in the VM's firewall, which is blocking communication. After checking, I found that the block was caused by an incorrect configuration of my VM, which prevented the indexer from installing and, consequently, the cluster from initializing: the required configurations were not met when Ubuntu was created.

Solution: To resolve the issue, I enabled the firewall and allowed communication on port 9200. Before allowing the installation of the indexer and any other core components (dashboard and manager), I increased my VM's resources to 4 GB of RAM and 2 CPUs.


**Problème 3 :** 

