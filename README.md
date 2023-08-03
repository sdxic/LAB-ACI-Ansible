# ACI Ansible Automation Lab
In this lab you will use Ansible playbooks to deploy, modify and remove various objects within a Cisco ACI fabric.  This lab will familiarize you with basic Ansible concepts and structure in the playbooks. Success at various steps will be verified by the user through examination of the ACI GUI and live traffic tests using ICMP, SSH, etc.

You will be required to modify some files during the course of this workshop.  You will not be required to write your own playbooks as this would require much more time.  The playbooks used are open source and thus free to use and modify.  That being said, writing playbooks and running them in a test environment is one of the best ways to learn.

***Disclaimer***:  *The playbooks used in these labs have been created specifically for the lab scenarios used within. Do not run them in any production environment without careful attention to utility and intended function and a complete understanding of their likely outcome.*

<br><br>

# Outline
* Part 1: Technology Intro
    * Cisco Application Centric Infrastructure (ACI)
    * Red Hat Ansible
    * Red Hat Ansible Automation Platform (RHAAP)
* Part 2: Lab Access 
    * Okta Account Setup
    * Virtual Desktop
        * ACI GUI
        * Ansible CLI
        * AP GUI
* Part 3: Lab 1 - Ansible CLI
* Part 4: Lab 2 - AP GUI

<br><br>

# Part 1: Technology Intro
## Cisco ACI
ACI is Cisco’s primary software defined network (SDN) solution for data center. In common with the goals of most SDN solutions, it reduces the management plane to a single pane of glass, for an entire data center fabric. The single pane of glass is a controller (the APIC) which is a redundant cluster that distributes the user defined policy to the fabric itself which services the data plane. The controller does not itself participate in the fabric state except to effect change instructions; it merely distributes configurations and accepts telemetry from the fabric components to provide centralized monitoring of network health.

## Red Hat Ansible
Orchestration and automation are not the same thing. But, Ansible can do both. Automation usually is implemented to fulfil a goal to replicate identical or near-identical repetitive actions. For example, the automatic transmission. Using a similar analogy, orchestration is akin to what the “self-driving car” does. Its controller must gather multiple inputs and react correctly, in near real time and provide adaptive change to critically co-dependent entities (e.g. steering, brakes, throttle).

Ansible is not at the heart of the self-driving car (we hope), but it is a good fit for creating and deleting complete “systems” (or parts of systems) like network, compute, and storage which all have interdependencies. Orchestration of these entities makes sense because the discrete components (combined) create a system. Deployed in isolation they each provide (almost) no function and require subsequent “manual” orchestration to become an actual “system”. Taking “manual” siloes out therefore creates opportunities for the value-add of accuracy, efficiency and de-risking repeat orchestrations.

## Ansible Automation Platform
Ansible lacks native controls around governance items like authorization (who can do what). It also has only a command line interface. Ansible Automation Platform brings a more user-friendly GUI experience and provides role-based-access rights. As such, Ansible playbooks (which are still the engine behind Automation Platform) become subject to structured authorization. Target architectures can now be split into very granular permissions, such that only specifically assigned capabilities will be visible and executable by a user who logs in, based on their assigned rights. This granularity may be mandatory in environments where regulatory compliance is a focus.

Automation Platform also introduces workflow templates that allows jobs, or playbooks, to be sequenced in a graphical manner. The end user who “kicks off” a play does not necessarily see the playbook itself, but they may be entering string variables, making dropdown selections, or pressing radio buttons in a GUI window to populate it. Thus, Ansible Automation Platform is providing additional abstraction from the underlying technology and furthering an “intent based” approach to networking.

Automation Platform contains an API as well enabling programmatic control by higher layer orchestration systems to create and/or kick off workflows. This is especially powerful as an integration point for example with ITSM and CMDB tools like ServiceNow.

## Lab Overview
Each lab utilizes the same basic topology components:
* A dedicated ACI tenant which hosts the test virtual machines within a classic “3-tier” ACI application profile comprising three security zones (EPGs): Web, App, and DB. Full administrative access is granted within this tenant and to all of its related logical components.
* An external services tenant hosts the lab JumpBox and Automation Platform. No administrative rights or visibility of this tenant are provided, however, it is included in topology diagrams for clarity.

