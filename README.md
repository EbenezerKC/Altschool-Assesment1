
# (Deploy LAMP Stack)

Automate the provisioning of two Ubuntu-based servers, named “Master” and “Slave”, using Vagrant.
On the Master node, create a bash script to automate the deployment of a LAMP (Linux, Apache, MySQL, PHP) stack.
This script should clone a PHP application from GitHub, install all necessary packages, and configure Apache web server and MySQL. 
Ensure the bash script is reusable and readable.
Using an Ansible playbook:
Execute the bash script on the Slave node and verify that the PHP application is accessible through the VM’s IP address (take screenshot of this as evidence)
Create a cron job to check the server’s uptime every 12 am.


## Documentation

You should have a Vagrantfile for the Virtual Machines saved in your local file folder just like mine below after you start your machine. But if you don't I can walk you through it. Remember you need to create 2 Ubuntu servers named "master" and "slave". Open your Virtual box down as you will need it open to start your Virtual Machines. Create two folders and name them "master" and "slave".

1. Open the folders with Gitbash. Let us start with the "master" folder.
2. Next is to Initialize your master Ubuntu machine. After opening the "master" folder with git bash, run:
3. Open the Vagrantfile and make some edits to the configuration file using the command nano vagrantfile. 

4. Go ahead to paste the configuration 

# Provision master Node
config.vm.define "master" do |master|
master.vm.box = "ubuntu/focal64"
master.vm.hostname = "master"
master.vm.network "private_network", ip: "192.168.33.10", type: "dhcp"
end

just under Vagrant.configure("2") do |config|. 

Also, add config.ssh.insert_key = false
under config.vm.box = ubuntu/focal64. 
Press "crtl+O" to save, then "enter" and "ctrl+x" to exit the vagrantfile.

Then run vagrant up to start the machine

Connect the Virtual machine by runing vagrant ssh

Now that you have logged into your "masters" virtual machine, follow the same procedure for the "slave" virtual machine.

NOTE: Every other thing is the same for the procedure except for the configuration in the vagrant file which should be under Vagrant.configure("2") do |config|.Use the command below under the Vagrant.configure("2") do |config|.

# Provision Slave Node
config.vm.define "slave" do |slave|
slave.vm.box = "ubuntu/focal64"
slave.vm.hostname = "slave"
slave.vm.network "private_network", ip: "192.168.33.11", type: "dhcp"
end

Connect the "master" and "slave" machine by _ssh-ing _into each of them
Use the command ip a to check for the IP address of each of the virtual machines.
Do the same for the "slave" virtual machine.

On your "master" Virtual Machine, write ssh-keygen to generate a private & public key. When you run the command you will see Enter file in which to save the key (/home/vagrant/.ssh/id_rsa): Enter passphrase (empty for no passphrase): Enter same passphrase again: Keep pressing the "enter" key when those 3 lines appear

Repeat the above step for the "slave" virtual machine

In the "master" virtual machine, change the directory into ssh with the command cd .ssh and then ls to list all the files in the ssh directory. 

Open the content of id_rsa.pub and copy it to save it somewhere for later use. That is where your public key is and that is what you are copying. Use this command to open it cat id_rsa.pub. 

Change the directory to .ssh with the command cd .ssh just as we did in the "master" virtual machine. Create and open a file in the .ssh directory of the "slave" machine at once so that the public key you copied from the "masters" machine and saved somewhere, you will paste it in the created file. I named my file 'public'. Run the command nano public and paste the public key from the "master" virtual machine here. After pasting all the public keys correctly, press 'crtl+O' to save, press 'enter', and then press 'ctrl+X' to exit.

Then run the command cat public >> authorized_keys to copy the public keys into the authorized_keys file from the file named public in the "slave" machine. Run the command cat authorized_keys to see the content of the copied public keys of the "masters" virtual machine and the "slave" virtual machine.

Connect the "master" virtual machine to the "slave" virtual machine by using ssh to connect the IP address of the "slave" machine to the "master" machine. First of all, run the command ip a to know the IP addresses in each machine. Then run the command ssh vagrant@slave ip in the "master" virtual machine. It should look like this ssh vagrant@192.168.33.8 Make sure to replace the IP address with your own "slave" IP address. Whatever it prompts you, type 'yes'. From the screenshot, you can see that we can connect to our "slave" machine from the "master" machine.

