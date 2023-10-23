# Title: Deploy Laravel and Set up Postgresql.


## Description: This is a documentation on the deployment of Laravel application on two vagrant machines (Master$Slave_1) using bash scripts and an ansible playbook.

# Step 1 : create vms using bash script {node.sh}
## node.sh
## Vagrant Multi-Machine Setup Script

- This Bash script sets up a Vagrant environment with two virtual machines: one named "master" and the other named "slave_1". It uses the `ubuntu/focal64` box as the base image.

- #!/bin/bash: This is called a "shebang" line. It tells the system that the script should be executed by the Bash shell.

- vagrant init ubuntu/focal64: This command initializes a new Vagrant environment using the base box ubuntu/focal64. A base box is a pre-configured virtual machine image that Vagrant uses as a starting point for creating new VMs.

- cat <<EOF > Vagrantfile: This starts a "here document" block in Bash. It allows you to write multiple lines into a file. In this case, it's writing content into a file named Vagrantfile.

- Vagrant.configure("2") do |config|: This line begins the configuration block for Vagrant, specifying version 2 of the Vagrant configuration syntax.

- Inside this configuration block, there are two VM definitions: slave_1 and master.
a. config.vm.define "slave_1" do |slave_1|: This defines a virtual machine named slave_1. The configuration for this VM will be specified within the block.
b. Inside the slave_1 block, there are several configurations:
slave_1.vm.hostname = "slave-1": Sets the hostname of the VM to "slave-1".
slave_1.vm.box = "ubuntu/focal64": Specifies the base box for this VM.
slave_1.vm.network "private_network", ip: "192.168.20.11": Sets up a private network interface with a static IP address of 192.168.20.11.
slave_1.vm.provision "shell", inline: <<-SHELL ... SHELL: Specifies provisioning instructions for the slave_1 VM. It uses a shell script for provisioning. This script does the following:
Updates the package repositories (apt-get update).
Upgrades installed packages (apt-get upgrade -y).
Installs avahi-daemon and libnss-mdns.
c. The config.vm.define "master" do |master| block is similar to the slave_1 block, but it configures a virtual machine named master.
d. Inside the master block, there are configurations similar to those for slave_1.

- config.vm.provider "virtualbox" do |vb| ... end: This section sets up provider-specific configuration for VirtualBox. It allocates 1024 MB of memory and 2 virtual CPUs for both VMs.

- EOF: This marks the end of the "here document" block. It signals the end of the content that is being written into the Vagrantfile.

- Finally, vagrant up is the command to start up the VMs according to the configurations defined in the Vagrantfile.

- in summary, this script automates the process of setting up two Ubuntu 20.04 VMs with specific configurations, including hostnames, IP addresses, and provisioning tasks. It uses Vagrant to manage the VMs and VirtualBox as the virtualization provider.

# Step 2: automating the deployment of LAMP (Linux,Apache,MySQL,PHP) stack and cloning of the PHP application from GitHub.
## laravel-master.sh

- Updating and Upgrading the Server:
sudo apt update && sudo apt upgrade -y < /dev/null: Updates and upgrades the system packages without user interaction (< /dev/null is used to ensure there is no user input required).

- Installation of LAMP Stack:
Installs Apache, MySQL Server, and PHP along with necessary modules.
sudo apt-get install apache2 -y < /dev/null
sudo apt-get install mysql-server -y < /dev/null
Adds the PHP repository and installs PHP modules.
Adjusts the php.ini configuration to fix a path issue.
Restarts Apache.

- Installing Composer:
Installs Composer (a dependency manager for PHP).
sudo apt install curl -y
Downloads and installs Composer.
Moves Composer to /usr/local/bin for global access.
Checks Composer version.

- Configuring Apache:
Creates a virtual host configuration for the Laravel application.
Enables mod_rewrite.
Enables the Laravel site.
Restarts Apache.

- Cloning Laravel Application and Dependencies:
Creates a directory for the Laravel application and navigates into it.
Clones the Laravel repository from GitHub.
Installs application dependencies using Composer without dev dependencies.
Sets ownership and permissions for files and directories.
Copies the environment configuration file and generates a unique application key.

- Configuring MySQL:
Creates a MySQL database and user.
Randomly generates a password if none is provided.
Grants privileges and flushes privileges.

- Executing Key Generation and Migration Commands for PHP:
Updates the .env file with the database name, username, and password.
Clears the configuration cache.
Migrates the database.

- Please note:

This script assumes that it's being run with at least one argument, which should be the desired database name. An optional second argument can be used to set a custom MySQL password.
It's essential to understand that this script assumes a specific environment and configuration. Be cautious when running scripts with elevated privileges.

# Step 3: Using Ansible playbook; execute the bash script on the slave and verify that the PHP application is accessible through the VM's IP address.
## ansibleplaybook.yaml

- ---: This is a YAML file separator, indicating the start of a new YAML document.

- hosts: all: This specifies that the playbook will run on all hosts defined in your inventory.

- become: yes: This tells Ansible to use sudo or escalate privileges to become the superuser when executing tasks.

- pre_tasks: This section contains a list of tasks to be executed before any actual tasks defined later in the playbook.

- name: update & upgrade server: This task is named "update & upgrade server". It uses the apt module to update the package cache and upgrade all packages on the server.

