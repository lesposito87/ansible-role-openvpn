<!-- DOCSIBLE START -->

# 📃 Role overview

## ansible-role-openvpn

**Description** 

An Ansible Role that installs and configures an OpenVPN server,
automatically generating all necessary client configuration files
for secure connections.

⚠️ This Ansible Role has been tested exclusively on **Amazon Linux 2**

**How to execute it?**

**1-** Create the following files, customizing the contents of the inventory and vars.yml files according to your requirements:
```
.
├── ansible.cfg
├── inventory
├── main.yml
├── requirements.yml
└── vars.yml
```

`ansible.cfg`:
```
➜ cat ansible.cfg
[defaults]
host_key_checking = False
inventory = inventory

[privilege_escalation]
become=True
become_method=sudo
become_user=root
become_ask_pass=False

[ssh_connection]
ssh_args = -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa
```

`inventory`:
```
➜ cat inventory
[openvpn_server]
123.123.123.123 ansible_ssh_port=33333 ansible_ssh_user=ec2-user ansible_ssh_private_key_file=/Users/myuser/.ssh/id_rsa_myuser
```

`vars.yml`:
```
➜ cat vars.yml
openvpn_client_bundle_copy_locally:
  local_copy: true
  client_dir: "/Users/myuser/openvpn/"
```

`ansible.cfg`:
```
➜ cat ansible.cfg
[defaults]
host_key_checking = False
inventory = inventory

[privilege_escalation]
become=True
become_method=sudo
become_user=root
become_ask_pass=False

[ssh_connection]
ssh_args = -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa
```

`requirements.yml`:
```
➜ cat requirements.yml
---
roles:
  - name: lesposito87.ansible-role-openvpn
    src: https://github.com/lesposito87/ansible-role-openvpn.git
    version: main
```

`main.yml`:
```
➜ cat main.yml
---
- hosts: openvpn_server
  become: true
  gather_facts: True
  vars_files:
    - vars.yml

  tasks:
    - include_role:
        name: lesposito87.ansible-role-openvpn
```

**2-** Install the Ansible Role locally:
```
➜ ansible-galaxy install -r requirements.yml --force
```

**3-** Execute the Ansible Playbook:
```
➜ ansible-playbook  main.yml
```
At the end of the execution you should find locally all the required OpenVPN Client files:
```
/Users/myuser/openvpn/
├── ca.crt
├── ca.key
├── client.conf
├── client.crt
├── client.key
├── dh.pem
└── pfs.key
```