Creating the Bash Script and Ansible to automate the provisioning of the two Ubuntu-based Virtual Machines we've created.
In your "master" virtual machine, create your script that will automate the deployment of LAMP (Linux, Apache, MySQL, PHP) stack.

Name your script anything of your choice with the .sh extension. I will be naming mine LAMP.sh. Use the command touch LAMP.sh to create your script. run ls to see if the script file you ran is showing.

Run nano LAMP.sh to open the file script and write your script. Please copy and paste this script inside it. Each line of the command has a comment with a brief explanation with a '#' symbol in front of it. I made sure to pick each line of commands one by one to run before compiling it into a full script below.


#!/bin/bash

# Define variables for paths and IP address
LARAVEL_DIR="/var/www/html/laravel"
LARAVEL_CONF="/etc/apache2/sites-available/laravel.conf"
VIRTUAL_HOST="192.168.33.8"

# Update all package index and upgrade packages
sudo apt update
sudo apt upgrade -y
echo "Done with upgrade package"

# Update the PHP repository
sudo add-apt-repository -y ppa:ondrej/php
echo "Done with php repo update"

# Install Apache
sudo apt install -y apache2
sudo systemctl enable apache2
echo "Done with Apache installation"

# Install MySQL
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password'
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password'
sudo apt install -y mysql-server
echo "Done with MySQL installation"

# Install PHP 8.2 and 8.3 with necessary extensions
sudo apt install -y php libapache2-mod-php php-mysql php8.2 php8.2-curl php8.2-dom php8.2-xml php8.2-mysql php8.2-sqlite3 php8.3 php8.3-curl php8.3-dom php8.3-xml php8.3-mysql php8.3-sqlite3
echo "Done with PHP 8.2 and 8.3 installation"

# Run MySQL secure installation
expect <<EOF
spawn sudo mysql_secure_installation
expect "Would you like to setup VALIDATE PASSWORD component?"
send "y\r"
expect {
    "Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG" {
        send "1\r"
        exp_continue
    }
    "Remove anonymous users?" {
        send "y\r"
        exp_continue
    }
    "Disallow root login remotely?" {
        send "n\r"
        exp_continue
    }
    "Remove test database and access to it?" {
        send "y\r"
        exp_continue
    }
    "Reload privilege tables now?" {
        send "y\r"
        exp_continue
    }
}
EOF
echo "Done with MySQL secure installation"

# Restart Apache
sudo systemctl restart apache2
echo "Done with Apache restart"

# Install Git
sudo apt install -y git
echo "Done with Git installation"

# Clone Laravel repository
sudo git clone https://github.com/laravel/laravel $LARAVEL_DIR
echo "Done with cloning Laravel repository"

# Change directory to Laravel folder
cd $LARAVEL_DIR
echo "Changed directory to $LARAVEL_DIR"

# Install Composer
sudo apt install -y composer
echo "Done with installing Composer"

# Upgrade Composer to version 2
sudo php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
sudo php -r "if (hash_file('sha384', 'composer-setup.php') === 'dac665fdc30fdd8ec78b38b9800061b4150413ff2e3b6f88543c636f7cd84f6db9189d43a81e5503cda447da73c7e5b6') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
sudo php composer-setup.php --install-dir /usr/bin --filename composer
echo "Done with upgrading Composer to version 2"

# Use Composer to install dependencies
yes | sudo composer install
echo "Done installing Composer dependencies"

# Copy Laravel configuration file and set permissions
sudo cp .env.example .env
sudo chown www-data:www-data .env
sudo chmod 640 .env
echo "Done copying Laravel configuration file and setting permissions"

# Create virtual host in /etc/apache2/sites-available
sudo tee $LARAVEL_CONF >/dev/null <<EOF
<VirtualHost *:80>
    ServerName $VIRTUAL_HOST
    ServerAlias *
    DocumentRoot $LARAVEL_DIR/public

    <Directory $LARAVEL_DIR>
        AllowOverride All
    </Directory>
</VirtualHost>
EOF
echo "Done creating virtual host in /etc/apache2"

# Generate application key
sudo php artisan key:generate
echo "Done generating application key"

