---
lab:
    title: 'Lab: Deploying and configuring Windows Server on Azure VMs'
    type: 'Answer Key'
    module: 'Module 6: Deploying and Configuring Azure VMs'
---

# Lab answer key: Deploying and configuring Windows Server on Azure VMs

## Exercise 1: Authoring Azure Resource Manager (ARM) templates for Azure VM deployment

#### Task 1: Connect to your Azure subscription and enable enhanced security of Microsoft Defender for Cloud

In this task, you will connect to your Azure subscription and enable enhanced security features of Microsoft Defender for Cloud.

1. Connect to **SEA-ADM1**, and then, if needed, sign in as **CONTOSO\\Administrator** with a password of **Pa55w.rd**.
1. On **SEA-ADM1**, start Microsoft Edge, go to the [Azure portal](https://portal.azure.com), and sign in by using the credentials of a user account with the Owner role in the subscription you'll be using in this lab.

>**Note**: Skip the remaining steps in this task and proceed directly to the next one if you have already enabled Microsoft Defender for Cloud in your Azure subscription.

1. In the Azure portal, in the **Search resources, services, and docs** text box, on the toolbar, search for and select **Microsoft Defender for Cloud**.
1. On the **Microsoft Defender for Cloud \| Getting started** page, select **Upgrade**, and then select **Install agents**.

#### Task 2: Generate an ARM template and parameters files by using the Azure portal

In this task, you will use the Azure portal to create resource groups and create a disk in the resource group.

1. On **SEA-ADM1**, in the Azure portal, in the **Search resources, services, and docs** text box, on the toolbar, search for and select **Virtual machines**. In the **Virtual machines** page, select **+ Create**, and then select **Virtual machine**.
1. In the **Create a virtual machine** page, on the **Basics** tab, specify the following settings and leave all other settings with their default values, but do not deploy it:

   |Setting|Value|
   |---|---|
   |Subscription|the name of the Azure subscription you will be using in this lab.|
   |Resource group|the name of a new resource group **AZ800-L0601-RG**|
   |Virtual machine name|**az800l06-vm0**|
   |Region|Use the name of an Azure region in which you can provision Azure virtual machines|
   |Availability options|No infrastructure redundancy required|
   |Image|**Windows Server 2022 Datacenter: Azure Edition - Gen2**|
   |Azure Spot instance|No|
   |Size|**Standard_D2s_v3**|
   |Username|**Student**|
   |Password|**Pa55w.rd1234**|
   |Public inbound ports|None|
   |Would you like to use an existing Windows Server license|No|

1. Select **Next: Disks >**, and then on the **Create a virtual machine** page, on the **Disks** tab, specify the following settings, leaving all other settings with their default values:

   |Setting|Value|
   |---|---|
   |OS disk type|**Standard HDD**|

1. Select **Next: Networking >**, and in the **Create a virtual machine** page, on the **Networking** tab, select the **Create new** hyperlink that follows the **Virtual network** text box*.
1. On the **Create virtual network** page, specify the following settings, leaving all other settings with their default values, and then select **OK**:

   |Setting|Value|
   |---|---|
   |Name|**az800l06-vnet**|
   |Address range|**10.60.0.0/20**|
   |Subnet name|**subnet0**|
   |Subnet range|**10.60.0.0/24**|

1. Back on the **Create a virtual machine** page, on the **Networking** tab, specify the following settings, leaving all other settings with their default values:

   |Setting|Value|
   |---|---|
   |Public IP|None|
   |NIC network security group|None|
   |Accelerated networking|Off|
   |Place this virtual machine behind an existing load balancing solution?|No|

1. Select **Next: Management >**, and on the **Create a virtual machine** page, on the **Management** tab, specify the following settings, leaving all other settings with their default values:

   |Setting|Value|
   |---|---|
   |Boot diagnostics|**Enabled with managed storage account (recommended)**|
   
1. Select **Next: Advanced >**, on the **Advanced** tab of the **Create a virtual machine** page, review the available settings without modifying any of them, and then select **Review + Create**.

   >**Note**: Do not create the virtual machine. You will use for this purpose the autogenerated template.

#### Task 3: Download the ARM template and parameters files from the Azure portal

1. In the Azure portal, on the **Create a virtual machine** page, select **Download a template for automation**.
1. On the **Template** page, select **Download**.
1. Select the ellipsis button next to the **template.zip**, and then in the pop-up menu, select **Show in folder**. This will automatically open File Explorer displaying the content of the **Downloads** folder.
1. In File Explorer, copy **template.zip** to the **C:\\Labfiles\\Lab06** folder on **SEA-ADM1** (create a new folder if needed).
1. From the **Template** page, browse back to the **Create a virtual machine** page, and close it without completing the deployment.

## Exercise 2: Modifying ARM templates to include VM extension-based configuration

#### Task 1: Review the ARM template and parameters files for Azure VM deployment

1. On **SEA-ADM1**, start File Explorer, and then browse to the **C:\\Labfiles\\Lab06** folder.
1. Extract the content of the **template.zip** file into the same folder.
1. Open the **template.json** file in Notepad, and review its content. Keep the Notepad window open.
1. From File Explorer, open the **C:\\Labfiles\\Lab06\\parameters.json** file in Notepad and review its content.
1. Close the Notepad window displaying the **parameters.json** file.

#### Task 2: Add an Azure VM extension section to the existing template

1. On **SEA-ADM1**, in the Notepad window displaying the content of the **template.json** file, insert the following code directly after the `    "resources": [` line):

   ```json
        {
           "type": "Microsoft.Compute/virtualMachines/extensions",
           "name": "[concat(parameters('virtualMachineName'), '/customScriptExtension')]",
           "apiVersion": "2018-06-01",
           "location": "[resourceGroup().location]",
           "dependsOn": [
               "[concat('Microsoft.Compute/virtualMachines/', parameters('virtualMachineName'))]"
           ],
           "properties": {
               "publisher": "Microsoft.Compute",
               "type": "CustomScriptExtension",
               "typeHandlerVersion": "1.7",
               "autoUpgradeMinorVersion": true,
               "settings": {
                   "commandToExecute": "powershell.exe Install-WindowsFeature -name Web-Server -IncludeManagementTools && powershell.exe remove-item 'C:\\inetpub\\wwwroot\\iisstart.htm' && powershell.exe Add-Content -Path 'C:\\inetpub\\wwwroot\\iisstart.htm' -Value $('Hello World from ' + $env:computername)"
              }
           }
        },
   ```

