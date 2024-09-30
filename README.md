# assignment-1


## 1. Introduction to SSH and doctl

### SSH Keys
SSH (Secure Shell) keys provide a secure way to connect to remote servers without using passwords. The `ed25519` key type is preferred for its strong security and smaller key size.

### doctl Command-Line Tool
`doctl` is DigitalOcean’s command-line interface, allowing you to manage droplets, images, and other resources directly from the terminal.

---

## Making a droplet using cloud-init

SSH keys allow for secure, password-less authentication. To generate SSH keys, follow these steps:

1. Open your terminal and run:
   ```bash
   ssh-keygen -t ed25519 -f ~/.ssh/do-key2 -C "your_email@example.com"
   ```
   - `-t ed25519`: Specifies the key type (modern, strong).
   - `-f` directs the location where you want to place your key.
   - `-C "your_email@example.com"`: Adds a comment for identification.

2. When prompted for a passphrase, press `Enter` to skip (or add a passphrase for extra security).

3. Adding the SSH key to your DigitalOcean account:
   - Copy your public key by running the command:
     ```bash
     pbcopy < ~/.ssh/do-key2.pub
     ```
   - Log in to your DigitalOcean account, go to **Settings** in the left panel, open **Security**, and press **Add SSH Key**.
   - Enter the key you copied and give a name to your key. Now your key is registered on your DigitalOcean account.

---

### Step 3: Add a Custom Arch Linux Image

1. In the DigitalOcean Dashboard, go to **Images** and click **Custom Images**.
2. Download the appropriate Arch Linux image. You want the most recent "cloudimg" file with the `.qcow2` extension. The date may differ.
3. Set your region to `sfo3` and specify the size of the droplet where this image will be used. Once the upload is complete, you can use this image for your Droplet.

---

### Step 4: Create a Cloud-init Configuration File

1. Go into the `.ssh` directory and create a folder named `assignment` using the `mkdir` command:
   ```bash
   mkdir assignment
   ```
2. Go into the `assignment` directory using:
   ```bash
   cd assignment
   ```
   Create a new cloud-init file:
   ```bash
   touch cloud-config.yml
   ```
3. Open the cloud-init file using:
   ```bash
   open cloud-config.yml
   ```
4. Now enter the following text in the `cloud-config.yml` file:
   ```yaml
   #cloud-config
   users:
     - name: user-name #change me
       primary_group: group-name #change me
       groups: wheel
       shell: /bin/bash
       sudo: ['ALL=(ALL) NOPASSWD:ALL']
       ssh-authorized-keys:
         - ssh-ed25519 ...
   
   packages:
     - ripgrep
     - rsync
     - neovim
     - fd
     - less
     - man-db
     - bash-completion
     - tmux
   
   disable_root: true
   ```
5. Replace `user-name` with your desired username.
6. Replace `ssh-ed25519 AAAAC3Nza...` with your public SSH key. Use the command:
   ```bash
   pbcopy < ~/.ssh/do-key2.pub
   ```

---

### Step 5: Create an Arch Linux Droplet

1. Go to the **Droplets** section in the DigitalOcean web console.
2. Click **Create Droplet**.
3. Select your custom Arch Linux image from the **Images** section.
4. Choose a plan (e.g., 1 vCPU with 1GB RAM plan).
5. Under **Authentication**, select the SSH key you added earlier.
6. Set your region (the same region where the custom image was uploaded).
7. Under **Advanced Options**, select **User Data** and paste the contents of your cloud-init file.

8. Click **Create Droplet**. After the droplet is created, connect using the following command:
   ```bash
   ssh newuser@your_droplet_ip_address
   ```
   - Replace `newuser` with the username you defined in your `cloud-config.yml` file.
   - Replace `your_droplet_ip_address` with the actual IP address of your Droplet.

To create a shortcut, you can create a config file in the same folder and add the following text:
   ```bash
   Host arch
     HostName 147.182.247.243
     User arch
     PreferredAuthentications publickey
     IdentityFile ~/.ssh/do-key
     StrictHostKeyChecking no
     UserKnownHostsFile /dev/null
   ```
In the hostname, put the IP address of the droplet you created.

---

### Creating a droplet using the existing droplet

Login to your droplet using the `ssh arch` command.

---

## Creating SSH Keys with ed25519

To generate `ed25519` SSH keys on macOS, open the Terminal and run:
   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```
You will be prompted to save the key in a location (by default `~/.ssh/id_ed25519`). Press `Enter` to accept the default.

Next, view your public key using:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
Copy the output.

---

### Adding the SSH Key in your DigitalOcean account

1. Log in to your DigitalOcean account, go to **Settings** in the left panel, open **Security**, and press **Add SSH Key**.
2. Enter the key you copied and give a name to your key. Now your key is registered.

---

### Install doctl Using pacman

Use the command:
   ```bash
   sudo pacman -S doctl
   ```

---

### Authenticate doctl with DigitalOcean

1. Obtain a Personal Access Token from the DigitalOcean API Tokens page by clicking **Generate New Token**.
2. Open your terminal and run:
   ```bash
   doctl auth init
   ```
3. Paste your generated token when prompted.
4. To verify the account, use the command:
   ```bash
   doctl account get
   ```

5. Now get the SSH key ID using the command:
   ```bash
   doctl compute ssh-key list
   ```

---

## Creating an Arch Linux Droplet Using doctl

### Step 1: Find the Arch Linux Image

List available images with this command:
   ```bash
   doctl compute image list
   ```
Find the Arch Linux image ID from the list.

---

### Step 2: Create the Droplet

Run the following command to create your droplet:
   ```bash
   doctl compute droplet create arch-droplet \
     --region nyc3 \
     --image IMAGE_ID \
     --size s-1vcpu-1gb \
     --ssh-keys YOUR_SSH_KEY_ID \
     --user-data-file cloud-config.yml \
     --wait
   ```
- Replace `IMAGE_ID` with the Arch Linux image ID you found.
- Replace `YOUR_SSH_KEY_ID` with the ID of your `ed25519` SSH key (list SSH keys with `doctl compute ssh-key list`).

---

### Connecting to Your Droplet via SSH

After creating the droplet, connect to it using SSH:
   ```bash
   ssh -i ~/.ssh/id_ed25519 newuser@your_droplet_ip_address
   ```
- Replace `newuser` with the username you set in the `cloud-config.yml` file.
- Replace `your_droplet_ip_address` with the public IP of your droplet.