# Run migrations
sudo php artisan migrate --force
echo "Done running migrations"

# Change ownership permissions
sudo chown -R www-data:www-data $LARAVEL_DIR/database/ $LARAVEL_DIR/storage/logs/ $LARAVEL_DIR/storage $LARAVEL_DIR/bootstrap/cache
echo "Done changing ownership permissions"

# Set file permissions
sudo chmod -R 775 $LARAVEL_DIR/database/ $LARAVEL_DIR/storage/logs/ $LARAVEL_DIR/storage
echo "Done setting file permissions"

# Disable default configuration file
sudo a2dissite 000-default.conf
echo "Done disabling default configuration file"

# Enable Laravel configuration file
sudo a2ensite laravel.conf
echo "Done enabling Laravel configuration file"

# Restart Apache
sudo systemctl restart apache2
echo "Done restarting Apache"


uptime > /var/log/uptime.log


Press 'crtl+O' to save, press 'enter' and then 'crtl+X' to exit the script.

Test run the script to see if it works by running the command ./LAMP.sh

Still in your "master" machine create an Ansible directory with the command:

mkdir Ansible

Change into the Ansible directory so it becomes your present working directory:

cd Ansible

Run this command to update any dependencies:

sudo apt-get update

Then to install Ansible run the command below. It would be best if you had something like the image at the end of the installation:

sudo apt install ansible


Still, in your Ansible directory, create your inventory. The inventory file is used to store the address of the slave nodes to be configured which will enable the playbook to run in the "slave" machine even if it is created in the "master's" machine:

touch inventory 

Open your inventory file to put in the IP address of your "slave" machine. Press 'crtl+O' to save, press 'enter', and then 'crtl+X' to exit the text editor.

Create your playbook with the .yml extension:

touch playbook.yml

To ping your inventory run the command in your "master" machine. It should look like the screenshot below. Press 'crtl+C' to cancel it afterwards:

ansible all -m ping -i inventory



Change the directory back to Ansible with the command cd Ansible. Then open the playbook with the command nano playbook.yml. Paste the command below inside your playbook but change the IP address to your "slave" machine IP. Press 'crtl+O' to save, press 'enter', and then 'crtl+X' to exit the playbook.



---

- hosts: slave
  become: yes
  tasks:

    # Copy LAMP.sh script to the slave node
    - name: Copy LAMP.sh script to slave node
      copy:
        src: /home/vagrant/Ansible/LAMP.sh
        dest: /home/vagrant/LAMP.sh
        mode: 0755  

      # Make the script executable

    # Execute the LAMP.sh script on the slave node
    - name: Execute LAMP.sh script on slave node
      shell: /home/vagrant/LAMP.sh


- name: Create a cron job to check server's uptime every 12 am
  hosts: slave
  become: yes
  tasks:

    # Create an uptime script that logs the server's uptime to a file
    - name: Create the uptime script
      copy:
        content: |

          #!/bin/bash

          # Log the server's uptime to a file
          uptime > /var/log/uptime.log
        dest: /usr/local/bin/check_uptime.sh
        mode: '0755'  # Make the script executable

    # Create a cron job that runs the uptime script every day at 12 am (midnight)

    - name: Create a cron job to check uptime every 12 am
      cron:
        name: Check uptime every 12 am
        job: "/usr/local/bin/check_uptime.sh"

        # Run at 12:00 am
        minute: '0'  
        hour: '0'
        state: present

    # Display the contents of the uptime log file
    - name: Display uptime log file
      shell: cat /var/log/uptime.log
      register: uptime_output

    # Display the server's uptime as captured in the uptime log file
    - name: Display the server's uptime
      debug:
        msg: "{{ uptime_output.stdout }}"

    # Task to fetch the content of the PHP application using curl
    - name: Fetch PHP application content using curl
      command: "curl -s http://192.168.33.8"  # Perform a silent HTTP GET request to the target IP address
      register: php_application  # Register the output of the curl command
      ignore_errors: true  # Continue playbook execution even if this task fails

    # Task to display the content fetched from the PHP application
    - name: Display content of PHP application
      debug:
        msg: "PHP Application Content:\n{{ php_application.stdout }}"  # Display the output of the curl command



Run the command below to execute the playbook:


ansible-playbook playbook.yml -i inventory