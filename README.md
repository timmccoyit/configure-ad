






<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Step 1
- Step 2
- Step 3
- Step 4

<h2>Setup Virtual Environment</h2>

![image](https://github.com/user-attachments/assets/f477b1a5-052e-4f38-827a-d32d2e92bf41)
![image](https://github.com/user-attachments/assets/61b42279-c5f0-46cb-909d-a427cdd0323d)
![image](https://github.com/user-attachments/assets/4bf2fe1e-747e-435e-a799-fe06785ca261)

<p>
In this step I will be creating a resource group within Microsoft Azure, then I set up a Virtual network for them all to connect to, a Domain controller to run windows Server on, and A client PC for users to log into. Next, we're going into the domain controller's NIC settings to change the private IP Address to be Static instead of Dynamic meaning it will not change. Lastly, we will set the Client's DNS settings to the Domain Controller's private IP address. As a test you can also ping the private IP address of the Domain controller from your client PC to make sure they are properly connected. Running ipconfig /all should also show the Domain controller's private IP address in the DNS settings.
</p>
<br />

<h2>Deploying Active Directory<h2/>

![image](https://github.com/user-attachments/assets/8d1dc377-9a22-4683-a41d-e28609f70436)
![image](https://github.com/user-attachments/assets/258bc0b9-bf5d-49db-ba43-e81c2df54521)


<p>
First we'll be installing the Active Directory itself so we can start creating the basic infrastructure of it. To do this we go into the server manager and click Add roles and features > go through setup wizard until you reach server roles and check Active Directory Domain Services. After this process is finished you can close the setup wizard and get ready to promote DC to a Domain controller and setup a new forest as mydomain.com 
</p>
<br />

![image](https://github.com/user-attachments/assets/5bc238a8-d9a8-4677-96e8-5611a6b20204)
![image](https://github.com/user-attachments/assets/6e0e5398-28d2-400d-bd47-ee938e8154d0)
![image](https://github.com/user-attachments/assets/e0b8e4d8-4e65-416b-a834-272b206d3883)


<p>
Now we are creating a Domain Admin user within our domain. We will do this by accessing the Active Directory Users and Computers (ADUC) environment. I create an Organizational Unit(OU) within this for employees then another one for admins. We then create a new employee that will be our domain admin and her name is Jane Doe. I add this user to the "Domain Admins" Security Group as well then log out and log back into this new admin user we've created. 
</p>
<br />

<h2>Join Client to domain<h2/>

![image](https://github.com/user-attachments/assets/d62aff2c-97ab-4843-984e-0669201b2bd3)
![image](https://github.com/user-attachments/assets/eb237abb-2f8e-417f-a22b-dc54a4dd234e)


<p>
First we'll login to the client vm as the original local admin and join it to the domain which will also restart the computer. Next, we log into the domain controller and verify the client vm shows up in ADUC. Lastly, I create a new OU and name it "_CLIENTS" and drag client-1 into it. To start this process we go to Client-1's vm and open system > rename this PC(advanced) > Computer name tab > Change > Domain (mydomain.com) > use your new admin user (jane_admin) login to finalize this change. After this you can see client-1's computer inside the computers section in Active Directory now, which we will join to the clients OU we create.
</p>
<br />

<h2>Setup Remote Desktop for non admin users on Client-1<h2/>

![image](https://github.com/user-attachments/assets/ec752181-cbc9-41bc-ba0a-df7785ac3ea8)
![image](https://github.com/user-attachments/assets/a274b4c1-6ff1-4b5a-9ddc-167483c8131e)
![image](https://github.com/user-attachments/assets/b0d91db3-17ff-4cbd-b3b4-d137bcdc862b)
![image](https://github.com/user-attachments/assets/79e6c8c6-debc-4ea0-b7fd-521c8c7b6372)

<p>
First off we will be logging into client vm as jane_admin and opening system properties > Remote Desktop > allow domain users access to remote desktop and now I can log into client vm as a normal non admin user. Keep in mind that this would normally be done with Group policy to affect a many systems at once but for the purpose of simple easily digestable demonstration im doing it this way. With this step being done this will allow ALL domain users to be able to login to client-1. For the final part of this section we'll log back in to DC-1 as jane_admin and open powershell_Ise as an admin. We're going to be creating a new file and pasting a script created to randomly generate a specific amount of users(100 in our case) to be sent into the employees OU. This will allow us to pick any user that was generated and login to client-1 vm with the acocunt. Observe the users that were added and log in to who ever you want!  
</p>
<br />

<h2>Dealing with account lockouts</h2>

![image](https://github.com/user-attachments/assets/b3465b78-a95a-4261-a737-809145f8a206)
![image](https://github.com/user-attachments/assets/095c1469-5a46-4373-8a3a-f6d0fe49c1d6)
![image](https://github.com/user-attachments/assets/69836024-9636-4444-843d-3fd50fa99e95)
![image](https://github.com/user-attachments/assets/51a26439-a526-4825-b1ed-77e2cc8b0ef0)
![image](https://github.com/user-attachments/assets/91cdd5db-ae2c-4380-838a-3cfce471cb83)
![image](https://github.com/user-attachments/assets/5f4923fd-4405-4bd3-9e2f-90783f55f4e2)
![image](https://github.com/user-attachments/assets/f60a7efd-2b39-41f8-900b-034d5238de7e)


<p>
The first we must complete for the account lockouts to function is open the Group Policy Management Console(GPMC) to create a policy that locks users out of their accounts after a certain amount of failed login attempts. Open the Group policy managment editor and go to computer config > policies > windows settings > security settings > account lockout policy then configure account lockout settings. It will recommend you settings depending on the lockout duration you pick but keep in mind you can still change it. Once this is changed, you COULD wait until the policy changes on its own but we're going to login to jane admin and run a command that forces a group policy update to save time(gpupdate /force). Running gpresult /r as an admin will show those group policy update results and when the group policy was last updated. After trying to log into the user we last used and failing the password attempts you should be locked out of your account. Active directory should show that this user has been locked out of their account, simply check the unlock account field and you should be able to login as that user again with the correct password. To reset password simple right click that user and go to reset password and choose which options apply for the situation. 
</p>
<br />

<h2>Enabling and Disabling Accounts</h2>

![image](https://github.com/user-attachments/assets/e8828a35-b5fd-4e78-9b6e-007ce8068f3e)
![image](https://github.com/user-attachments/assets/2e06197e-2a0a-48a5-93a6-086330107c1a)


<p>
To lock and unlock an account you use the same process as resetting, HOWEVER keep in mind this user may have been locked out for a legitimate reason and for this reason make sure you are checking with your leadership/management to verify if they are allowed to have their account unlocked or not.
</p>
<br />

<h2>Log Observation</h2>

![image](https://github.com/user-attachments/assets/51412f4d-e551-4fd6-a0fd-4b8e1b820f6a)

<p>
To open the logs we'll go back to dc-1 vm and simply type event viewer into the start window and open it. From here we just go to Windows Logs > Security then from here you can view the logs and even press control+f to find the specific user we used and look at any logs showing events that pertain to that specific user.
</p>
<br />