1. Save the change and close the file.

## Exercise 3: Deploying Azure VMs running Windows Server by using ARM templates

#### Task 1: Deploy an Azure VM by using an ARM template

1. On **SEA-ADM1**, switch to the browser window displaying the Azure portal.
1. In the Azure portal, on the toolbar, in the **Search resources, services, and docs** text box, search for and select **Template deployment (deploy using custom templates)**.
1. In the **Custom deployment** page, select **Build your own template in the editor**.
1. On the **Edit template** page, select **Load file**, upload the template file **template.json** that you edited in the previous exercise, and then select **Save**.
1. On the **Custom deployment** page, select **Edit parameters**.
1. On the **Edit parameters** page, select **Load file**, upload the parameters file **parameters.json** that you reviewed in the previous exercise, and then select **Save**.
1. Back on the **Custom deployment** page, specify the following settings, and leave the other settings with their default values:

   |Setting|Value|
   |---|---|
   |Subscription|the name of the Azure subscription you are using in this lab|
   |Resource group|**AZ800-L0601-RG**|
   |Region|the name of the Azure region into which you can provision Azure VMs|
   |Admin Password|**Pa55w.rd1234**|

1. Select **Review + create**, and then select **Create**.

   >**Note**: The deployment might take about 10 minutes.

1. Verify that the deployment completed successfully.

#### Task 2: Review results of the Azure VM deployment