**4-** Import the `client.conf` file into your preferred OpenVPN client software (e.g. [tunnelblink](https://tunnelblick.net/)) and establish a connection to your OpenVPN server."

<br>

### Defaults

**These are static variables with lower priority**

#### File: defaults/main.yml

| Var          | Type         | Value       |Required    | Title       |
|--------------|--------------|-------------|-------------|-------------|
| [openvpn_server_ip](defaults/main.yml#L4)   | str   | `{{ ansible_host }}` |    True  |  Public IP of your OpenVPN instance |
| [openvpn_pkgs](defaults/main.yml#L8)   | list   | `['openvpn', 'git']` |    True  |  Required packages |
| [openvpn_subnet_cidr](defaults/main.yml#L14)   | str   | `10.8.0.0` |    True  |  OpenVPN Subnet |
| [openvpn_subnet_netmask](defaults/main.yml#L18)   | str   | `255.255.255.0` |    True  |  OpenVPN Netmask |
| [openvpn_nat_rules_subnets](defaults/main.yml#L22)   | list   | `['10.8.0.0/24']` |    True  |  OpenVPN Subnet (used to configure iptables NAT rules) |
| [easy_rsa_git_repo_tag](defaults/main.yml#L27)   | str   | `v3.2.1` |    True  |  EasyRSA Git Repository tag |
| [openvpn_client_bundle_copy_locally](defaults/main.yml#L31)   | dict   | `{'local_copy': True, 'client_dir': '/tmp/openvpn/'}` |    True  |  Local directory where the resulting OpenVPN client configuration files will be copied. Set this to false if you do not wish to copy the client bundle to your local machine. |

<br>

### Tasks

#### File: tasks/openvpn.yml

| Name | Module | Has Conditions |
| ---- | ------ | --------- |
| Check if SELinux is installed | find | False |
| Configuring SELinux in permissive mode | ansible.posix.selinux | True |
| Ensure Firewalld/Iptables service are stopped | service | False |
| Install OpenVPN packages | package | False |
| Clone Easy-RSA repo | git | False |
| Load "iptable_nat" module | shell | False |
| Enable IPV4 IP forwarding | sysctl | False |
| Get the first non-loopback network interface | set_fact | False |
| Render OpenVPN NAT rules template | template | False |
| Copy OpenVPN NAT rules systemd unit | copy | False |
| Start & Enable OpenVPN NAT rules systemd service | systemd_service | False |
| Retrieve dir containing the binary based on easy-rsa major version | set_fact | False |
| Make sure easyrsa binary exist ("{{ easyrsa_binary_dir }}/easyrsa") | stat | False |
| Unnamed | fail | True |
| Check if the Easy-RSA PKI dir was already initialized ("{{ easyrsa_binary_dir }}/pki") | stat | False |
| Unnamed | debug | True |
| Unnamed_block | block | True |
| Initialize the PKI, build the CA, generate the Diffie-Hellman key, create/sign server & client certificates | shell | False |
| Generate a TLS key for OpenVPN | shell | False |
| Copy OpenVPN systemd unit file | copy | False |
| Copy OpenVPN "server.conf" template | template | False |
| Start & Enable openvpn systemd service | systemd_service | False |
| Create the client-bundle directory | file | False |
| Copy OpenVPN "client.conf" template | template | False |
| Copy client files to the "/etc/openvpn/client-bundle" directory | copy | False |
| Unnamed_block | block | True |
| Unnamed | ansible.builtin.find | False |
| Copy OpenVPN Client Bundle locally on "{{ openvpn_client_bundle_copy_locally.client_dir }}" | ansible.builtin.fetch | False |

#### File: tasks/main.yml

| Name | Module | Has Conditions |
| ---- | ------ | --------- |
| Wait for connection... |  | False |
| Assert that "openvpn_server_ip" is set, not empty, and a valid IP address | assert | False |
| Assert that "openvpn_client_bundle_copy_locally.client_dir" is set, is not empty and ends with "/" | assert | False |
| Including Openvpn setup tasks | include_tasks | False |

<br>

## Task Flow Graphs

### Graph for openvpn.yml

```mermaid
flowchart TD
Start
classDef block stroke:#3498db,stroke-width:2px;
classDef task stroke:#4b76bb,stroke-width:2px;
classDef includeTasks stroke:#16a085,stroke-width:2px;
classDef importTasks stroke:#34495e,stroke-width:2px;
classDef includeRole stroke:#2980b9,stroke-width:2px;
classDef importRole stroke:#699ba7,stroke-width:2px;
classDef includeVars stroke:#8e44ad,stroke-width:2px;
classDef rescue stroke:#665352,stroke-width:2px;

  Start-->|Task| Check_if_SELinux_is_installed0[check if selinux is installed]:::task
  Check_if_SELinux_is_installed0-->|Task| Configuring_SELinux_in_permissive_mode1[configuring selinux in permissive mode<br>When: **selinux result files   length   0**]:::task
  Configuring_SELinux_in_permissive_mode1-->|Task| Ensure_Firewalld_Iptables_service_are_stopped2[ensure firewalld iptables service are stopped]:::task
  Ensure_Firewalld_Iptables_service_are_stopped2-->|Task| Install_OpenVPN_packages3[install openvpn packages]:::task
  Install_OpenVPN_packages3-->|Task| Clone_Easy_RSA_repo4[clone easy rsa repo]:::task
  Clone_Easy_RSA_repo4-->|Task| Load__iptable_nat__module5[load  iptable nat  module]:::task
  Load__iptable_nat__module5-->|Task| Enable_IPV4_IP_forwarding6[enable ipv4 ip forwarding]:::task
  Enable_IPV4_IP_forwarding6-->|Task| Get_the_first_non_loopback_network_interface7[get the first non loopback network interface]:::task
  Get_the_first_non_loopback_network_interface7-->|Task| Render_OpenVPN_NAT_rules_template8[render openvpn nat rules template]:::task
  Render_OpenVPN_NAT_rules_template8-->|Task| Copy_OpenVPN_NAT_rules_systemd_unit9[copy openvpn nat rules systemd unit]:::task
  Copy_OpenVPN_NAT_rules_systemd_unit9-->|Task| Start___Enable_OpenVPN_NAT_rules_systemd_service10[start   enable openvpn nat rules systemd service]:::task
  Start___Enable_OpenVPN_NAT_rules_systemd_service10-->|Task| Retrieve_dir_containing_the_binary_based_on_easy_rsa_major_version11[retrieve dir containing the binary based on easy<br>rsa major version]:::task
  Retrieve_dir_containing_the_binary_based_on_easy_rsa_major_version11-->|Task| Make_sure_easyrsa_binary_exist___easyrsa_binary_dir_easyrsa__12[make sure easyrsa binary exist   easyrsa binary<br>dir easyrsa  ]:::task
  Make_sure_easyrsa_binary_exist___easyrsa_binary_dir_easyrsa__12-->|Task| Unnamed_task_1313[unnamed task 13<br>When: **not stat easyrsa binary stat exists**]:::task
  Unnamed_task_1313-->|Task| Check_if_the_Easy_RSA_PKI_dir_was_already_initialized___easyrsa_binary_dir_pki__14[check if the easy rsa pki dir was already<br>initialized   easyrsa binary dir pki  ]:::task
  Check_if_the_Easy_RSA_PKI_dir_was_already_initialized___easyrsa_binary_dir_pki__14-->|Task| Unnamed_task_1515[unnamed task 15<br>When: **stat easyrsa pki stat exists**]:::task
  Unnamed_task_1515-->|Block Start| Unnamed_task_1616_block_start_0[[unnamed task 16<br>When: **not stat easyrsa pki stat exists**]]:::block
  Unnamed_task_1616_block_start_0-->|Task| Initialize_the_PKI__build_the_CA__generate_the_Diffie_Hellman_key__create_sign_server___client_certificates0[initialize the pki  build the ca  generate the<br>diffie hellman key  create sign server   client<br>certificates]:::task
  Initialize_the_PKI__build_the_CA__generate_the_Diffie_Hellman_key__create_sign_server___client_certificates0-->|Task| Generate_a_TLS_key_for_OpenVPN1[generate a tls key for openvpn]:::task
  Generate_a_TLS_key_for_OpenVPN1-.->|End of Block| Unnamed_task_1616_block_start_0
  Generate_a_TLS_key_for_OpenVPN1-->|Task| Copy_OpenVPN_systemd_unit_file17[copy openvpn systemd unit file]:::task
  Copy_OpenVPN_systemd_unit_file17-->|Task| Copy_OpenVPN__server_conf__template18[copy openvpn  server conf  template]:::task
  Copy_OpenVPN__server_conf__template18-->|Task| Start___Enable_openvpn_systemd_service19[start   enable openvpn systemd service]:::task
  Start___Enable_openvpn_systemd_service19-->|Task| Create_the_client_bundle_directory20[create the client bundle directory]:::task
  Create_the_client_bundle_directory20-->|Task| Copy_OpenVPN__client_conf__template21[copy openvpn  client conf  template]:::task
  Copy_OpenVPN__client_conf__template21-->|Task| Copy_client_files_to_the___etc_openvpn_client_bundle__directory22[copy client files to the   etc openvpn client<br>bundle  directory]:::task
  Copy_client_files_to_the___etc_openvpn_client_bundle__directory22-->|Block Start| Unnamed_task_2323_block_start_0[[unnamed task 23<br>When: **openvpn client bundle copy locally local copy and<br>openvpn client bundle copy locally client dir  <br>length   0 and openvpn client bundle copy locally<br>client dir endswith**]]:::block
  Unnamed_task_2323_block_start_0-->|Task| Unnamed_task_00[unnamed task 0]:::task
  Unnamed_task_00-->|Task| Copy_OpenVPN_Client_Bundle_locally_on_____openvpn_client_bundle_copy_locally_client_dir____1[copy openvpn client bundle locally on     openvpn<br>client bundle copy locally client dir    ]:::task
  Copy_OpenVPN_Client_Bundle_locally_on_____openvpn_client_bundle_copy_locally_client_dir____1-.->|End of Block| Unnamed_task_2323_block_start_0
  Copy_OpenVPN_Client_Bundle_locally_on_____openvpn_client_bundle_copy_locally_client_dir____1-->End
```


### Graph for main.yml

```mermaid
flowchart TD
Start
classDef block stroke:#3498db,stroke-width:2px;
classDef task stroke:#4b76bb,stroke-width:2px;
classDef includeTasks stroke:#16a085,stroke-width:2px;
classDef importTasks stroke:#34495e,stroke-width:2px;
classDef includeRole stroke:#2980b9,stroke-width:2px;
classDef importRole stroke:#699ba7,stroke-width:2px;
classDef includeVars stroke:#8e44ad,stroke-width:2px;
classDef rescue stroke:#665352,stroke-width:2px;

  Start-->|Task| Wait_for_connection___0[wait for connection   ]:::task
  Wait_for_connection___0-->|Task| Assert_that__openvpn_server_ip__is_set__not_empty__and_a_valid_IP_address1[assert that  openvpn server ip  is set  not empty <br>and a valid ip address]:::task
  Assert_that__openvpn_server_ip__is_set__not_empty__and_a_valid_IP_address1-->|Task| Assert_that__openvpn_client_bundle_copy_locally_client_dir__is_set__is_not_empty_and_ends_with____2[assert that  openvpn client bundle copy locally<br>client dir  is set  is not empty and ends with    ]:::task
  Assert_that__openvpn_client_bundle_copy_locally_client_dir__is_set__is_not_empty_and_ends_with____2-->|Include task| openvpn_yml3[including openvpn setup tasks<br>include_task: openvpn yml]:::includeTasks
  openvpn_yml3-->End
```

## Author Information
https://www.linkedin.com/in/lucaesposito87/

#### License
MIT

#### Minimum Ansible Version
2.7

#### Platforms

- **Amazon Linux**: ['2']

<!-- DOCSIBLE END -->
