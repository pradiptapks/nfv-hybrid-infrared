# OpenStack NFV Hybrid Infrastructure deployment with Infrared v2.0

<!-- wp:heading {"level":4} -->
<h4>InfraRed Intro</h4>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>The <strong>InfraRed</strong> project originated by Red Hat OpenStack infrastructure team focuses to provide an “easier” method for installing OpenStack from CLI. InfraRed refers to Ansible based poject that aims to provide an easy-to-use CLI for OpenStack deployment with advanced modules and v2.0 allows deploying hybrid cloud which includes virtual nodes and bare metal nodes.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","width":580,"height":415,"linkDestination":"custom"} -->
<div class="wp-block-image"><figure class="aligncenter is-resized"><a href="https://i1.wp.com/abregman.com/wp-content/uploads/2017/03/infrared.png"><img src="https://i1.wp.com/abregman.com/wp-content/uploads/2017/03/infrared.png?resize=676%2C484" alt="" width="580" height="415"/></a></figure></div>
<!-- /wp:image -->

<!-- wp:heading {"level":4} -->
<h4 id="jive_content_id_RHOSP_14_NFVi_Deployment"><strong>RedHat&nbsp;OpenStack&nbsp;NFV&nbsp;Infrastructure</strong></h4>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Network Functions Virtualization (NFV) uses virtualization to move network node functions into building blocks that interconnect to create communication services.  NFV is a new way to define, create and manage networks by replacing dedicated hardware appliances with software and automation. OpenStack provides NFV infrastructure with features that are modular, high performance and opensource.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","linkDestination":"custom"} -->
<div class="wp-block-image"><figure class="aligncenter"><a href="https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/14/html-single/network_functions_virtualization_product_guide/#sect-nfv-architecture-components"><img src="https://access.redhat.com/webassets/avalon/d/Red_Hat_OpenStack_Platform-10-Network_Functions_Virtualization_Product_Guide-en-US/images/32804f727be072df52211f4bafd1e85c/OpenStack_NFV_Reference_Arch_422691_1116_JCS.png" alt=""/></a></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>These following official links briefly discuss Red Hat’s effort to accelerate NFV  deployments using the Red Hat OpenStack Platform.</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li><a href="https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/14/html-single/release_notes/#network-functions-virtualization">Red Hat OpenStack Release Note</a></li><li><a href="https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/14/html-single/network_functions_virtualization_product_guide/">Red Hat OpenStack NFVi Product Guide</a></li><li><a href="https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/14/html-single/network_functions_virtualization_planning_and_configuration_guide/">Red Hat OpenStack NFVi Configuration Guide</a></li><li></li></ul>
<!-- /wp:list -->

<!-- wp:heading {"level":4} -->
<h4 id="jive_content_id_NFV_Hybrid_Hardware_requirement"><strong>NFVi Hybrid requirement</strong></h4>
<!-- /wp:heading -->

<!-- wp:list -->
<ul><li>KVM host built with Infrared modules which carry the Virtual control plane traffic.</li><li>Bare metal nodes specifically for high-performance data traffic (e.g., <strong>SRIOV, OVS-DPDK, RT KVM</strong>).</li><li>Ensure all the underlying network configuration for KVM host and bare-metal compute which are part of OpenStack deployment are properly established.</li></ul>
<!-- /wp:list -->

<!-- wp:quote {"className":"is-style-default"} -->
<blockquote class="wp-block-quote is-style-default"><p><strong>Note</strong>: If the Overcloud control plane VMs split into multiple KVM host then each KVM has configured with Infrared with similar underlaying ansible network playbook.</p></blockquote>
<!-- /wp:quote -->

<!-- wp:heading {"level":4} -->
<h4><strong>Infrared Deployment</strong></h4>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>There are some manual steps to prepare a KVM server for infrared deployment.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>First, provision a KVM server with RHEL 7.6 Server. When the KVM server is ready, log to it as root and update the host with the libvirt package.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># yum -y update
# yum -y install libvirt libvirt-python</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>By default, KVM host allocates all the space in /home, which is not ideal for libvirt. On the KVM system, log in as root and adjust libvirt storage:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># cd /var/lib/libvirt
# rm -rf images
# mkdir /home/images
# chcon -t virt_image_t /home/images
# ln -s /home/images .</code></pre>
<!-- /wp:code -->

