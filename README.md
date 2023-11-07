# configure-ad
<p align="center">
<img src="https://i.imgur.com/ntqcBtC.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1 align = "center">Installing and Configuring Active Directory in Microsoft Azure</h1>
This lab shows how to install and configure Active Directory using Microsoft Azure. We will create two VMs in Azure in the same virtual network. One VM will be the <b>Domain Controller</b> and have Active Directory installed, and other VM will be used as a <b>client</b>. We will then configure the Active Directory to permit the Client to join the domain as well as creating user accounts using a Powershell script. 

<br />

<h2>Environments and Technologies Used</h2>
<ul>
  <li>Microsoft Azure (Virtual Machines/Compute)</li>
  <li>Remote Desktop</li>
  <li>Active Directory Domain Services</li>
  <li>Powershell</li>
  <li>(OPTIONAL) Notepad for writing down usernames and passwords for VMs</li>
</ul>

<br />

<h2>Operating Systems Used</h2>
<ul>
  <li>Windows Server 2022</li>
  <li>Windows 10 Pro (21H2)</li>
</ul>

<br />

<h2>Installation Steps</h2>

<h3>Setting up Resources in Azure</h3>

<p>
  <ul>
    <li>Our Client VM should be installed normally using the Windows 10 image (OS)</li>
    <ul>
      <li>Tutorial on how to install VMs and to access them using Remote Desktop can be found <b><a href ="https://github.com/ColtonTrauCC/vm-network">here</a></b></li>
    </ul>      
    <li>Our Domain Controller VM using Active Directory should be created using the image <b>Windows Server 2022 Datacenter: Azure Edition - x64 Gen2</b></li>
    <ul>
      <li><img src = "https://github.com/ColtonTrauCC/active-directory/assets/147654000/2ba43657-9799-46d3-9140-c9952282614e" height="80%" width="80%"/></li>
    </ul>
    <li>After the VMs are created, we'll set the Domain Controller's IP Address as <i>static</i> since having it dynamic will make them difficult for the VM to communicate with our client VM.</li>
    <li>Go to your Virtual Machines in Azure and go to <b>Networking</b> then go to the link listed next to <b>Network Interface</b>. Head to <b>IP Configurations</b> under <b>settings</b>, go to the ipconfig link to open up a window to toggle the IP configuration and allocation to <b>Static</b>.</li>
    <ul>
      <li>IP Configuration for the Domain VM</li>
      <li><img src="https://github.com/ColtonTrauCC/active-directory/assets/147654000/f9883539-f1b2-4ada-80d9-2e0df6c2de6d" height="80%" width="80%"/></li>
    </ul>
  </ul>
</p>

<br />

<h3>Ensuring Connectivity</h3>

<p>
  <ul>
    <li>Logging in to the Client VM, open the Command Prompt and enter the command <b>ping [Domain Controller Private IP Address] -t</b> to endlessly send ping in order ensure reachability with the Domain Controller. Connection should time out after the first ping due to the Domain Controller's Firewall Settings.</li>
    <ul>
      <li><img src = "https://github.com/ColtonTrauCC/active-directory/assets/147654000/be2faa69-d835-4222-a105-b6f5201c018f" height = 80% width = 80% /></li>
    </ul>
    <li>Logging into the Domain Controller VM, go to the <b>Windows Defender Firewall with Advanced Security</b>. Head to the <b>Inbound Rules</b> and enable the rules under the protocol <b>ICMPv4</b>, specifically <i>Core Networking Diagnostics - ICMP Echo Request (ICMPv4-In)</i></li>
    <ul>
	<li><img src = "https://github.com/ColtonTrauCC/active-directory/assets/147654000/63bef2fe-62b6-4230-aac2-d6528f038bc5" height = 80% width = 80% /></li>
    </ul>
    <li>Head back to the Client VM, and we should now be seeing replies</li>
    <ul>
	<li><img src = "https://github.com/ColtonTrauCC/active-directory/assets/147654000/0918e56e-4fe8-497e-aa05-e9e4b3139042" height = 80% width = 80% /></li>
    </ul>
  </ul>
</p>

<br />

<h3>Installing Active Directory on the Domain Controller</h3>

<p>
  <ul>
	  <li>In your Domain Controller VM, go to the Server Manager Dashboard and click on <b>Add Roles and Features</b>. Go through the installation process and upon getting to <b>Server Roles</b>, make sure to check the box for <b>Active Directory Domain Services</b></li>
	  <ul>
	    <li><img src = "https://github.com/ColtonTrauCC/active-directory/assets/147654000/0b835d58-c357-40ce-a759-a0770c7929c9" height = 80% width = 80% /></li>
	  </ul>
	 <li>Once installed, we now have to promote the server into a domain controller. To do so, you may notice a <b>warning notification</b> on the top right where the flag icon is. Click on that flag and click <b>Promote this server to a domain controller</b>. Click on Add a new forest and specify a domain name. For this tutorial, we'll name the domain <b>mydomain.com</b>, specifiy the password, and proceed with the install. Noted, you will be automatically signed out, re-log in through Remote Desktop, and installation is fully completed!</li>
	  <ul>
	    <li><img src = "https://github.com/ColtonTrauCC/active-directory/assets/147654000/af0fb9e9-2f55-4fb0-a372-487d2dab317e" height = 80% width = 80% /></li>
	    <li><img src = "https://github.com/ColtonTrauCC/active-directory/assets/147654000/fb6b02e1-2e22-4147-aede-1d126c634ec8" height = 80% width = 80% /></li>
    	 </ul>
  </ul>
</p>

<br />

<h3>Important Log In Note</h3>

