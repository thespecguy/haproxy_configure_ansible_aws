__<h1>How to configure HAProxy and automatic updation of it’s configuration file each time managed node is added using Ansible over AWS EC2 Instance</h1>__

![Ansible_HAProxy_HTTPD_AWS](https://miro.medium.com/max/875/1*Ne3xzqtSQ10NaNSZrSg6AQ.jpeg)

<br>
<h2> Content </h2>
<h4><ul>
<li><i>About Ansible</i></li>
<li><i>About HAProxy</i></li>
<li><i>About Apache Httpd Webserver</i></li> 
<li><i>Project Understanding : Dynamic Inventory Setup</i></li> 
<li><i>Project Understanding : Ansible Playbook Setup</i></li>
</ul></h4><br>

<h2>About Ansible</h2>
<h3>What is Ansible ?</h3>
<i><b>Ansible</b> is a radically simple IT automation engine that automates cloud provisioning, configuration management, application deployment, intra-service orchestration, and many other IT needs.</i>

<h3>Why use Ansible ?</h3>
<h4><ol>
<li>
  Simple
  <ul>
    <li><i>Human readable automation</i></li>
    <li><i>No special coding skills needed</i></li>
    <li><i>Tasks executed in order</i></li>
    <li><i>Get productive quickly</i></li>
  </ul>
</li><br>
<li>
  Powerful
  <ul>
    <li><i>App deployment</i></li>
    <li><i>Configuration management</i></li>
    <li><i>Workflow orchestration</i></li>
    <li><i>Orchestrate the app lifecycle</i></li>
  </ul>
</li><br>
<li>
  Agentless
  <ul>
    <li><i>Agentless architecture</i></li>
    <li><i>Uses OpenSSH and WinRM</i></li>
    <li><i>No agents to exploit or update</i></li>
    <li><i>Predictable, reliable and secure</i></li>
  </ul>
</li><br>
</ol></h4>
For Ansible Documentation, visit the link mentioned below:<br>
https://docs.ansible.com/

<br>
<p align="center"><b>. . .</b></p><br>

<h2>About HAProxy</h2>
<ul>
  <li><i><b>HAProxy</b> is free, open source software that provides a high availability <b>load balancer</b> and <b>proxy server</b> for TCP and HTTP-based applications that spreads requests across multiple servers.</i></li>
  <li><i>It’s written in <b>C</b> and has a reputation for being fast and efficient in terms of processor and memory usage.</i></li>
</ul>

For HAProxy Documentation, visit the link mentioned below:<br>
https://www.haproxy.com/documentation/hapee/latest/onepage/intro/

<br>
<p align="center"><b>. . .</b></p><br>

<h2>About Apache HTTPD Webserver</h2>
<ul>
  <li><i>The <b>Apache HTTP Server</b>, colloquially called <b>Apache</b>, is a free and open-source cross-platform web server software, released under the terms of <b>Apache License 2.0</b>.</i></li>
  <li><i>Apache is developed and maintained by an open community of developers under the auspices of the <b>Apache Software Foundation</b>.</i></li>
  <li><i>The vast majority of Apache HTTP Server instances run on a Linux distribution, but current versions also run on Microsoft Windows, OpenVMS and a wide variety of Unix-like systems.</i></li>
</ul>

For Apache HTTPD Documentation, visit the link mentioned below:<br>
https://httpd.apache.org/docs/

<br>
<p align="center"><b>. . .</b></p><br>

<h2>Project Understanding : Dynamic Inventory Setup</h2>
<h3>What is Dynamic Inventory for AWS in Ansible ?</h3>
<i><b>Dynamic inventory</b> is an <b>Ansible</b> plugin that makes an API call to <b>AWS</b> to get the instance information in the run time. It gives the <b>ec2</b> instance details <b>dynamically</b> to manage the <b>AWS</b> infrastructure.</i><br><br>
<p>Lets understand the procedure to setup Dynamic Inventory for AWS EC2 Instance one by one :</p>
<ul>
  <li>Create a directory for installing <b>dynamic inventory</b> modules for Amazon EC2.</li><br>
  
  ```shell
   mkdir dynamic_inventory
  ```
  
  <li>Install dynamic inventory modules i.e., ec2.py and ec2.ini, also both files are interdependent on each other. <b>ec2.ini</b> is responsible for storing information related to AWS account whereas <b>ec2.py</b> is responsible for executing modules that collects the information of instances launched on AWS.</li><br>
  <p>Command for creation of ec2.py dynamic inventory file 👇</p>
  
  ```shell
  wget   https://raw.githubusercontent.com/ansible/ansible/stable-2.9/contrib/inventory/ec2.py
  ```
  
  <br>
  <p>Command for creation of ec2.ini dynamic inventory file 👇</p>
  
  ```shell
  wget   https://raw.githubusercontent.com/ansible/ansible/stable-2.9/contrib/inventory/ec2.ini
  ```
  
  <br>
  <p><i><b>Note:</b></i> Install <b>wget</b> command in case it’s not present. The command for the same(for RHEL) is mentioned below:</p>
  
  ```shell
  sudo yum install wget
  ```
  
  <br>
  
  ![dynamic_inventory](https://miro.medium.com/max/875/1*JB5MKt5QyAeFL6Fqml-prQ.png)
  
  <li>Next, make the dynamic inventory files executable using chmod command.</li><br>
  
  ```shell
  chmod +x ec2.py
  
  chmod +x ec2.ini
  ```
  
  <br>
  
  ![executable_dynamic_inventory](https://miro.medium.com/max/718/1*f0JmUvhWhSisr_PZ_wl1_g.png)
  
  <li>Key needs to be provided in order to login to the instances newly launched. Also key with <b>.pem</b> format works in this case and not the one with <b>.ppk</b> format. Permission needs to be provided to respective key to set it up in <b>read mode</b>.</li><br>
  
  ```shell
  chmod 400 keypair.pem
  ```
  
  <br>
  
  ![keypair_read_mode](https://miro.medium.com/max/555/1*9lbkjsCzd7fJXLcJ_dMReA.png)
  
  <br>
  <p><i><b>Note:</b></i></p>
  <p><b>Permissions in Linux</b></p>
  
  ```
  0 -> No Permission      -> ---
  1 -> Execute            -> --x
  2 -> Write              -> -w-
  3 -> Execute+Write      -> -wx
  4 -> Read               -> r--
  5 -> Read+Execute       -> r-x
  6 -> Read+Write         -> rw-
  7 -> Read+Write+Execute -> rwx
  ```
  
  <br>
  <li>Ansible Configuration Setup</li><br>
  <ol>
    <li><b>Mention the path to the directory created for installing dynamic inventory module under <i>inventory</i> keyword in the configuration file.</b></li>
    <li><b>Since Ansible works on SSH protocol for Linux OS and it prompts yes/no by default when used, in order to disable it , <i>host_key_checking</i> needs to be set to false.</b></li>
    <li><b>Warnings given by commmands could be disabled by setting <i>command_warnings</i> to false.</b></li>
    <li><b>The path to the private key could be provided under <i>private_key_file</i> in the configuration file.</b></li>
  </ol>
  <br>
  
  ![inventory](https://miro.medium.com/max/875/1*3cG7K5-Y-WqJNGZEd1fVUQ.png)
  
  <br>
  
  ```
  [defaults]
  
  inventory = path_to_directory
  host_key_checking = false
  command_warnings = false
  private_key_file = path_to_key
  ask_pass = false
  
  [privilege_escalation]
  become=true
  become_method=sudo
  become_user=root
  become_ask_pass=false
  ```
  
  <br>
  <li>Let’s execute the command to check if there is any issue in the dynamic inventory files in the directory containing the same.</li><br>
  
  ![ansible_all](https://miro.medium.com/max/875/1*bZabDAGtcvC-dfZkDDK_yA.png)
  
  <br>
  <li>As per the output correct above, some changes needs to be made in the respective file. In case of ec2.py, version specified needs to be changed i.e., from <b>python</b> to <b>python3</b>.</li><br>
  
  ![ec2_py](https://miro.medium.com/max/875/1*n29_C3okBMw3IsLSQZNaFw.png)
  
  <p align="center"><b>ec2.py</b></p><br>
  <br>
  
  ```shell
  #!/usr/bin/python3
  ```
  
  <br>
  <li>Also, we need to update regions, AWS account <b>access_key</b> and <b>secret_access_key</b> inside ec2.ini file.</li><br>
  
  ![ec2_ini](https://miro.medium.com/max/875/1*cPJ7k5D_SLGA-1j2APf4Sw.png)
  
  <p align="center"><b>ec2.ini</b></p><br><br>
  
  ![ec2_ini_region](https://miro.medium.com/max/875/1*3fMWhOLYCuP15MiR5CoTRQ.png)
  
  <p align="center"><b>Region specified is ‘ap-south-1’ : ec2.ini</b></p><br>
  
  <li>After the changes made in the previous step, export <b>aws_region</b>, <b>aws_access_key</b> and <b>aws_access_secret_key</b> on the command line.</li><br>
  
  ```
  export AWS_REGION='ap-south-1'(in this case)
  export AWS_ACCESS_KEY=XXXX
  export AWS_ACCESS_SECRET_KEY=XXXX
  ```
  
  <br>
  <li>After the following setup, the command used before provides the list of hosts in AWS properly. It indicates that the particular issue has been resolved successfully.</li><br>
  
  ![ansible_all_working](https://miro.medium.com/max/875/1*4EUMjeila4Bw_C6TDRQ3FQ.png)
  
</ul>
<br>
<p align="center"><b>. . .</b></p><br>

<h2>Project Understanding : Ansible Playbook Setup</h2>
<p>Let’s understand its implementation part by part</p>

<h3>Part 1 : Ansible Playbook (main.yml)</h3>
<h3><i>AWS Infrastructure Setup</i></h3>
<ul>
  <li><i><b>Prerequisite:</b></i> Installation of boto3 (Python SDK for AWS)</li><br>
  
  ```yaml
  - name: Installation of boto3 library
    pip:
      name: boto3
      state: present
  ```
  
  <br>
  <li><b><i>“Creation of VPC for EC2 Instance”:</i></b> Creates <b>AWS VPC(Virtual Private Cloud)</b> using the <b>ec2_vpc_net</b> module and stores the value in the variable <b>ec2_vpc</b>. The required parameters to be specified are <b>cidr_block</b> and <b>name</b>.</li><br>
  
  ```yaml
  - name: Creation of VPC for EC2 Instance
    ec2_vpc_net:
      aws_access_key: "{{ aws_access_key }}"
      aws_secret_key: "{{ aws_secret_key }}"
      region: "{{ region }}"
      name: "{{ vpc_name}}"
      cidr_block: "{{ cidr_block_vpc }}"
      state: present
    register: ec2_vpc
  ```
  
  <br>
  
  ![VPC_1](https://miro.medium.com/max/875/1*crLcdtoDm7Rbj_oWIIBADg.png)
  
  ![VPC_2](https://miro.medium.com/max/875/1*Yy0c_e6iUBatPwf227R7SQ.png)
  
  <br>
  <li><b><i>“Internet Gateway for VPC”:</i></b> Sets up internet gateway for VPC created in the previous task. One of the parameters required for this setup is <b>vpc_id</b> (return value of <b>ec2_vpc_net</b> module). It uses <b>ec2_vpc_igw</b> module and stores the value in variable <b>igw_info</b>.</li><br>
  
  ```yaml
  - name: Internet Gateway for VPC
    ec2_vpc_igw:
      aws_access_key: "{{ aws_access_key }}"
      aws_secret_key: "{{ aws_secret_key }}"
      region: "{{ region }}"
      state: present
      vpc_id: "{{ ec2_vpc.vpc.id  }}"
      tags:
        Name: "{{ igw_name }}"
    register: igw_info
  ```
  
  <br>
  
  ![igw](https://miro.medium.com/max/875/1*Thi4x_OFEzEFCAIMLSxPYQ.png)
  
  <br>
  <li><b><i>“VPC Subnet Creation”:</i></b> Creates Subnet for VPC created before in the specified availability zone in the region. One of the parameters required for this setup is <b>vpc_id</b>(return value of <b>ec2_vpc_net</b> module). Also, <b>map_public</b> should be set to <b>yes</b> so that the instances launched within it would be assigned public IP by default. It uses <b>ec2_vpc_subnet</b> module and stores the value in variable <b>subnet_info</b>.</li><br>
  
  ```yaml
  - name: VPC Subnet Creation
    ec2_vpc_subnet:
      aws_access_key: "{{ aws_access_key }}"
      aws_secret_key: "{{ aws_secret_key }}"
      region: "{{ region }}"
      vpc_id: "{{ ec2_vpc.vpc.id  }}"
      az: "{{ availability_zone }}"
      state: present
      cidr: "{{ cidr_subnet  }}"
      map_public: yes
      tags:
        Name: "{{ subnet_name }}"
    register: subnet_info

  ```
  
  <br>
  
  ![subnet](https://miro.medium.com/max/875/1*GuyjIhOn8stk4VK2XvNBBQ.png)
  
  <br>
  <li><b><i>“Creation of VPC Subnet Route Table”:</i></b> Creates Route Table and associates it with the subnet. Also, it specifies the route to the internet gateway created before in the route table. Parameters like <b>vpc_id(required)</b>, <b>subnets</b> & <b>gateway_id</b> could be obtained as return value from <b>ec2_vpc_net</b>, <b>ec2_vpc_subnet</b> and <b>ec2_vpc_igw</b> respectively.</li><br>
  
  ```yaml
  - name: Creation of VPC Subnet Route Table
    ec2_vpc_route_table:
      aws_access_key: "{{ aws_access_key }}"
      aws_secret_key: "{{ aws_secret_key }}"
      region: "{{ region }}"
      vpc_id: "{{ ec2_vpc.vpc.id  }}"
      state: present
      tags:
        Name: "{{ route_table_name }}"
      subnets: [ "{{ subnet_info.subnet.id }}" ]
      routes:
        - dest: 0.0.0.0/0
          gateway_id: "{{ igw_info.gateway_id }}"
  ```
  
  <br>
  
  ![subnet_route_table](https://miro.medium.com/max/875/1*vTMuXn1VSHrb_xA-6HYhKw.png)
  
  <br>
  <li><b><i>“Security Group Creation”:</i></b> Creates Security Group for both loadbalancer & webserver. One of the attributes i.e., <b>vpc_id</b> could be obtained as a return value from <b>ec2_vpc_net</b> module. Incoming traffic could be limited by specifying the rules that includes protocol, ports as well CIDR IP. It uses <b>ec2_group</b> module and stores the value in variable <b>security_group_info</b>.</li><br>
  
  ```yaml
  - name: Security Group Creation
    ec2_group:
      aws_access_key: "{{ aws_access_key }}"
      aws_secret_key: "{{ aws_secret_key }}"
      region: "{{ region }}"
      name: "{{ sg_name  }}"
      vpc_id: "{{ ec2_vpc.vpc.id  }}"
      state: present
      description: Security Group for both Loadbalancer & Webserver
      tags:
        Name: "{{ sg_name  }}"
      rules:
      - proto: tcp
        ports:
          - 80
          - 8080
          - 22
        cidr_ip: 0.0.0.0/0
    register: security_group_info
  ```
  
  <br>
  
  ![security_group](https://miro.medium.com/max/875/1*t0vcqcRR_d7YOXCazcsKTw.png)
  
  ![security_group_inbound](https://miro.medium.com/max/875/1*bn_W-rK9TAqQS833PjAUeg.png)
  
  ![security_group_outbound](https://miro.medium.com/max/875/1*uyIwSsf3MS9YXTmhP2bt-A.png)
  
  <br>
  <li><b><i>“Creation of EC2 Instance for Load Balancer”:</i></b> Creates EC2 Instance for Load Balancer. Parameters like <b>vpc_subnet_id</b> & <b>group_id</b> could be obtained as a return value from <b>ec2_vpc_subnet</b> and <b>ec2_group</b> module. <b>exact_count</b> parameter is set to 1 in this case. <b>wait</b> parameter is used to wait for the instances created to reach its desired state, also timeout w.r.t. waiting has also been set using <b>wait_timeout</b> parameter.</li><br>
  
  ```yaml
  - name: Creation of EC2 Instance for Load Balancer
    ec2:
      aws_access_key: "{{ aws_access_key }}"
      aws_secret_key: "{{ aws_secret_key }}"
      region: "{{ region }}"
      image: "{{ image_id }}"
      exact_count: 1
      instance_type: "{{ instance_type }}"
      vpc_subnet_id: "{{ subnet_info.subnet.id }}"
      key_name: "{{ key_name }}"
      group_id: "{{ security_group_info.group_id  }}"
      wait: yes
      wait_timeout: 600
      instance_tags:
        Name: loadbalancer
      count_tag:
        Name: loadbalancer
  ```
  
  <br>
  
  ![ec2_loadbalancer](https://miro.medium.com/max/875/1*kxr2h7r0EevPMnVP2wKCLA.png)
  
  <br>
  <li><b><i>“Creation of EC2 Instance for Web Server”:</i></b> Same as previous task. Except in this case , the value of <b>exact_count</b> parameter is dynamic and could be specified as per requirement.</li><br>
  
  ```yaml
  - name: Creation of EC2 Instance for Web Server
    ec2:
      aws_access_key: "{{ aws_access_key }}"
      aws_secret_key: "{{ aws_secret_key }}"
      region: "{{ region }}"
      image: "{{ image_id }}"
      instance_type: "{{ instance_type }}"
      vpc_subnet_id: "{{ subnet_info.subnet.id }}"
      key_name: "{{ key_name }}"
      group_id: "{{ security_group_info.group_id  }}"
      exact_count: "{{ instance_count }}"
      wait: yes
      wait_timeout: 600
      instance_tags:
        Name: webserver
      count_tag:
        Name: webserver
    ignore_errors: yes
  ```
  
  ![ec2_webserver](https://miro.medium.com/max/875/1*wsulUSmfoVkemQdG_yaK1g.png)
  
  <br>
  <li><b><i>“meta: refresh_inventory”:</i></b> Meta tasks are a special kind of task that influences Ansible internal execution or state. <b>refresh_inventory</b> is the meta task that reloads the dynamic inventory generated. In this case, it reloads the dynamic inventory that makes it easier to scale up or scale down the number of EC2 Instances without much hindrance.</li><br>
  
    ```yaml
    - meta: refresh_inventory
    ```
  
  <br>
  <li><b><i>“pause”:</i></b> This module is used for pausing Ansible playbook execution. In this case, it pauses the execution to provide meta task to refresh the dynamic inventory.</li><br>
  
  ```yaml
  - pause:
      minutes: 2
  ```
  
  <p><b><i>Note:</i></b></p>
  <ol>
  <li>The main reason behind specifying the <b>instance_tags</b> for both loadbalancer and webserver, as dynamic inventory would create host groups accordingly which provides a proper distinction between loadbalancer and webserver instances. The list of dynamic host groups could be obtained using the command mentioned below:</li><br>
    
  ```shell
  ./ec2.py --list
  ```
    
  <br>
    
  ![dynamic_inventory_host](https://miro.medium.com/max/875/1*6HjjG-WJ3fnFaAcMZTFfNw.png)
    
  <br>
  <li>The reason behind usage of <b>ignore_errors</b> keyword in <b>“Creation of EC2 Instance for Web Server”</b> task is because there are chances where it might fail due to <b>timeout</b>, though the required number of instances are set up. This would break the flow of execution of the playbook, thereby to prevent this, the keyword has been set to <b>yes</b>.</li><br>
  <li>Parameters i.e., aws_access_key, aws_secret_key and region specified in each task above could be skipped, if its value has been already exported as mentioned below:</li><br>
    
  ```shell
  export AWS_REGION='ap-south-1'(in this case)
  export AWS_ACCESS_KEY=XXXX
  export AWS_ACCESS_SECRET_KEY=XXXX
  ```
    
  <br>
  <li><b>count_tag</b> parameter needs to mandatorily specified along with <b>exact_count</b> parameter to determine the number of instances running based on a tag criteria.</li>
  </ol>
</ul>
<br><br>
<h3><i>HAProxy Setup (loadbalancer)</i></h3>

<ul>
  <li><i><b>"Installation of HAProxy Software" :</b></i> Installs HAProxy Software</li><br>
  
  ```yaml
  - name: Installation of HAProxy Software
    package:
      name: "haproxy"
      state: present
  ```
  
  <br> 
  <li><i><b>“HAProxy Configuration File Setup” :</b></i> Copies the configuration file from templates directory in the Controller Node to the default path of HAProxy configuration file in the Managed Node. Changes in configuration file notifies the handler that results in restart of HAProxy service.</li><br>
  
  ```yaml
  - name: HAProxy Configuration File Setup
    template:
      dest: "/etc/haproxy/haproxy.cfg"
      src: "templates/haproxy.cfg"
    notify: Restart Load Balancer
  ```
  
  <br>
  <li><i><b>“Starting Load Balancer Service” :</b></i> Starts HAProxy Service</li><br>
  
  ```yaml
  - name: Starting Load Balancer Service
    service:
      name: "haproxy"
      state: started
      enabled: yes
  ```
  
  <br>
  <li><b>handlers -<i>“Restart Load Balancer</i>” :</b> Restarts HAProxy service in case of change in configuration file(when new managed node configured with Httpd Webserver is added).</li><br>
  
  ```yaml
  - name: Restart Load Balancer
    systemd:
      name: "haproxy"
      state: restarted
      enabled: yes
  ```
  
</ul>  
<br><br>
<h3><i>Apache Httpd Setup (webserver)</i></h3>  

<ul>
  <li><b><i>“HTTPD Installation” :</i></b> Installs Apache Httpd Software</li><br>
  
  ```yaml
  - name: HTTPD Installation
    package: 
      name: "httpd"
      state: present
  ```
  
  <br>
  <li><b><i>“Web Page Setup” :</i></b> Copies the web page from web directory in Controller Node to the Document Root in the Managed Node.</li><br>
  
  ```yaml
  - name: Web Page Setup
    template:
      dest: "/var/www/html"
      src: "web/index.html"
  ```
  
  <br>
  <li><b><i>“Starting HTTPD Service” :</i></b> Starts Httpd Service</li>
  
  ```yaml
  - name: Starting HTTPD Service
    service:
      name: "httpd"
      state: started
      enabled: yes
  ```
  
</ul>
<br><br>
<h3><i>Complete Playbook</i></h3>

```yaml
- hosts: localhost
  gather_facts: no
  vars_files:
    - vars.yml
  tasks:
  - name:
    pip: Installation of boto3 library
      name: boto3
      state: present
      
  - name: Creation of VPC for EC2 Instance
    ec2_vpc_net:
      aws_access_key: "{{ aws_access_key }}"
      aws_secret_key: "{{ aws_secret_key }}"
      region: "{{ region }}"
      name: "{{ vpc_name}}"
      cidr_block: "{{ cidr_block_vpc }}"
      state: present
    register: ec2_vpc
    
  - name: Internet Gateway for VPC
    ec2_vpc_igw:
      aws_access_key: "{{ aws_access_key }}"
      aws_secret_key: "{{ aws_secret_key }}"
      region: "{{ region }}"
      state: present
      vpc_id: "{{ ec2_vpc.vpc.id  }}"
      tags:
        Name: "{{ igw_name }}"
    register: igw_info
  
  - name: VPC Subnet Creation
    ec2_vpc_subnet:
      aws_access_key: "{{ aws_access_key }}"
      aws_secret_key: "{{ aws_secret_key }}"
      region: "{{ region }}"
      vpc_id: "{{ ec2_vpc.vpc.id  }}"
      az: "{{ availability_zone }}"
      state: present
      cidr: "{{ cidr_subnet  }}"
      map_public: yes
      tags:
        Name: "{{ subnet_name }}"
    register: subnet_info
  
  - name: Creation of VPC Subnet Route Table
    ec2_vpc_route_table:
      aws_access_key: "{{ aws_access_key }}"
      aws_secret_key: "{{ aws_secret_key }}"
      region: "{{ region }}"
      vpc_id: "{{ ec2_vpc.vpc.id  }}"
      state: present
      tags:
        Name: "{{ route_table_name }}"
      subnets: [ "{{ subnet_info.subnet.id }}" ]
      routes:
        - dest: 0.0.0.0/0
          gateway_id: "{{ igw_info.gateway_id }}"
  
  - name: Security Group Creation
    ec2_group:
      aws_access_key: "{{ aws_access_key }}"
      aws_secret_key: "{{ aws_secret_key }}"
      region: "{{ region }}"
      name: "{{ sg_name  }}"
      vpc_id: "{{ ec2_vpc.vpc.id  }}"
      state: present
      description: Security Group for both Loadbalancer & Webserver
      tags:
        Name: "{{ sg_name  }}"
      rules:
      - proto: tcp
        ports:
          - 80
          - 8080
          - 22
        cidr_ip: 0.0.0.0/0
    register: security_group_info
  
  - name: Creation of EC2 Instance for Load Balancer
    ec2:
      aws_access_key: "{{ aws_access_key }}"
      aws_secret_key: "{{ aws_secret_key }}"
      region: "{{ region }}"
      image: "{{ image_id }}"
      exact_count: 1
      instance_type: "{{ instance_type }}"
      vpc_subnet_id: "{{ subnet_info.subnet.id }}"
      key_name: "{{ key_name }}"
      group_id: "{{ security_group_info.group_id  }}"
      wait: yes
      wait_timeout: 600
      instance_tags:
        Name: loadbalancer
      count_tag:
        Name: loadbalancer
  
  - name: Creation of EC2 Instance for Web Server
    ec2:
      aws_access_key: "{{ aws_access_key }}"
      aws_secret_key: "{{ aws_secret_key }}"
      region: "{{ region }}"
      image: "{{ image_id }}"
      instance_type: "{{ instance_type }}"
      vpc_subnet_id: "{{ subnet_info.subnet.id }}"
      key_name: "{{ key_name }}"
      group_id: "{{ security_group_info.group_id  }}"
      exact_count: "{{ instance_count }}"
      wait: yes
      wait_timeout: 600
      instance_tags:
        Name: webserver
      count_tag:
        Name: webserver
    ignore_errors: yes
  
  - meta: refresh_inventory
  
  - pause:
      minutes: 2
      
      
- hosts: tag_Name_loadbalancer
  vars_files:
    - vars.yml  
  remote_user: "{{ remote_user }}"
  tasks:
  - name: Installation of HAProxy Software
    package:
      name: "haproxy"
      state: present
  
  - name: HAProxy Configuration File Setup
    template:
      dest: "/etc/haproxy/haproxy.cfg"
      src: "templates/haproxy.cfg"
    notify: Restart Load Balancer
  
  - name: Starting Load Balancer Service
    service:
      name: "haproxy"
      state: started
      enabled: yes
  
  handlers:
    - name: Restart Load Balancer
      systemd:
        name: "haproxy"
        state: restarted
        enabled: yes


- hosts: tag_Name_webserver
  vars_files:
    - vars.yml
  remote_user: "{{ remote_user }}"
  tasks:
  - name: HTTPD Installation
    package: 
      name: "httpd"
      state: present
  
  - name: Web Page Setup
    template:
      dest: "/var/www/html"
      src: "web/index.html"
  
  - name: Starting HTTPD Service
    service:
      name: "httpd"
      state: started
      enabled: yes
```

<br>
<p><b><i>Note:</i></b></p>
<p>In case of hosts corrresponding to <b>loadbalancer</b> and <b>webserver</b> i.e., <b>“tag_Name_loadbalancer”</b> and <b>“tag_Name_webserver”</b> respectively, remote-user needs to be setup in this case as the setup is present in the remote system like EC2 Instance.</p>
<br><br>
<h3>Part 2 : Variable Files (vars.yml)</h3>

```yaml
aws_access_key: "XXXX"
aws_secret_key: "XXXX"
region: "ap-south-1"
vpc_name: "ansible_vpc"
cidr_block_vpc: "10.0.0.0/16"
igw_name: "ansible_igw"
cidr_subnet: "10.0.1.0/24"
subnet_name: "ansible_subnet"
route_table_name: "ansible_route_table"
sg_name: "ansible_sg"
instance_type: "t2.micro"
key_name: "bastion"
instance_count: 10
image_id: "ami-068d43a544160b7ef"
availability_zone: "ap-south-1a"
remote_user: "ec2-user"
```

<br><br>
<h3>Part 3 : Web Page (index.html)</h3>

```html
<body bgcolor='aqua'>
<br><br>

<b>IP Addresses</b> : {{ ansible_all_ipv4_addresses }}
```

<br><br>
<h3>Part 4 : HAProxy Configuration File Template (haproxy.cfg)</h3>

```
#---------------------------------------------------------------------
# Example configuration for a possible web application.  See the
# full configuration options online.
#
#   https://www.haproxy.org/download/1.8/doc/configuration.txt
#
#---------------------------------------------------------------------#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon# turn on stats unix socket
    stats socket /var/lib/haproxy/stats# utilize system-wide crypto-policies
    ssl-default-bind-ciphers PROFILE=SYSTEM
    ssl-default-server-ciphers PROFILE=SYSTEM#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------
frontend main
    bind *:8080
    acl url_static       path_beg       -i /static /images /javascript /stylesheets
    acl url_static       path_end       -i .jpg .gif .png .css .jsuse_backend static          if url_static
    default_backend             app#---------------------------------------------------------------------
# static backend for serving up images, stylesheets and such
#---------------------------------------------------------------------
backend static
    balance     roundrobin
    server      static 127.0.0.1:4331 check#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend app
    balance          roundrobin
    
{%  for  hosts  in groups['tag_Name_webserver']  %}
    server  app1 {{ hosts }}:80 check
{% endfor %}
```

<br>
<p><b>Jinja template</b> and <b>for</b> loop specified above makes HAProxy configuration file more dynamic as it would add managed nodes specified under <b>‘tag_Name_webserver’</b> host dynamically.</p>

<br><br>
<h3>Part 5 : Output</h3>

![output](https://miro.medium.com/max/875/1*PjlI3ab12y_FLORm4BOoGA.gif)

<p align="center"><b>Output GIF</b></p>
<br>
<p align="center"><b>. . .</b></p><br>

<h2>Thank You :smiley:<h2>
<h3>LinkedIn Profile</h3>
https://www.linkedin.com/in/satyam-singh-95a266182