<!-- wp:quote {"className":"is-style-default"} -->
<blockquote class="wp-block-quote is-style-default"><p><strong>Note: </strong>For IPv6 usecase, load the ipv6 module and edit the file with the following information (replace with the right network interface, the one that is connected to the external network) :</p></blockquote>
<!-- /wp:quote -->

<!-- wp:code -->
<pre class="wp-block-code"><code># modprobe nf_reject_ipv6
# vi /etc/sysctl.d/98-ipv6.conf
net.ipv6.conf.em2.accept_ra = 2

Alternative command:
# sysctl -w net.ipv6.conf.em2.accept_ra=2</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Following RPM need to install in KVM for Infrared deployment:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># sudo yum install git gcc libffi-devel openssl-devel python-virtualenv libselinux-python redhat-rpm-config</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Prepare a virtual environment for infrared:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># virtualenv ~/.venv_infrared
# source ~/.venv_infrared/bin/activate
# pip install --upgrade pip
# pip install --upgrade setuptools

# cd ~/.venv_infrared
# git clone https://github.com/redhat-openstack/infrared.git
# cd infrared
# pip install .
# echo ". $(pwd)/etc/bash_completion.d/infrared" >> ${VIRTUAL_ENV}/bin/activate</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Infrared v2.0 should now be ready on your KVM host.</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":3} -->
<h3><strong>Infrared Workspace Creation</strong></h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>With workspace,  infrared can manage multiple environments and also alternate between them.  All runtime files (Inventory, hosts, ssh configuration, ansible.cfg,  etc…) will be loaded from a workspace directory and all output files  (Inventory, ssh keys, environment settings, facts caches, etc…) will also be generated into the same directory.</p>
<!-- /wp:paragraph -->

<!-- wp:quote {"className":"is-style-default"} -->
<blockquote class="wp-block-quote is-style-default"><p><strong>Note:</strong> In this deployment, I am using one KVM host so I prefer to use one workspace with name  "nfv-lab1".<br><br>But for any NFVi split environment where KVM host manages multiple overcloud environment, it is required to create multiple infrared workspaces to manage and isolate the environment in an effective way.</p><cite><br></cite></blockquote>
<!-- /wp:quote -->

<!-- wp:code -->
<pre class="wp-block-code"><code>Example:
# infrared workspace create nfv-lab1</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>The create option will not switch to the newly created workspace. In order to switch to  the new workspace, the checkout command should be used</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># infrared workspace checkout nfv-lab1</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Creates a new workspace if the --create or -c is specified and switches to it:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>infrared workspace checkout --create nfv-lab1</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Checked out workspace is tracked via a status file in workspaces_dir, which means checked out workspace is persistent across shell sessions.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># infrared workspace list
+--------------------------------------+--------+
| Name                                 | Active |
+--------------------------------------+--------+
| nfv-lab1                             |    *   |
+--------------------------------------+--------+
| workspace_2019-01-14_15-18-03        |        |
+--------------------------------------+--------+</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Workspace delete: To delete any workspace refer following ir command:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># infrared workspace delete nfv-lab1</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Workspace Cleanup: Removes all the files from the workspace. Unlike delete:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># infrared workspace cleanup nfv-lab1</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":3} -->
<h3 id="jive_content_id_3_Create_a_network_topology_for_virtual_enviroment_"><strong>Create a network topology for the Infrared virtual environment </strong></h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>In infrared most common configuration can include for 3 bridged networks, but you can modify the network based on NFVi usecase. Before creating NFVi environment, you need to decide the number and types of networks to create.<br>In this NFVi hybrid topology, the following virtual bridge with PCI uplink is configured to communicate with bare-metal nodes via TOR switch.<br></p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li><strong>br-ctlplane: </strong>This network is mainly useful to provision virtual and bare metal nodes. In this scenario, I used physical uplink to the bridge.</li><li> <strong>br-data: </strong>This network act as a trunk to configure network isolation for OpenStack management &amp; tenant network</li><li><strong>br-ex</strong>: This network is a useful external network for floating IP communication.</li><li></li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>The below format is used to provide topology networks: <br>Example:<br></p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># cat &lt;&lt; EOF > /root/.venv_infrared/infrared/plugins/virsh/vars/topology/network/custom_nets.yml

