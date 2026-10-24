# Deploy, Secure, and Automate Full-Stack Web Applications

*From cloud infrastructure provisioning to secure, automated production deployments.*

## Introduction

Deploying a web application involves more than writing code. It requires building reliable infrastructure, securing application access, automating deployments, and maintaining system performance in production.

This project focuses on understanding the fundamentals of application deployment rather than relying entirely on managed platforms that hide the underlying infrastructure. By working through each stage manually before introducing automation, we gain practical knowledge of how servers, networking, security, and deployment pipelines work together.

The project follows a hands-on approach inspired by real-world production environments, using a full-stack application consisting of a Vue.js frontend, a FastAPI backend, and a self-hosted Meilisearch search engine.

The techniques demonstrated are applicable across different environments, including cloud virtual machines, AWS EC2 instances, bare-metal servers, and local hardware.

Throughout this project, we explore infrastructure provisioning, server hardening, reverse proxy configuration, continuous integration and delivery (CI/CD), automated security scanning, reliable deployments, and rollback strategies.

**The goal is to build a production-ready application environment that is secure, automated, maintainable, and reliable.**

## The roadmap
This book is structured systematically to guide you from absolute zero to an advanced automation posture.

- Module 1: The foundation — Cloud infrastructure provisioning, setting up locked rootless user models, and establishing strict SSH cryptographic authentication.
- Module 2: The application runtime — Manual stack deployment, memory swap optimization for resource-constrained systems, Gunicorn process engineering, and Supervisor monitoring.
- Module 3: Data & search — Self-hosting and isolating internal database and indexing layers securely behind standard server firewalls.
- Module 4: Global delivery & app security — DNS configuration, edge proxy routing, end-to-end SSL/TLS encryption, and robust Nginx web-layer hardening against soft 404 vulnerabilities and automated scanners.
- Module 5: The automation pipeline — Shifting security left by constructing a multi-stage GitHub Actions CI/CD framework featuring dependency auditing (SCA), source code scanning (SAST), integration testing, and atomic zero-downtime deployments.
- Module 6: Optimization & maintenance — Native log-based traffic analytics with GoAccess, real-time terminal resource observation, dynamic frontend vulnerability scanning (DAST), and automated rolling maintenance.

# Module 1: The foundation

## Cloud setup & access

### Introduction
- Before you deploy any code, you need a secure environment. We use DigitalOcean here, but these concepts apply to AWS EC2 , Linode, or even a Raspberry Pi.
- By the end of this section, you will have a server running Ubuntu 24.04 LTS, accessed securely via SSH keys, with password authentication completely disabled.

### Create the project

- We use DigitalOcean to host our web application.
- Create a project to hold your resources. Click on the New project button.
- The location of the + New Project button on the DigitalOcean dashboard.

<img width="1158" height="603" alt="1_1_1_create_new_project_inkscape" src="https://github.com/user-attachments/assets/a9d87cb7-e659-4d52-bfff-3ae6a7295f1b" />

- Give your project a name and a description. Choose a descriptive name so you can remember the purpose of the project later. I named mine “fullstackdeployment”. Click on Create Project when you are done.
- Give the project a name and a description.

<img width="1495" height="818" alt="1_1_2_information_about_project_step_1" src="https://github.com/user-attachments/assets/fea5b25b-db22-4997-8bbe-1475c4caaf05" />

- DigitalOcean will ask if you want to move resources to this project. Click Skip for now since you are starting from scratch.
- Skip this step because you don’t have any resources yet.

<img width="1082" height="591" alt="1_1_3_information_about_project_step_2" src="https://github.com/user-attachments/assets/06d5e8f3-861f-4bfb-8485-fd66b68784af" />

## Create the droplet

- Now, create the virtual machine. In DigitalOcean, these are called “droplets”. Click on Create in the top menu, then select droplets.
- A droplet is a Linux-based virtual machine that runs on virtualized hardware. This machine will host the code for your website.
- Click Create, then select droplets to start setting up your server.

<img width="1499" height="728" alt="1_1_4_create_droplet_inkscape" src="https://github.com/user-attachments/assets/fff04e19-0221-42f3-a0c8-537f50850a4a" />

- Choose a Region where your VM will be hosted. Always select a region geographically near you or your target users. I live in Morocco, so I chose New York datacenter.
- Select the region where your droplet will be hosted.

<img width="1448" height="610" alt="1_1_5_choose_region" src="https://github.com/user-attachments/assets/9a818d07-ccd4-48a9-9f2e-0e3fffb29119" />

- Next, choose an Operating System. Linux is the standard for servers. Select Ubuntu 24.04 (LTS) because it is stable and well-supported.
- Choose the operating system for your droplet.

<img width="1448" height="386" alt="1_1_6_choose_an_image" src="https://github.com/user-attachments/assets/6544364e-4a39-49dc-8a10-8afdee6c55a0" />

