<h1>Active Directory Home Lab</h1>

<h2>Description</h2>
Installation of Zentyal on Proxmox.
<br />

<h2>Requirements</h2>

- <b>Proxmox VE installed</b> 
- <b>Zentyal DE 8.0 ISO file (https://www.zentyal.com/community)</b>

<h2>Walk-through:</h2>

### **1. Upload Zentyal ISO to Proxmox**
1. Open **Proxmox Web UI** (usually `https://your-proxmox-ip:8006`).
2. In the **left panel**, select your Proxmox node.
3. Go to **Datacenter > Storage** and select the local storage (`local-lvm`).
4. Click on the **“ISO Images”** tab.
5. Click **“Upload”** and select the Zentyal ISO file.
6. Wait for the upload to finish.
---
### **2. Create a New VM for Zentyal**

1. Click **"Create VM"** in the top-right corner.
2. Fill in the following:
   - **Node**: Leave as default
   - **VM ID**: Auto-assigned
   - **Name**: `zentyal-server`
3. Click **Next**.
---
### **3. Select the ISO Image**
1. In the **OS tab**, set:
   - **ISO Image**: Select the uploaded Zentyal ISO.
   - **Guest OS Type**: Linux
   - **Version**: Ubuntu
2. Click **Next**.
---
### **4. Configure System Settings**
- **Graphics Card**: Default
- **BIOS**: OVMF (UEFI)
- **Machine**: Default 
Click **Next**.
---
### **5. Set Hard Disk Parameters**
- **Bus/Device**: VirtIO
- **Disk Size**: 20GB
- **Storage**: Choose local or another storage pool.
Click **Next**.
---
### **6. Configure CPU and Memory**

- **Cores**: 2
- **Sockets**: 1
- **Memory**: 4GB
Click **Next**.
---
### **7. Configure Network**
- **Bridge**: Choose main Proxmox bridge (`vmbr0`)
- **Model**: VirtIO
Click **Next** and then **Finish**.
---
### **8. Start the VM and Install Zentyal**
1. Select the newly created VM in the left panel.
2. Click **“Start”**.
3. Open the **Console** tab.
4. The Zentyal installer should start. Follow on-screen instructions:
   - Select language, location, and keyboard.
   - Partition disk.
   - Create admin user and hostname.
   - Wait for installation to complete.
---
### **9. Initial Setup After Installation**

1. Once Zentyal reboots, log in via console.
2. Open a browser and access the **Zentyal web admin interface**:
   ```
   https://<zentyal-vm-ip>:8443
   ```
3. Login with admin credentials.
---
### Add Proxmox Tools to the VM
Install `qemu-guest-agent`
```bash
sudo apt update
sudo apt install qemu-guest-agent
sudo systemctl start qemu-guest-agent
sudo systemctl enable qemu-guest-agent
```
Then go to Proxmox > **Options**, and make sure "QEMU Guest Agent" is set to **Yes**.

---
### Duplicate entire above process to create secondary server
---
## Set Up DNS and Active Directory on Zentyal

### **1. Log in to the Zentyal Web Interface**
Open your browser and go to:
```
https://<zentyal-vm-ip>:8443
```
Login with the admin credentials you created during install.

---
### **2. Install AD & DNS Modules**

1. In the **Dashboard**, go to **Software Management > Zentyal Components**.
2. Select and install:
   -  **Domain Controller and File Sharing**
   -  **DNS Service**
3. Click **Save Changes** and allow the system to install and apply the configuration.
---
### **3. Configure the Domain Controller (AD)**

1. Go to **Module Status**, make sure **Domain Controller and File Sharing** is **enabled**.
2. Go to **Users and Computers > Manage**.
3. You'll be prompted to configure your domain. Choose:

   - **Create a new domain** (unless you're joining an existing AD)
   - Domain name: e.g., `corp.local` or `mydomain.lan`
   - NetBIOS name: e.g., `CORP`
   - Administrator password (for domain admin)

4. Click **OK** and let Zentyal configure Samba/AD. This may take a few minutes.
---
### **4. Verify DNS Settings**
Zentyal automatically configures DNS to support the AD domain.
To double-check:
1. Go to **DNS > General Settings**:
   - Ensure the **domain name** (e.g., `corp.local`) is listed.
   - DNS server should be set to **listen on all interfaces**, or at least the LAN interface.

2. Under **DNS > Domains**, make sure:
   - Your domain (e.g., `corp.local`) exists.
   - Necessary **A** records, **NS** records, and **SRV** records are present (these are auto-generated).
---
### **5. Create Users and Groups**

Go to **Users and Computers > Manage**, and:

- Create **new users** under the domain
- Create **groups** as needed (e.g., `Admins`, `IT`, `HR`)
---
### **6. Join a Windows Client to the Domain**
1. On a Windows PC:
   - Open **System Properties** > **Computer Name** > **Change settings**
   - Click **Domain**, enter the domain name (e.g., `corp.local`)
2. When prompted, enter the **Zentyal domain admin credentials**.
3. After a successful join, **reboot** the client.
Set the **Windows PC's DNS server** to the **Zentyal server IP** before trying to join the domain. Otherwise, it won’t resolve domain names correctly.
---
## Optional Next Steps
- Enable **Roaming Profiles** or **Folder Redirection**
- Set up **Group Policies** using RSAT on Windows
- Add **Shared Folders** under **File Sharing**
- Configure **Backups** of AD data
---

<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