---
networks:
    net1:
        name: "br-ctlplane"
        forward: bridge
        nic: em2
    net2:
        name: "br-data"
        forward: bridge
        nic: em3
    net3:
        external_connectivity: yes
        name: "br-ex"
        ip_address: "172.16.0.1"
        netmask: "255.255.255.0"
        forward: nat
        dhcp:
            range:
                start: "172.16.0.2"
                end: "172.16.0.100"
            subnet_cidr: "172.16.0.0/24"
            subnet_gateway: "172.16.0.1"
        floating_ip:
            start: "172.16.0.101"
            end: "172.16.0.150"
nodes:
    undercloud:
        interfaces:
             - network: "br-ctlplane"
              bridged: yes
            - network: "br-ex"
              bridged: yes
        external_network:
            network: "br-ex"
    controller:
        interfaces:
            - network: "br-ctlplane"
              bridged: yes
            - network: "br-data"
              bridged: yes
        external_network:
            network: "br-ctlplane"
EOF</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Ensure the above template should reside in the following directory to be recognized by infrared network-topology plugin:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>/root/.venv_infrared/infrared/plugins/virsh/vars/topology/network/</code></pre>
<!-- /wp:code -->

<!-- wp:quote {"className":"is-style-default"} -->
<blockquote class="wp-block-quote is-style-default"><p><strong>Network topology limitation</strong><br>If the host bridge map with PCI uplink Infrared default plugin unable to delete the network while performing a cleanup, so after infra cleanup, you need to manually delete the network for redeploying the usecase.<br>In my NFVI lab redeployment, I prefer to use below command to delete the existing bridges after ir cleanup.</p><p><strong>Hints: </strong>But the below manual task can be accomplished with Ansible modules.</p></blockquote>
<!-- /wp:quote -->

<!-- wp:code -->
<pre class="wp-block-code"><code>#ifconfig br-ctlplane down; brctl delif br-ctlplane em2; brctl delbr br-ctlplane;
#ifconfig br-data down; brctl delif br-data em3; brctl delbr br-data;</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":3} -->
<h3 id="jive_content_id_4_Provision_Director_and_Controller_VM">Provision Virtual RHOSP Director and Overcloud Controller</h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Infrared virsh provisioner is explicitly designed to be used for setup of virtual environments. Such environments are used to emulate production environment like Director and overcloud controller VMs running on one bare metal machine. The first thing we need to decide before deploy environment with node topology which refers to the number and type of VMs in your desired deployment environment.  For OpenStack as an example, a topology may look something like:</p>
<!-- /wp:paragraph -->

<!-- wp:list {"ordered":true} -->
<ol><li>Undercloud</li><li>Overcloud Controller</li><li>Overcloud Compute</li><li>Overcloud Composable nodes (eg., Networker, Storage, Ceph, ODL, OVN)</li></ol>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>In NFVi hybrid deployment, we prefer to provision Director &amp; Controller as virtual machine manages by infrared with appropriate resource allocation. While provision the virtual instance, ensure you should map the <strong>--topology-network</strong> with the custom network which is created in the previous step. In this way, the virtual nodes are aligned with the respective network.<br>Example:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># infrared virsh -vvvv --host-address  nfv-lab1 \
        --host-key ~/.ssh/id_rsa \
        --topology-nodes undercloud:1,controller:3 \
        --host-memory-overcommit True \
        -e override.undercloud.memory=16000 -e override.undercloud.disks.disk1.size=80G \
        -e override.controller.memory=10000 -e override.controller.cpu=15 -e override.controller.disks.disk1.size=100G \
        --topology-network custom_nets.yml \
        --disk-pool=/home/images/ \
        --image-url file:///root/rhel-server-7.6-x86_64-kvm.qcow2;</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":3} -->