<p>
  <ul>
    <li>When logging back in to the domain controller VM through Remote Desktop Connection, it is important to log in with the <b>context of the domain.</b></li>
    <li>Type out the domain path and then the name of the user. For example: <b>mydomain.com/labuser.</b></li>  
  </ul>
</p>

<br />

<h2>Configuration Steps</h2>

<h3>Creating Organizational Units (OUs) and Users</h3>

<p>
  <ul>
    <li>OUs act like folders that hold information, privileges, and login access of users in the directory</li>
    <li>In the Server Manager Dashboard, go to the <b>Tools</b> tab to open the Active Directory Users and Computers console, right click on the domain (mydomain.com) and make two OUs, <b>_ADMIN</b> and <b>_EMPLOYEES</b>.</li>
	  <ul>
		  <li>These OUs names are needed for a later step were we create multiple accounts</li>
	  </ul>
    <li>In the _ADMIN OU, we'll create the user <b>Jane Doe</b> with the user name <b>jane_admin</b> and password of your creation</li>
    <ul>
	<li><img src = "https://github.com/ColtonTrauCC/active-directory/assets/147654000/8ab7e7b5-b5c4-4da6-b748-03d452778879" height = 80% width = 80% /></li>
    </ul>
    <li>We'll be granting Jane admin privileges. Using the <b>Security Group</b>, right click on the user and open their <b>Properties</b>b>. Click Member Of then Add to apply the appropraite security group.</li>
    <ul>
	    <li><img src = "https://github.com/ColtonTrauCC/active-directory/assets/147654000/e926f68b-be89-40f5-a294-b479602f9869" height = 80% width = 80% /></li>
    </ul>
    <li>Now, the user Jane will be used to log in from here on, using the login username jane_admin.</li>
  </ul>
</p>

<br />

<h3>Joining the Client to the Domain</h3>

<p>
  <ul>
    <li>First, we need to configure the Domain Name System (DNS) server. Go to your Domain Controller VM in the Azure Portal and go to <b>Networking</b> then go to the link listed next to <b>Network Interface</b>. Head to <b>DNS Servers</b> under <b>settings</b>, and set the DNS Server to <b>Custom</b>. Then, enter the domain controller's private IP address and save the changes. Restart the client VM in order to ensure the DNS changes are saved.</li>
    <ul>
	    <li><img src = "https://github.com/ColtonTrauCC/active-directory/assets/147654000/09ba39f8-0e5c-4d64-b276-ab10af8a0efd" height = 80% width = 80% /></li>
    </ul>
    <li>In the System menu of the client VM, click on Rename this PC (advanced) and Change.</li>
    <li>Enter the domain and necessary credentials in order to let the client join the domain (logging in as jane_admin). It is important to note that the login credentials have to be input within the context of the domain path (mydomain.com\jane_admin).</li>
    <ul>
	    <li><img src = "https://github.com/ColtonTrauCC/active-directory/assets/147654000/49ec7774-3dd4-4c62-81fe-91da91d80d08" height = 80% width = 80% /></li>
    </ul>
    <li>The client should now be part of the domain (A popup should appear welcoming you to the domain). On the domain controller, the client should now appear in Computers in the Active Directory Users and Computers panel.</li>
    <ul>
	    <li><img src = "https://github.com/ColtonTrauCC/active-directory/assets/147654000/a8ef09bf-36e2-4aab-8c44-d99d7b3fcb7a" height = 80% width = 80% /></li>
    </ul>  
  </ul>
</p>

<br />

<h3>Setup Remote Desktop for non-administrative users on Client VM</h3>

<p>
  <ul>
    <li>Before users in the domain can use the client computer, Remote Desktop has to be enabled for non-administrative users.</li>
    <li>While logged in as the administrator (jane_admin), open <b>System Properties</b>. Click on <b>Remote Desktop</b> and Select users that can remotely access this PC.</li>  
    <li>Allow Domain Users access to Remote Desktop. Non-administrative users can now log in to the Client.</li>
    <ul>
	    <li><img src = "https://github.com/ColtonTrauCC/active-directory/assets/147654000/96e3a27f-83fd-40c3-86bc-f3532c24bf6b" height = 80% width = 80% /></li>
    </ul>
  </ul>
</p>

<br />

<h3>Creating Users and attempt to log into the Client VM with one of the users</h3>

<p>
  <ul>
    <li>In the Domain Controller VM logged in as jane_admin, open <b>Powershell ISE</b> as an administrator</li>
    <li>Using <a href = "https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1">this powershell script</a>, we will create thousands of randomly generated accounts all with the password "Password1"</li>
    <li>In Powershell ISE, create a new file and copy-and-paste the powershell script into the file and then run the script</li>
    <ul>
	    <li>All these users are generated and put into the _EMPLOYEES Organizational Unit in the Active Directory</li>
	    <li><img src = "https://github.com/ColtonTrauCC/active-directory/assets/147654000/0033e7ae-6446-46d1-ae13-3873d475e8ac" height = 80% width = 80% /></li>
    </ul>
    <li>Head to the Active Directory Users and Computers console and select a random username and obtain their login information by going to <b>Properties</b> and in the <b>Account</b> tab</li>
    <ul>
	    <li>The username generate should appear as <b>[first name].[last name]</b>, in this image the user is selecting "falojo.kugori"</li>
	    <li><img src = "https://github.com/ColtonTrauCC/active-directory/assets/147654000/569cac00-65bf-4315-be16-6f502fb44b49" height = 80% width = 80% /></li>
    </ul>
    <li>Attempt to log in the Client VM using the generate username you have selected (username being <b>mydomain.com\username</b>) and the password "Password1"</li>
  </ul>
</p>

<br />