1. In the Azure portal, on the toolbar, in the **Search resources, services, and docs** text box, search for and select **Resource groups**.
1. On the **Resource groups** page, select the **AZ800-L0601-RG** entry.
1. On the **AZ800-L0601-RG** page, on the **Overview** page, review the list of resources, including the Azure VM **az800l06-vm0**.
1. Within the list of resources, select the Azure VM **az800l06-vm0** entry. 
1. On the **az800l06-vm0** page, select **Extensions**, and on the list of extensions, verify that the **customScriptExtension** has been provisioned successfully.
1. Browse back to the **AZ800-L0601-RG** page, and in the **Settings** section, select **Deployments**.
1. On the **AZ800-L0601-RG \| Deployments** page, select the **Microsoft.Template** link.
1. On the **Microsoft.Template \| Overview** page, select **Template**, and note that this is the same template you used for deployment.

## Exercise 4: Configuring administrative access to Azure VMs running Windows Server

#### Task 1: Verify the status of Azure Microsoft Defender for Cloud

1. In the Azure portal, on the toolbar, in the **Search resources, services, and docs** text box, search for and select **Microsoft Defender for Cloud**.
1. On the **Overview** page of Microsoft Defender for Cloud, on the vertical menu on the left side, in the **Management** section, select **Environment settings**. 
1. On the **Environment settings** page, select the entry representing your Azure subscription.
1. On the **Settings \| Defender plans** page, verify that the tile **Enable all Microsoft Defender for Cloud plans** is selected and, in the vertical menu on the left side, in the **Settings** section, select **Auto provisioning**.
1. On the **Settings \| Auto provisioning** page, in the list of extensions, to the right side of the **Log Analytics agent for Azure VMs** entry, select the **Edit configuration** link.
1. On the **Extension deployment configuration**, ensure that the **Connect Azure VMs to the default workspace(s) created by Defender for Cloud** entry is selected, select **Apply**, and back on the **Settings \| Auto provisioning** page, select **Save**.

#### Task 2: Review Just in time VM access settings

1. Browse back to the **Overview** page of Microsoft Defender for Cloud, and then, in the **Cloud Security** section, select **Workload protections**.
1. On the ***Microsoft Defender for Cloud \| Workload protections** page, select **Just-in-time VM access**.
1. On the **Just-in-time VM access** page, review the **Configured**, **Not Configured**, and **Unsupported** tabs.

   >**Note**: It might take up to 24 hours for the newly deployed VM to appear on the **Unsupported** tab. Rather than wait, continue to the next exercise.

## Exercise 5: Configuring Windows Server security in Azure VMs

#### Task 1: Create and configure an NSG

1. In the Azure portal, on the toolbar, in the **Search resources, services, and docs** text box, search for and select **Network security groups**.
1. On the **Network security groups** page, select **+ Create**.
1. On the **Basics** tab of the **Create network security group** page, specify the following settings (leave others with their default values):

   |Setting|Value|
   |---|---|
   |Subscription|the name of the Azure subscription you are using in this lab|
   |Resource group|**AZ800-L0601-RG**|
   |Name|**az800l06-vm0-nsg1**|
   |Region|the name of the Azure region into which you provisioned the Azure VM **az800l06-vm0**|

1. On the **Create network security group** page, on the **Basics** tab, select **Review + create**, and then select **Create**.
1. In the Azure portal, browse back to the **AZ800-L0601-RG** page, and then in the list of resources, select the entry representing the newly created network security group **az800l06-vm0-nsg1**.
1. On the **az800l06-vm0-nsg1** page, review the listing of the default inbound and outbound security rules, and then in the **Settings** section, select **Inbound security rules**.
1. On the **az800l06-vm0-nsg1 \| Inbound security rules** page, select **+ Add**.
1. On the **Add inbound security rule** page, specify the following settings, leaving all others with their default values, and then select **Add**:

   |Setting|Value|
   |---|---|
   |Source|**Any**|
   |Source port ranges|*|
   |Destination|**Any**|
   |Service|**HTTP**|
   |Action|**Allow**|
   |Priority|**300**|
   |Name|**AllowHTTPInBound**|

#### Task 2: Configure Inbound HTTP access to an Azure VM