<h3 id="jive_content_id_5_UnderCloud_Deployment_by_Infrared">Tripleo Undercloud installation by Infrared</h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>In RedHat OpenStack 14, all openstack services running as a container in Director. Hence, we can choose the respective satellite and container registry details to automate undercloud installation via Infrared. In the below example I used the local satellite and container registry server to install the undercloud.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># infrared tripleo-undercloud -vvvv --version 14 \
        --images-task=rpm \
        --build GA \
        --buildmods cdn \
        --cdn /root/undercloud_cdn.yml \
        --config-file /root/undercloud.conf \
        --validate True \
        --registry-skip-puddle True \
        --registry-tag-discover True \
        --registry-namespace server-xxx-xx.xxx.xxx.redhat.com:5000 \
        --registry-mirror server-xxx-xx.xxx.xxx.redhat.com:5000 \
        --registry-prefix red_hat_xxx-osp14_containers- \
        --post True \
        --images-cleanup True</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":6} -->
<h6>Following is the description of the plugin that has been used for undercloud deployment:</h6>
<!-- /wp:heading -->

<!-- wp:code -->
<pre class="wp-block-code"><code>--version VERSION     The product version (product == director), Numbers are for ROSP releases

--build BUILD         String represents rhos-release labels. Example: GA, Z1, Z2, Z3 etc.

--buildmods BUILD MODE
List of flags for the rhos-release module. Currently, this parameter works with 
pin - pin puddle (dereference 'latest' links to prevent content from changing)  flea - enable flea repos  unstable - this will enable brew repos or poodles (in old releases)
cdn - use internal mirrors of the CDN repos. (internal use)   none - use none of those flags
Default value: 'pin'.

--images-task IMAGES-TASK
Specifies the source for the OverCloud images:
     * rpm - packaged with the product
     * import - Download pre-built images from a given source (versions 12, 13, 14). Requires '--images-url'.
     * build - build images locally (takes longer) on top of regular cloud guest image.
CentOS/RHEL will be used for RDO/OSP. Allowed values: ['rpm', 'import', 'build'].

--cdn         
 YAML file. Register the undercloud with a Red Hat Subscription Management platform.
      Example:
       server_hostname: example.redhat.com
       username: user
       password: xxxxx
       autosubscribe: yes
       server_insecure: yes

--config-file CONFIG-FILE
 A path to a custom undercloud.conf file to use for deployment. If not set, it will look under template the path for a file named undercloud.conf.
 If no undercloud.conf file found it will use the default /usr/share/instack-undercloud/undercloud.conf.sample that is provided by the installation.

--validate VALIDATE  
 Specifies whether we should run pre-validation tasks. Default value: 'True'.

--registry-skip-puddle REGISTRY-SKIP-PUDDLE
 Skip reading any private puddle files to auto-detect the containers parameters. Default value: 'False'.

--registry-tag-discover REGISTRY-TAG-DISCOVER
 If this option is set then infrared will try to auto-discover tag. Default value: 'False'.

--registry-namespace REGISTRY-NAMESPACE
 The alternative docker registry namespace to use for undercloud deployment. Default value: 'rhosp14'.

--registry-mirror REGISTRY-MIRROR
 The alternative docker registry to use for undercloud deployment.

--registry-prefix REGISTRY-PREFIX
 Container images prefix. Default value: 'openstack-'.

--post POST          
 Specifies whether we should run post-install tasks. Default value: 'True'.

--images-cleanup IMAGES-CLEANUP
 Removes all the downloaded images when images-task is in 'rpm' or 'import'. Default value: 'True'.</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Note: To get more infrared undercloud plugin parameter, refer <strong>infrared tripleo-undercloud --help</strong></p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":3} -->