- name: set cron job to check uptime...: This task uses the cron module to schedule a cron job. This job will run the command /usr/bin/uptime > /var/log/uptime_check.log 2>&1 at midnight (minute: "0", hour: "0", day: "*", month: "*", weekday: "*").
- name: copy the bash script to slave machine: This task uses the copy module to transfer a file named laravel-slave.sh from the local machine to the remote machine's home directory (~), owned by the root user.
- name: Set Execute Permissions on the Script: This task uses the command module to run the chmod +x ~/laravel-slave.sh command, which grants execute permissions to the script.
- name: Run Bash Script: This task uses the command module to execute the bash laravel-slave.sh sijuwade sijuwade007 < /dev/null command. It is running the laravel-slave.sh script with two arguments: sijuwade and sijuwade007. < /dev/null is used to ensure there is no user input required.

- Please note:

- The laravel-slave.sh script seems to be related to a Laravel application setup. It's assumed that this script exists in the same directory as the playbook.
- This playbook assumes that you have appropriate permissions and that the specified tasks are suitable for the target hosts.
Ensure that you have a valid inventory file with the correct hostnames or IP addresses defined.

# Laravel-slave.sh

- Updating and Upgrading the Server:
sudo apt update && sudo apt upgrade -y < /dev/null: Updates and upgrades the system packages without user interaction (< /dev/null is used to ensure there is no user input required).

- Installation of LAMP Stack:
Installs Apache, Ansible, MySQL Server, and PHP along with necessary modules.
sudo apt-get install apache2 -y < /dev/null
sudo apt install ansible -y < /dev/null
sudo apt-get install mysql-server -y < /dev/null
Adds the PHP repository and installs PHP modules.
Adjusts the php.ini configuration to fix a path issue.
Restarts Apache.

- Installing Composer:
Installs Composer (a dependency manager for PHP).
sudo apt install curl -y
Downloads and installs Composer.
Moves Composer to /usr/local/bin for global access.
Checks Composer version.

- Configuring Apache:
Creates a virtual host configuration for the Laravel application.
Enables mod_rewrite.
Enables the Laravel site.
Restarts Apache.

- Cloning Laravel Application and Dependencies:
Creates a directory for the Laravel application and navigates into it.
Clones the Laravel repository from GitHub.
Installs application dependencies using Composer without dev dependencies.
Sets ownership and permissions for files and directories.
Copies the environment configuration file and generates a unique application key.

- Configuring MySQL:
Creates a MySQL database and user.
Randomly generates a password if none is provided.
Grants privileges and flushes privileges.

- Executing Key Generation and Migration Commands for PHP:
Updates the .env file with the database name, username, and password.
Clears the configuration cache.
Migrates the database.

- Please note:

The laravel.conf Apache configuration assumes that your Laravel application will be accessible at 192.168.20.11. If this is not the case, you'll need to adjust it.


# Step 4: Creating a cron job to check the server's uptime every 12 am. ## From the ansible playbook; ansibleplaybook.yaml

- name: set cron job to check uptime...: This task uses the cron module to schedule a cron job. This job will run the command /usr/bin/uptime > /var/log/uptime_check.log 2>&1 at midnight (minute: "0", hour: "0", day: "*", month: "*", weekday: "*").

## ansible dependencies:
- ansible.cfg

- [defaults]: This section header indicates that the following configurations apply to the default behavior of Ansible.

- inventory = inventory: This line specifies the location of the inventory file that Ansible should use. In this case, it's set to a file named inventory.

- private_key_file = ~/.ssh/id_rsa: This line specifies the location of the private SSH key file that Ansible should use when connecting to remote hosts. In this case, it's set to ~/.ssh/id_rsa, which is the default location for the SSH private key.

- host_key_checking = False: This line disables host key checking. When set to False, Ansible won't check if a host's key is already known in the known_hosts file. This can be useful in situations where hosts may have dynamically changing keys (e.g., in a cloud environment).

- These configurations are helpful for specifying the default behavior of Ansible when it's used in a project. The inventory file is where you define your hosts, and private_key_file ensures Ansible uses the correct SSH key for connecting to those hosts.

- Inventory

- In Ansible, an inventory is a file or collection of files that defines the list of hosts (remote machines) that Ansible can connect to and manage. It provides a way to organize and manage the target hosts for your automation tasks.

- An inventory file typically contains information such as;

- Hostnames or IP addresses: These are the addresses of the remote machines that Ansible will interact with.


# Screenshots from the project


- [php application on the master node](/screenshots/Laravel%20on%20the%20master%20node.png)

- [php application on slave node](/screenshots/Laravel%20on%20the%20slave%20node.png)

- [further SC](/screenshots/Screenshot%202023-10-21%20at%2017.26.07.png)

- [further SC](/screenshots/Screenshot%202023-10-21%20at%2017.40.10.png)

- [further SC](/screenshots/Screenshot%202023-10-21%20at%2018.04.37.png)

- [further SC](/screenshots/Screenshot%202023-10-21%20at%2018.14.56.png)

- [further SC](/screenshots/Screenshot%202023-10-21%20at%2018.17.32.png)

- [further SC](/screenshots/Screenshot%202023-10-21%20at%2018.19.15.png)














 