- Now you need to choose the size of your virtual machine. You can pick between shared CPU and dedicated CPU. Inside each option, you must decide how much RAM, disk space, and how many CPU cores you want.
- Dedicated VMs are more expensive because the resources are reserved only for you. Adding more RAM, disk space, or CPU cores will also increase the price.
- Fortunately, you can change the droplet size later if needed. For now, create a droplet with a shared CPU and the lowest resources. This costs $4 per month, and I will show you how to upgrade it later.
- Select the droplet size that fits your needs.

<img width="1448" height="775" alt="1_1_7_choose_size" src="https://github.com/user-attachments/assets/7a68825e-e5d6-45e2-b613-03d36fbb9249" />

- If you need more storage, click Add Volume and enter the size in GB. This feature is not free; it costs $1 per month per 10 GB. You can also enable automatic backups if you need them.

<img width="1431" height="388" alt="1_1_8_space_and_backup" src="https://github.com/user-attachments/assets/05667530-65ca-473c-b803-1f216b545cc9" />

## Configure authentication

- This is a critical security step. You can access your server using a password or an SSH key.
- Select SSH Key and click on the Add SSH Key button.

<img width="1431" height="344" alt="1_1_9_authentication" src="https://github.com/user-attachments/assets/958e4d21-430a-4b2b-a8fe-8163f712d5d3" />

- Always use an SSH key. Passwords are vulnerable to brute-force attacks, whereas SSH keys are significantly more secure. Think of it like a real lock and key:
  - The public key is the lock. You put this on the server (the droplet). Anyone can see the lock, but that does not help them open the door.
  - The private key is the physical key. You keep this on your computer. You never share it.
- When you try to connect, the server checks if your "key" fits its "lock." If it does, you get in without a password.

<img width="1804" height="1297" alt="1_1_10_how_ssh_keys_work" src="https://github.com/user-attachments/assets/7d7ad06f-f231-4067-9540-00ccb584b09f" />

The diagram above shows the conversation that happens between your computer and the server:

  1. SSH request: Your computer contacts the server and says, "I want to connect using this specific key ID."
  2. Challenge: The server is cautious. It sends back a random string of characters (called a nonce) and says, "Prove you own the private key by signing this random message."
  3. Send signature: Your computer takes that random message and "signs" it using your private key. This creates a unique digital signature that gets sent back to the server.
  4. The result: The server uses the public key (which you uploaded to it) to verify the signature. If the signature matches, the server knows you have the private key and grants access.

- To generate a key, open the terminal on your local computer. Use the Ed25519 algorithm, which is faster and more secure than the older RSA standard.
```bash
ssh-keygen -t ed25519 -f ~/.ssh/<your_key_name> -C "<key_comment>"
```
Here is what the arguments do:
- -t ed25519: Specifies the modern Ed25519 algorithm.
- -f ~/.ssh/<your_key_name>: Saves the key with a specific name so you don’t overwrite your default keys.
- -C "<key_comment>": Adds a comment to help you identify the key.

- The system will ask you to add a passphrase. This is an extra password to protect your private key. If someone steals your private key file, they cannot use it without this passphrase.
```bash
ssh-keygen -t ed25519 -f ~/.ssh/<your_key_name> -C "<key_comment>"

# Output:
Generating public/private ed25519 key pair.
Enter passphrase for "/home/<your_username>/.ssh/<your_key_name>" (empty for no passphrase):
```
- I strongly recommend adding a passphrase for your personal key.
- But if you need a key for a CI/CD pipeline later (to deploy code automatically), generate a separate key pair without a passphrase. Never remove the passphrase from your personal key.
- You have generated the key pair. Now, copy the public key.

```bash
cat ~/.ssh/<your_key_name>.pub
```

- The command displays your public key. Copy the entire text starting with ssh-ed25519. Paste it into the box, give it a name, and click Add SSH Key.

<img width="1131" height="605" alt="1_1_11_adding_a_public_key" src="https://github.com/user-attachments/assets/4b4d9bbe-228e-48be-a261-2ad3b40a1146" />

- You can add more than one SSH key by clicking the New SSH Key button. This allows you to add a specific key for your CI without a passphrase, while keeping the one on your computer protected with a passphrase.

<img width="1257" height="393" alt="1_1_12_add_additional_ssh_keys" src="https://github.com/user-attachments/assets/526e0bd4-7ade-4ca5-bf50-950010f48d28" />

- You are almost done. Select Add improved metrics monitoring and alerting because it is free.

<img width="1257" height="355" alt="1_1_13_advanced_options" src="https://github.com/user-attachments/assets/6eba9176-0cad-4d8a-a3a7-565c08e497da" />

## Finalize and connect

- Give your droplet a name you can recognize, add tags, and assign it to the project you created. You can deploy multiple droplets, but for now, keep the quantity set to 1 droplet.
- Give your droplet a name, tags, and assign it to a project.

<img width="1257" height="400" alt="1_1_14_final_step_before_creating_droplet" src="https://github.com/user-attachments/assets/5a1c75f4-bdf2-433d-b47c-db8469102126" />