<h3 id="jive_content_id_6_Introspect_Virtual_Controller_and_Physical_Compute_bare_metal">Introspect Virtual Controller and Physical Compute bare metal</h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Create a JSON file which lists all the bare metal nodes required for deployment. In this  JSON file, you don't need to append the controller node details,  Infrared leverage to append the controller resource details during the node introspection time.<br>Example:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>cat &lt;&lt; EOF > hybrid_nodes.json
{
   "nodes": [
     {
        "name": "compute-0",
        "pm_addr": "baremetal-mgmt.redhat.com",
        "mac": ["00:00:00:00:00:00"],
        "arch": "x86_64",
         "pm_type": "pxe_ipmitool",
        "pm_user": "admin",
        "pm_password": "admin",
        "cpu": "1",
        "memory": "4096",
        "disk": "40"
     }]
}
EOF</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Refer following example to define the provisioning network and json file for introspection.<br>Example:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># infrared tripleo-overcloud -vvvv --version 14 \
        --pre True \
        --deployment-files virt  \
        --introspect True \
        --tagging False \
        --deploy=no \
        -e provison_virsh_network_name=br-ctlplane \
        --hybrid /root/compute_instackenv.json</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":5} -->
<h5>Reference of parameter details:</h5>
<!-- /wp:heading -->

<!-- wp:code -->
<pre class="wp-block-code"><code>--pre PRE            
Specifies whether we should run pre-install tasks. Default value: 'False'.

--deployment-files DEPLOYMENT-FILES
In introspection, infrared will not actually referred the deployment files. So, I used the existing virt template directory.

--introspect INTROSPECT
Specifies whether to run introspection. Default value: 'False'.

--tagging TAGGING    
Specifies whether to create flavors automatically and tag our hosts with them. Default value: 'False'.

--deploy DEPLOY      
Specifies whether to deploy the overcloud. Default value: 'False'.</code></pre>
<!-- /wp:code -->

<!-- wp:quote -->
<blockquote class="wp-block-quote"><p><strong>Note:</strong><br> <strong>-e provison_virsh_network_name=br-ctlplane</strong><br> To introspect overcloud nodes undercloud requires a provisioning network. So, it requires to explicitly choose br-ctlplane network in undercloud for node introspection.<br> <strong>--tagging TAGGING </strong><br>Default value is True which will point to the default role files (roles_data.yaml) in Director to tag the profiles to overcloud nodes. Currently, there is no infrared ansible module available for custom composable role. For any custom composable role name you can create a shell to tag the profile with overcloud nodes. Else, you can modify the existing default role files (roles_data.yaml) in Director with the appropiate service name (eg., SRIOV, ODL, OVN).</p></blockquote>
<!-- /wp:quote -->

<!-- wp:heading {"level":3} -->
<h3 id="jive_content_id_7_Overcloud_Deployment">Overcloud Deployment</h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>NFVi underlying network configuration does not rely on the default Infrared network topology. Hence hybrid deployment of this step is important since one-time effort is required to design the overcloud template w.r.t composable profiles and network infrastructure. In RHOSP14, I designed my OOO template referring <a href="https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/14/html-single/network_functions_virtualization_planning_and_configuration_guide/#appe-sample-ovsdpdk-sriov-files">Red Hat NFVi configuration guide</a>.<br><br>In Infrared host, create a template directory (e.g, nfvi-14) and copy all the required templates and overcloud deployment script into this directory. Once the template is prepared, copy template folder to the "/root/.venv_infrared/plugins/tripleo-overcloud/vars/deployment/files/" and in CLI use --deployment-files with folder name (e.g., nfvi-14) and --deploy yes flags to run tripleo-overcloud deployment.<br><br>So, during this deployment process, Infrared can copy the templates and over cloud script to Director /home/stack directory and proceed to execute the script.<br>Example:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>IR_DIR=/root/.venv_infrared/infrared/plugins/tripleo-overcloud/vars/deployment/files/nfvi-14/
# infrared tripleo-overcloud -vvvv --version 14 \
        --deployment-files nfvi-14 \
        --storage-backend lvm \
        --network-backend vxlan,vlan,flat \
        --network-protocol ipv4 \
        --introspect no \
        --tagging no \
        --deploy yes \
        --containers yes \
        --registry-skip-puddle yes  \
        --registry-mirror server-xxx-xx.xxx.xxx.redhat.com:5000 \
        --registry-namespace server-xxx-xx.xxx.xxx.redhat.com:5000 \
        --registry-prefix red_hat_xxx-osp14_containers- \
        --registry-tag latest \
        --registry-tag-label True \
        --collect-ansible-facts True \
        --overcloud-script ${IR_DIR}/overcloud_deploy.sh</code></pre>