1. In the Azure portal, browse back to the **AZ800-L0601-RG** page, and then in the list of resources, select the entry representing the Azure VM **az800l06-vm0**.
1. On the **az800l06-vm0** page, select **Networking**.
1. On the **az800l06-vm0 \| Networking** page, select the link designating the network interface attached to **az800l06-vm0**.
1. On the page displaying the network interface properties, in the vertical menu on the left side, in the **Settings** section, select **Network security group**. 
1. On the **Network security group** page, in the drop-down list, select **az800l06-vm0-nsg1**, and then select **Save**.
1. Back on the page displaying the properties of the network interface, select **IP configurations**, and then select the **ipconfig1** entry.
1. On the **ipconfig1** page, in the **Public IP address** section, select **Associate**, and then, below the **Public IP address** drop-down list, select **Create new**.
1. In the **Add a public IP address** window, specify the following settings, and then select **OK**:

   |Setting|Value|
   |---|---|
   |Name|**az800l06-vm0-pip1**|
   |SKU|**Standard**|

1. Back on the **ipconfig1** page, select **Save**.
1. Browse back to the page displaying the network interface properties and select **Overview**. Note the value of the public IP address assigned to the interface.
1. Open another browser tab, browse to that IP address, and verify that a webpage opens, displaying **Hello World from az800l06-vm0**.
1. From the lab computer, start the Remote Desktop app, and try connecting to the same IP address. Verify that the connection fails.

   >**Note**: This is expected because the Azure VM is currently not accessible from the Internet via TCP port 3389. It is accessible only via TCP port 80.

#### Task 3: Trigger re-evaluation of the JIT status of an Azure VM

>**Note**: This task is necessary to trigger re-evaluation of the JIT status of the Azure VM. By default, this might take up to 24 hours.

1. In the Azure portal, browse back to the **AZ800-L0601-RG** page, and then in the list of resources, select the entry representing the Azure VM **az800l06-vm0**.
1. On the **az800l06-vm0** page, select **Configuration**. 
1. On the **az800l06-vm0 \| Configuration** page, select **Enable JIT VM access** and select the **Open Azure Security Center** link.
1. On the **Just-in-time VM access** page, verify that the entry representing the **az800l06-vm0** Azure VM appears on the **Configured** tab.

#### Task 4: Connect to the Azure VM via JIT VM access

1. Browse back to the **az800l06-vm0** page, select **Connect**, and then in the drop-down list, select **RDP**. 
1. On the **az800l06-vm0 \| Connect** page, in the **Source IP** section, select **My IP**, and then select **Request access**.
1. Wait for the notification stating that your request has been approved, select **Download RDP File** and follow prompts to connect to the target Azure VM.
1. When prompted for credentials, specify the following values, and then select **OK**:

   |Setting|Value|
   |---|---|
   |Username|**Student**|
   |Password|**Pa55w.rd1234**|

1. Verify that you can successfully access via Remote Desktop the operating system running in the Azure VM and close the Remote Desktop session.

## Exercise 6: Deprovisioning the Azure environment

#### Task 1: Start a PowerShell session in Cloud Shell

1. On **SEA-ADM1**, in the Microsoft Edge window displaying the Azure portal, open the Azure Cloud Shell pane by selecting the Cloud Shell icon.
1. If prompted to select either **Bash** or **PowerShell**, select **PowerShell**.

   >**Note**: If this is the first time you're starting Cloud Shell and you're presented with the **You have no storage mounted** message, select the subscription you are using in this lab, and then select **Create storage**.

#### Task 2: Identify all Azure resources provisioned in the lab

1. From the Cloud Shell pane, run the following command to list all resource groups created throughout this lab:

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L06*'
   ```
1. Run the following command to delete all resource groups created throughout this lab:

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L06*' | Remove-AzResourceGroup -Force -AsJob
   ```
   >**Note**: The command executes asynchronously (as determined by the *-AsJob* parameter). So, while you will be able to run another PowerShell command immediately after within the same PowerShell session, it will take a few minutes before the resource groups are actually removed.