- Click on Create droplet. The site redirects you to the project page. Under Resources, you should see a green dot next to your droplet. This means it is running.
- Ensure that the droplet is running and assigned to the project.

<img width="1257" height="599" alt="1_1_15_droplet_in_project_page" src="https://github.com/user-attachments/assets/cea09f96-95b3-4c63-a638-8bf6c112f295" />

- Later, if you find that this VM cannot handle the traffic, you can click on Upsize to add more resources.
- Increase resources by upsizing.

<img width="1221" height="1132" alt="1_1_16_upsize_droplet" src="https://github.com/user-attachments/assets/e5ac5ac2-0d7e-420e-beb8-7d33475731da" />

- You are now ready to connect to your VM. Find the IP address on the project page. Copy the IP address shown next to your droplet's name.

<img width="1257" height="599" alt="1_1_17_find_ip_address" src="https://github.com/user-attachments/assets/e78b9baa-05fe-4490-936b-431be75bf197" />

- Use the SSH command to connect to the machine. Pass your private key using the -i flag and add your IP address after root@.
```bash
ssh -i ~/.ssh/<your_key_name> root@<your_droplet_ip>
```
- A security warning appears about the authenticity of the host. Type yes and hit Enter to continue.
```bash
The authenticity of host '<your_droplet_ip> (<your_droplet_ip>)' can't be established.
ED25519 key fingerprint is SHA256:...
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

- Sometimes, the server is not fully ready even though the status says "Running". If you see a Connection closed message immediately after typing yes, don’t worry.
- If this happens, just wait a few seconds and run the SSH command again. It should work the second time.
```bash
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '<your_droplet_ip>' (ED69696) to the list of known hosts.
Connection closed by <your_droplet_ip> port 22
```

- Once connected, you will see a welcome message similar to this:
```bash
*** System restart required ***

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

root@<your_hostname>:~#
```
- Update your VM packages by running the following commands:
```bash
sudo apt update
sudo apt upgrade
```
- If you see a configuration screen during the update, select keep the local version currently installed. This option ensures you keep the working SSH configuration that DigitalOcean set up for you.

<img width="1821" height="993" alt="1_1_18_configure_open_ssh_server_warning (1)" src="https://github.com/user-attachments/assets/f09a35fa-bcac-43df-b18a-0944955a9ea8" />

- Reboot the machine to complete the upgrade and start using the new kernel and packages.
```bash
reboot
```
- The connection will close, but you can connect via SSH to the server again after it reboots. If you want to exit the VM, type exit and hit Enter or hit Ctrl+D.

## Create a non-root user and configure SSH access

- Running everything as root is dangerous. If you make a mistake, it might be unrecoverable. Also, if an attacker finds a vulnerability in your app while it is running as root, they gain full control of your server.
- Create a new user to act as a security barrier.
```bash
adduser <your_username>
```
- It will ask you to set a password and fill in some details. You can skip the details by pressing Enter.
```bash
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for <your_username>
Enter the new value, or press ENTER for the default
Full Name []: <your_fullname>
Room Number []:
Work Phone []:
Home Phone []:
Other []:
Is the information correct? [Y/n] y
info: Adding new user `<your_username>' to supplemental / extra groups `users' ...
info: Adding user `<your_username>' to group `users' ...
```

- Give the new user administrative privileges
```bash
usermod -aG sudo <your_username>
```
- Now, log out of the server (exit or Ctrl+D) and attempt to log in as your new user.
```bash
ssh -i ~/.ssh/<private_key_name> <your_username>@<your_droplet_ip>
```
- You will get an error. This happens because the SSH key you authorized exists only in the root user's authorized_keys file. The new user (<your_username>) has an empty list. You need to copy the key from root to <your_username>.
```bash
<your_username>@<your_droplet_ip>: Permission denied (publickey).
```
- Log back in as root.
```bash
ssh -i ~/.ssh/<private_key_name> root@<your_droplet_ip>
```
- Run these commands to copy the authorized keys to the new user. Copy the entire directory and preserve the strict root permissions using the --preserve=mode flag.
```bash
# Copy the .ssh directory, preserving the strict file permissions
cp -r --preserve=mode /root/.ssh /home/<your_username>/

# Give the new user ownership of this directory
chown -R <your_username>:<your_username> /home/<your_username>/.ssh
```
- Now, exit and try to log in as your new user again.
```bash
ssh -i ~/.ssh/<private_key_name> <your_username>@<your_droplet_ip>
```
- It should work now. You will see a prompt like this:
```bash
It should work now. You will see a prompt like this:
<your_username>@<your_hostname>:~$
```
- Before you move on to the next step, you must verify that your new user can actually perform administrative tasks. If you disable root login and find out your user cannot use sudo, you will be locked out of your own server.
```bash
sudo ls /root
```
- If the command works, you will see the contents of the root directory. If you get an error saying your user is "not in the sudoers file", STOP. Do not disable root login until you fix the user permissions.
- From now on, you will stop using root and use this account instead.