<br><br>

# Part 2: Lab Access
## Okta Account Setup
1. Navigate to https://siriussdx.okta.com/
2. Create an account if you do not currently have one by selecting the `Sign up` link at the bottom of the login box.<br>
![](images/okta_login.jpg)

<br>

## Schedule Your Lab
Once logged into Okta, under `My Apps` select `Catalog.`  This will launch the SDx Lab Catalog where you can browse the various available labs.  For this lab navigate [here](https://catalog.siriussdx.com/catalog.php?parent_id=1&category_id=2).<br>
![](images/catalog.jpg)

<br>

## Virtual Desktop
Navigate back to Okta, under `My Apps` select `LAB Access`  This will launch the virtual desktop that you will use to access the lab resources.
* ACI GUI
  1. Launch `FireFox` located on the left menu within the jumpbox.
  2. Login with the appropriate credentials found [here](https://catalog.siriussdx.com/my.labs.php).
  3. ACI will be the default landing page.  However you can use the FireFox bookmark or navigate to https://aci.siriussdx.com if needed.
* Ansible CLI
  1. Launch `Terminal` located on the left menu within the jumpbox.
* Automation Platform GUI
  1. Access `(RHAP) - Ansible Tower` through the Okta `My Apps` page.
  2. Authentication is handled automatically through Okta.

<br><br>

# Part 3: Lab 1 - Ansible CLI
Lab 1 covers the ACI object buildout introducing Ansible as the automation engine, but stops short of securing each of the zones. Review the topology diagram below. At the conclusion of lab 1, you will be able to ping, SSH and send HTTP requests to all test virtual machines in your tenant from your jump box. Additionally, all test virtual machines in your tenant will be able to ping, SSH and send HTTP requests to each other.
1. Review the 3-tier application topology.<br>![](images/topology.jpg)
1. Review the [Lab 1 repository](https://github.com/sdxic/LAB-ACI-Ansible/tree/main/lab1/)
    * Look through the playbooks to learn basic structure and how variables, located in the `vars.yml` file, are referenced.
1. Manually deploy a VRF in ACI
    * In the ACI GUI navigate to `Tenants -> labX -> Networking -> VRFs`.<br>![](images/lab1_step3a.jpg)
    * Right click on VRFs and selected `Create VRF`.<br>
    Enter any value in the `Name` field.<br>
    ***Uncheck: `Create A Bridge Domain`***<br>![](images/lab1_step3b.jpg)<br>
    Select the `Finish` button
1. Repeat step 3 creating a second VRF with a different name.<br>![](images/lab1_step4.jpg)
1. Open a terminal and clone the repository.
    ```bash
    sdx@lab242-130-jb:~/LAB-ACI-Ansible/lab1$ git clone https://www.github.com/sdxic/LAB-ACI-Ansible.git
    Cloning into 'LAB-ACI-Ansible'...
    warning: redirecting to https://github.com/sdxic/LAB-ACI-Ansible.git/
    remote: Enumerating objects: 72, done.
    remote: Counting objects: 100% (72/72), done.
    remote: Compressing objects: 100% (44/44), done.
    remote: Total 72 (delta 29), reused 64 (delta 25), pack-reused 0
    Unpacking objects: 100% (72/72), 636.99 KiB | 3.48 MiB/s, done.
    ```
1. Change to the `LAB-ACI-Ansible/lab1` directory.  List the files in this directory with the `ls` command.
    ```bash
    sdx@lab242-130-jb:~$ cd LAB-ACI-Ansible/lab1
    sdx@lab242-130-jb:~/LAB-ACI-Ansible/lab1$ ls
    collections  deploy_ap_epg.yml  deploy_epg_vmm.yml  deploy_jb_contract.yml  deploy_logical.yml  deploy_vrf_bd.yml  input_validation.yml  remove_all.yml  remove_vrf.yml  vars.yml
    ```
1. Install Ansible Galaxy modules
    *  Before some of the playbooks will run you need to install the supporting modules with the `ansible-galaxy install -r ../collections/requirements.yml` command.
    ```bash
    sdx@lab242-130-jb:~/LAB-ACI-Ansible/lab1$ ansible-galaxy install -r collections/requirements.yml
    Starting galaxy collection install process
    Process install dependency map
    Starting collection install process
    Installing 'cisco.aci:2.6.0' to '/home/sdx/.ansible/collections/ansible_collections/cisco/aci'
    Downloading https://galaxy.ansible.com/download/cisco-aci-2.6.0.tar.gz to /home/sdx/.ansible/tmp/ansible-local-2829fbmicdb3/tmpl3tupgbz
    cisco.aci (2.6.0) was installed successfully
    sdx@lab242-130-jb:~/LAB-ACI-Ansible/lab1$
    ```
1. Edit the `vars.yml` file and replace `<your_lab_id>` and `<your_lab_password>` with the one listed on the [My Labs](https://catalog.siriussdx.com/my.labs.php) page.
    ```
    # Generic Vars
    lab_name: "<your_lab_id>" # Check https://catalog.siriussdx.com/my.labs.php - Format: "lab242-1"
    lab_password: "<your_lab_password>"
    ```
1. Review the `remove_vrf.yml` file.  Note the module `cisco.aci.aci_vrf` that is utilized in the playbook.  This module is written in python and provides the underlying code which takes input from the playbooks and interprets back and forth to the ACI APICs.
1. Run the `remove_vrf.yml` playbook and observe the output.
    ```bash
    sdx@lab242-130-jb:~/LAB-ACI-Ansible/lab1$ ansible-playbook remove_vrf.yml 
    PLAY [Remove all VRFs] **********************************************************************

    TASK [Gathering Facts] **********************************************************************
    ok: [localhost]

    TASK [Query all VRFs] ***********************************************************************
    ok: [localhost -> localhost]

    TASK [Remove a VRF for a tenant] ************************************************************
    changed: [localhost -> localhost] => (item=Deleting VRF: My-VRF-New)
    changed: [localhost -> localhost] => (item=Deleting VRF: Test-VRF)

    PLAY RECAP **********************************************************************************
    localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

    sdx@lab242-130-jb:~/LAB-ACI-Ansible/lab1$ 
    ```
    Note the changes in the ACI GUI live as the playbook runs.  You should see all VFRs previously created will be deleted.<br>
    ![](images/lab1_step6.jpg)
1. Run the `ansible-doc cisco.aci.aci_vrf` command to view the documentation related to this module.  Look towards the bottom of the output to find examples typically containing `query`, `add` and `remove` snippets.<br>
    ```bash
    sdx@lab242-130-jb:~/LAB-ACI-Ansible/lab1$ ansible-doc cisco.aci.aci_vrf
    > ACI_VRF    (/home/user/.ansible/collections/ansible_collections/cisco/aci/plugins/modules/aci_vrf.py)

            Manage contexts or VRFs on Cisco ACI fabrics. Each context is a private network associated to a tenant, i.e. VRF. Enable Preferred Groups for VRF

      * This module is maintained by an Ansible Partner
    OPTIONS (= is mandatory):

    ... redacted ...

    EXAMPLES:

    - name: Add a new VRF to a tenant
      cisco.aci.aci_vrf:
        host: apic
        username: admin
        password: SomeSecretPassword
        vrf: vrf_lab
        tenant: lab_tenant
        descr: Lab VRF
        policy_control_preference: enforced
        policy_control_direction: ingress
        state: present
      delegate_to: localhost

    - name: Remove a VRF for a tenant
      cisco.aci.aci_vrf:
        host: apic
        username: admin
        password: SomeSecretPassword
        vrf: vrf_lab
        tenant: lab_tenant
        state: absent
      delegate_to: localhost

    - name: Query a VRF of a tenant
      cisco.aci.aci_vrf:
        host: apic
        username: admin
        password: SomeSecretPassword
        vrf: vrf_lab
        tenant: lab_tenant
        state: query
      delegate_to: localhost
      register: query_result
    ```
1. Run playbook `deploy_ap_epg.yml` and review any EPG faults for the newly created objects.  Navigate to `Application Profiles -> ap -> Application EPGs -> web_ep`.  Select `Faults` from the right side menu.<br>
    ```bash
    sdx@lab242-130-jb:~/LAB-ACI-Ansible/lab1$ ansible-playbook deploy_ap_epg.yml 
    PLAY [Deploy Application Profile and EPGs] **************************************************

    TASK [Gathering Facts] **********************************************************************
    ok: [localhost]

    TASK [Deploy Application Profile] ***********************************************************
    ok: [localhost -> localhost]

    TASK [Deploy Endpoint Group] ****************************************************************
    changed: [localhost -> localhost] => (item=web_ep)
    changed: [localhost -> localhost] => (item=app_epg)
    changed: [localhost -> localhost] => (item=db_epg)

    PLAY RECAP **********************************************************************************
    localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

    sdx@lab242-130-jb:~/LAB-ACI-Ansible/lab1$ 

    ```
    ![](images/lab1_step9a.jpg)<br>
    Why are we seeing fault(s)? What do the fault(s) mean and how can we fix them?
1. Run playbook `deploy_vrf_bd.yml` and observe the faults page for any changes.  You should see the lifecycle status change to `"Soaking, Clearing"` or `"Raised, Clearing"` or similar.<br>
    ```bash
    sdx@lab242-130-jb:~/LAB-ACI-Ansible/lab1$ ansible-playbook deploy_vrf_bd.yml 
    PLAY [Deploy VRF and Bridge Domain] *********************************************************

    TASK [Gathering Facts] **********************************************************************
    ok: [localhost]

    TASK [Deploy VRF] ***************************************************************************
    changed: [localhost -> localhost]

    TASK [Deploy Bridge Domain] *****************************************************************
    changed: [localhost -> localhost]

    TASK [Deploy Bridge Domain Subnet] **********************************************************
    changed: [localhost -> localhost]

    PLAY RECAP **********************************************************************************
    localhost                  : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

    sdx@lab242-130-jb:~/LAB-ACI-Ansible/lab1$ 
    ```
    ![](images/lab1_step10.jpg)<br>
    > *Note: it may take up to 2 minutes for the faults to fully clear.*
1. Run playbook `remove_all.yml` and observe changes in the ACI GUI.  What did you notice?  Your tenant should be blank and all VRFs, EPGs, APs and BDs will have been removed.
    ```bash
    sdx@lab242-130-jb:~/LAB-ACI-Ansible/lab1$ ansible-playbook remove_all.yml
    PLAY [Remove all VRFs BDs and APs] **********************************************************

    TASK [Gathering Facts] **********************************************************************
    ok: [localhost]

    TASK [Query all VRFs] ***********************************************************************
    ok: [localhost -> localhost]

    TASK [Remove a VRF for a tenant] ************************************************************
    changed: [localhost -> localhost] => (item=Deleting VRF: vrf)

    TASK [Query all AP in Tn] *******************************************************************
    ok: [localhost -> localhost]

    TASK [Remove the APs] ***********************************************************************
    changed: [localhost -> localhost] => (item=Deleting AP: ap)

    TASK [Query all Bridge Domains in Tn] *******************************************************
    ok: [localhost -> localhost]

    TASK [Delete all the Bridge Domains Found] **************************************************
    changed: [localhost -> localhost] => (item=Deleting BD: bd)

    PLAY RECAP **********************************************************************************
    localhost                  : ok=7    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

    sdx@lab242-130-jb:~/LAB-ACI-Ansible/lab1$ 
    ```
1. Run playbook `deploy_logical.yml`.  Review the objects that are created in the ACI GUI.  You should see all of the objects that were previously deleted have been redeployed using a single playbook.
    ```bash
    sdx@lab242-130-jb:~/LAB-ACI-Ansible/lab1$ ansible-playbook deploy_logical.yml 
    PLAY [Deploy Full Logical Configuration] ****************************************************

    TASK [Gathering Facts] **********************************************************************
    ok: [localhost]

    TASK [Deploy VRF] ***************************************************************************
    changed: [localhost -> localhost]

    TASK [Deploy Bridge Domain] *****************************************************************
    changed: [localhost -> localhost]

    TASK [Deploy Bridge Domain Subnet] **********************************************************
    changed: [localhost -> localhost]

    TASK [Deploy AP] ****************************************************************************
    changed: [localhost -> localhost]

    TASK [Deploy EPG] ***************************************************************************
    changed: [localhost -> localhost] => (item=web_epg)
    changed: [localhost -> localhost] => (item=app_epg)
    changed: [localhost -> localhost] => (item=db_epg)

    PLAY RECAP **********************************************************************************
    localhost                  : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

    sdx@lab242-130-jb:~/LAB-ACI-Ansible/lab1$ 
    ```
1. Run playbook `deploy_jb_contract.yml` and verify the consumed contract `LAB-JB` now exists on the VRF as a vzAny contract.
    * Navigate to `<Your Tenant>` -> `Networking` -> `VRFs` -> `"vrf"` -> `EPG|ESG Collection for VRF`
    * Scroll to the bottom secion and look for `Contract Interfaces`.  You should see the `LAB-JB` consumed contract interface listed.
    <br>![](images/lab1_step15.jpg)<br>
1. Verify connectivity from your jumpbox to each of the test virtual machines:
    * web1
    * web2
    * app1
    * app2
    * db1
    ```bash
    sdx@lab242-130-jb:~/LAB-ACI-Ansible/lab1$ ping web1 -c 3
    PING web1 (10.242.1.11) 56(84) bytes of data.
    64 bytes from lab242-1-web1 (10.242.1.11): icmp_seq=1 ttl=64 time=0.024 ms
    64 bytes from lab242-1-web1 (10.242.1.11): icmp_seq=2 ttl=64 time=0.020 ms
    64 bytes from lab242-1-web1 (10.242.1.11): icmp_seq=3 ttl=64 time=0.019 ms

    --- web1 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2045ms
    rtt min/avg/max/mdev = 0.019/0.021/0.024/0.002 ms
    ```
1. Remove all configuration
    * Before proceeding to lab 2 re-run the `remove_all.yml` to remove all of the existing configuration from your ACI tenant.

<br>

><br>Lab 1 Complete!  You should now be familiar with basic Ansible and ACI concepts and integrations.  Proceed to Lab 2 for an introduction into Red Hat Ansible Automation Platform.<br>&nbsp;

<br><br>

# Part 4: Lab 2 - Automation Platform GUI
Lab 2 deploys the same environment and then layers security functionality over it, achieving the entire deployment using Ansible Automation Platform job templates and workflow templates. You will create your own Automation Platform project, and build and run the templates.  Review the topology diagrams below to familiarize yourself with the various states the logical configuration will enter.  These portray how lab 2 incrementally introduces additional ACI security functionality to better secure the application profile created in lab 1, but it introduces Ansible Automation Platform to facilitate the changes.

*** Access Okta to launch the `Automation Platform` GUI ***

1. Create credentials for ACI
    * Select `Resources -> Credentials` under the left menu and then the `Add` button to create a new set of credentials.
    * Fill in a `Name` (e.g. `ACI Lab Creds`) and set `Credential Type` = `Network`.
    * Input your lab username in the format `labX-Y`.  (e.g. `lab242-160`).  Find your username & password [here](https://catalog.siriussdx.com/my.labs.php).
    * Input your lab password.
    * Select the `Save` button.
    <br>![](images/lab2_step1.jpg)<br>
1. Create a project and sync with repository.  Projects are references to the codebase that contains playbooks.
    * Select `Resources -> Projects` under the left menu and then the `Add` button to create a new project.
    * Fill in a `Name` and select `Git` as the `Source Control Type`.  This will expand more options to fill out below.
    * Under `Source Control URL` enter `https://github.com/sdxic/LAB-ACI-Ansible.git`.
    * Click on the question marks next to each of the checkboxes towards the bottom: Clean, Delete, Track submodules, etc. for a description of what each does.  It's recommended to select Clean, Delete and Update Revision on Launch.
    * Select the `Save` button.
    * *Note: when you save the project it will automatically begin a sync to the repository.  If you have any errors it will fail, otherwise the playbooks will automatically sync to Automation Platform.
    <br>![](images/lab2_step2.jpg)<br>
1. Create an inventory for ACI.
    * Every Job Template requires an inventory, which is how Ansible knows what to execute the playbooks against.  Some modules, such as the ACI modules, do not require the host inventory to be explicitly set, rather the module accepts a `hostname` variable and relies on it for sending API calls to the proper ACI endpoint.
    * Select `Resources -> Inventories` under the left menu and then the `Add` button to create an inventory.  This will be a dummy inventory only to be used as a placeholder.
    * Fill in a `Name` and select the `Save` button.
    * Select the `Save` button.
    <br>![](images/lab2_step3.jpg)<br>
1. Create `Initial Config` job template.
    * Job templates tie to individual playbooks and have various customizations that can be applied.  Explore the fields while walking through these steps.
    * Navigate to `Resources -> Templates` and select the `Add` button and `Add job template` to create a new job template.
    * Fill in the `Name` field and select the previously created `Inventory`.
    * Since you should only have 1 `Project` it may be selected by default, if not simply select the previously created project.
    * Select the `Organization` that matches your username / lab ID if it was not auto populated.
    * Select the dropdown for `Playbook` and notice the options listed.  Select `lab2/deploy_logical.yml`. *note the leading "lab2/"*
    * Select `Credentials`.  Within the popup window select the `Category` dropdown and select `Network`. At the bottom select `ACI Lab Creds` and press the `Select` button.
    * Select the `Save` button.
    <br>![](images/lab2_step4.jpg)
1. Create `JumpBox Contract` job template.
    * Navigate to `Resources -> Templates` and select the `Add` button and `Add job template` to create a new job template.
    * Fill in the `Name` field and select the previously created `Inventory`.
    * Select the previously created project if not already populated.
    * Select the `Organization` that matches your username / lab ID if it was not auto populated.
    * Select the dropdown for `Playbook` and notice the options listed.  Select `lab2/deploy_vzany_contract.yml`.
    * Select `Credentials`.  Within the popup window select the `Category` dropdown and select `Network`. At the bottom select `ACI Lab Creds` and press the `Select` button.
    * Select the `Save` button.
1. Run the `Initial Config` and `JumpBox Contract` job templates.
    * Navigate to `Resources` -> `Templates` and select the Rocket Ship icon to the right of the `Initial Config` template that was previously created.
    * This will launch the job template and start displaying the output to the screen.
    * Once complete run the `JumpBox Contract` job template.
    <br>![](images/lab2_step6.jpg)
1. Review job execution history and details.
    * Navigate to `Jobs` on the left menu.  Notice there are several jobs other than the 2 you just ran.  You should see some related to inventory and project sync, which if you recall, will pull down the playbooks every time a job is executed.
    * Select any of the jobs and then select `Output` along the top menu.  Here you can review output from playbooks and other tasks.
    * Select `Details` from the top menu and notice the information you can use for troubleshooting such as the name of the playbook, dates and times, etc.
    <br>![](images/lab2_step7.jpg)
    * The topology should now be in the state as shown in the diagram below.  This will match the end state of lab 1 previously.
    <br>![](images/lab2_step7a.jpg)
1. Create and run `VRF Enforcement` job template.
    * Use playbook `lab2/vrf_enforce_pg.yml`.
    * This playbook will turn `VRF enforcement` on, which requries contracts between all EPGs to allow communication.
    * The `preferred group` option is also enabled with this playbook.
    * Verify in the GUI these values are set correctly after launching the job template.
    <br>![](images/lab2_step8.jpg)
    * With VRF enforcement on the `web`, `app`, and `db` servers should no longer be able to reach each other.
1. Create and run `Enable Preferred Group` job template.
    * Use playbook `lab2/enable_pg.yml`.
    * This playbook will configure the `web` and `app` EPGs to be in the `preferred group` which allows communication between any EPG in the preferred group without a contract.
    * Verify in the GUI these values are set correctly after launching the job template.
    <br>![](images/lab2_step9.jpg)
    * Notice now the `web` and `app` servers are able to reach each other again after being placed in the `Preferred Group`.
    <br>![](images/lab2_step9a.jpg)
1. Create `Remove JumpBox Contract` job template.
    * Use playbook `lab2/remove_vzany_contract.yml`.
    * This playbook will remove the JumpBox contract that was applied to the vzAny earlier.
    * ***do not run this playbook yet***
1. Create `App-DB Contract` job template.
    * Use playbook `lab2/app_db_contract.yml`.
    * This playbook will create and apply a contract provide by the `db_epg` and consumed by the `app_epg`.
    * ***do not run this playbook yet***
1. Create `Web App JB Contract`
    * Use playbook `lab2/web_app_jb_contract.yml`.
    * This playbook will apply a consumed contract interface to the `web_epg` and `app_epg` enabling communication from your JumpBox.
    * ***do not run this playbook yet***
1. Create and run `Strict Access` ***workflow*** template.
    * This will create a `workflow template` which is a sequence of `job templates` stitched together to run as a whole.
    * Navigate to `Resources -> Templates` and select the `Add` button and `Add workflow template` to create a new workflow template.
    * Fill in the `Name` field and select the `Organization` that matches your username / lab ID if it was not auto populated.
    * Select the `Save` button then you will then see the workflow visualizer page.
    * Select the green `Start` button.
    * Now you will add each of the `job templates` to the workflow.
        * From the list add the `Remove JumpBox Contract` and select the `Save` button. 
        * Notice that job template now shows up in the workflow visualizer.  Hover over the `Remove JumpBox Contract` box to see a menu open of available options.
        <br>![](images/lab2_step13.jpg)
        * Select the `plus` icon to add another job template to the workflow. This time you will be presented with a new screen with `On Success`, `On Failure` and `Always` options.  This determines when to execute this next job template based on the output of the preiouvs.  You can begin to see the flexibility of workflow templates and how you can build them to react based upon success criteria.
        * Select `On Success` and the `Next` button. On the next screen choose `App-DB Contract` and select `Save`.
        * Repeat this step and add `Web App JB Contract` as a third job in the workflow.
        * Once complete select the `Save` button in the upper right corner to return.
    * Launch the workflow and observe the visual output it provides.
    <br>![](images/lab2_step13a.jpg)
    * ***Note:*** Only the `app_epg` will have access to the `db_epg`.  Your JumpBox and `web_epg` should NOT be able to ping, SSH or HTTP to the `db_epg` once these are applied.
    * See the diagram below depicting the current logical state:
    <br>![](images/lab2_step13b.jpg)
1. Create and run `Disable Preferred Group` job template.
    * Use playbook `lab2/disable_pg.yml`.
    * This playbook will remove `web_egp` and `app_epg` from the preferred group and disable the option under the VRF.
    * Verify in the GUI these values are set correctly after launching the job template.
    <br>![](images/lab2_step14.jpg)
    <br>![](images/lab2_step14a.jpg)
1. Create and run `Web-App Contract` job template.
    * Use playbook `lab2/web_app_contract.yml`.
    * This playbook will create apply a consumed contract on `web_epg` and provided contract on `app_epg` enabling communication between the web and app servers.
    * Verify in the GUI these values are set correctly after launching the job template.
    <br>![](images/lab2_step15.jpg)
    <br>![](images/lab2_step15a.jpg)
    * The final logical state is shown below.
    <br>![](images/lab2_step15b.jpg)

><br>Lab 2 Complete!  You should now be familiar with basic tasks in Red Hat Ansible Automation Plaform.<br>&nbsp;