<!-- /wp:code -->

<!-- wp:quote -->
<blockquote class="wp-block-quote"><p><strong>Note:</strong> Additionally the <strong>--overcloud-templates</strong> option can be used to pass additional templates. Ensure in the deployment script, the path should be correct to avoid the unwanted deployment failures.</p></blockquote>
<!-- /wp:quote -->

<!-- wp:heading {"level":5} -->
<h5>Example of overcloud deployment script:</h5>
<!-- /wp:heading -->

<!-- wp:code -->
<pre class="wp-block-code"><code># cat /root/.venv_infrared/infrared/plugins/tripleo-overcloud/vars/deployment/files/nfvi-14/overcloud_deploy.sh
#!/bin/bash

TEMPLATE_DIR=/home/stack/nfvi-14

set -ux

time openstack overcloud deploy \
--templates /usr/share/openstack-tripleo-heat-templates \
-n ${TEMPLATE_DIR}/network_data.yaml \
-r ${TEMPLATE_DIR}/roles_data.yaml \
--stack overcloud \
--libvirt-type kvm \
--ntp-server xx.xx.xx.xx \
-e ${TEMPLATE_DIR}/network/network-environment.yaml \
-e ${TEMPLATE_DIR}/hostnames.yml \
-e ${TEMPLATE_DIR}/debug.yaml \
-e ${TEMPLATE_DIR}/nodes_data.yaml \
-e /usr/share/openstack-tripleo-heat-templates/environments/neutron-ovs-dpdk.yaml \
-e /usr/share/openstack-tripleo-heat-templates/environments/neutron-sriov.yaml \
-e /usr/share/openstack-tripleo-heat-templates/environments/ovs-dpdk-permissions.yaml \
-e ${TEMPLATE_DIR}/rhel-registration/environment-rhel-registration.yaml \
-e ${TEMPLATE_DIR}/network/dpdk_sriov-environment.yaml \
-e /usr/share/openstack-tripleo-heat-templates/environments/services/skydive-environment.yaml \
-e /usr/share/openstack-tripleo-heat-templates/environments/disable-telemetry.yaml \
-e /usr/share/openstack-tripleo-heat-templates/environments/host-config-and-reboot.yaml \
-e /home/stack/containers-prepare-parameter.yaml \
--validation-warnings-fatal --timeout 300</code></pre>
<!-- /wp:code -->

<!-- wp:quote -->
<blockquote class="wp-block-quote"><p><strong>This hybrid deployment has been tested in a Lab environment and as all NFVi environment are not identical hence it will vary for individual usecase. Since infrared is not ready for production it requires best practice to adopt.</strong></p></blockquote>
<!-- /wp:quote -->

<!-- wp:paragraph -->
<p>Would appreciate your feedback to assess its success and further improvise the Infrared deployer.<br></p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":4} -->
<h4>References and Acknowledgements<br></h4>
<!-- /wp:heading -->

<!-- wp:list -->
<ul><li>https://infrared.readthedocs.io/en/stable/</li><li>https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/14/</li><li>www.redhat.com/telco</li><li>http://abregman.com/2017/03/20/infrared-deploying-and-testing-openstack-just-made-easier/</li></ul>
<!-- /wp:list -->
