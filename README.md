<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Windows Server Logo"/>
</p>

<h1>Active Directory Lab (Part 2) - Domain Services & Client Integration 🏢🧑‍💼</h1>
This lab continues building an Active Directory environment in Azure. We'll promote a domain controller, create administrative users, join a client to the domain, and configure Remote Desktop for domain users.<br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (VMs, Networking)
- Remote Desktop (RDP)
- Windows Active Directory Domain Services
- PowerShell ISE

<h2>Operating Systems Used</h2>

- Windows Server 2022 (Domain Controller)
- Windows 10 Pro (Client)

<h2>Project Objectives</h2>

- Install and configure Active Directory Domain Services
- Promote DC to a domain controller with a new forest
- Create domain admin and employee users
- Join a Windows 10 client to the new domain
- Enable Remote Desktop for domain users
- Bulk create domain users via a PowerShell script

<h2>Configuration Steps</h2>

<p>
<img src="https://i.imgur.com/eHRLGSe.png" height="80%" width="80%" alt="Domain Setup"/>
</p>
<p>
<b>Part 1: Promote DC-1 and Join Client to the Domain</b><br />
<ul>
  <li>Turn on <b>DC-1</b> and <b>Client-1</b> in Azure Portal (if off)</li>
  <li>Login to DC-1 and install <b>Active Directory Domain Services</b></li>
  <li>Promote it to a Domain Controller with a new forest:
    <br /><code>mydomain.com</code> (use your own naming convention)</li>
  <li>Restart the VM and log back into DC-1 as: <code>mydomain.com\labuser</code></li>
  <li>Open <b>Active Directory Users and Computers (ADUC)</b></li>
  <li>Create the following Organizational Units (OUs): <b>_EMPLOYEES</b> and <b>_ADMINS</b></li>
  <li>Create a user: <code>jane_admin / Cyberlab123!</code> inside <b>_ADMINS</b></li>
  <li>Add <code>jane_admin</code> to the <b>Domain Admins</b> security group</li>
  <li>Log out of DC-1 and back in as <b>mydomain.com\jane_admin</b></li>
  <li>Use <b>jane_admin</b> as your admin account from now on</li>
</ul>
</p>
<br />

<p>
<img src="https://i.imgur.com/AivJWpu.png" height="80%" width="80%" alt="Client Join Domain"/>
</p>
<p>
<b>Join Client-1 to the Domain</b><br />
<ul>
  <li>Ensure Client-1’s DNS is set to DC-1’s private IP (already done)</li>
  <li>Restart Client-1 from the Azure Portal (already done)</li>
  <li>Login to Client-1 as local <code>labuser</code></li>
  <li>Join the machine to <code>mydomain.com</code> (computer will reboot)</li>
  <li>Login to DC-1 and verify Client-1 appears in ADUC</li>
  <li>Create a new OU: <b>_CLIENTS</b> and move Client-1 into it</li>
</ul>
</p>
<br />

<p>
<img src="https://i.imgur.com/OcClSys.png" height="80%" width="80%" alt="Remote Desktop"/>
</p>
<p>
<b>Part 2: Enable Remote Desktop Access for Domain Users</b><br />
<ul>
  <li>Login to Client-1 as <b>mydomain.com\jane_admin</b></li>
  <li>Open <b>System Properties</b> > Remote Desktop settings</li>
  <li>Allow <b>Domain Users</b> to have Remote Desktop access</li>
  <li>You should now be able to login to Client-1 using non-admin domain accounts</li>
  <li>(Normally this would be automated via Group Policy — perhaps in a future lab)</li>
</ul>
</p>
<br />

<p>
<img src="https://i.imgur.com/E3KbgER.png" height="80%" width="80%" alt="PowerShell Script"/>
</p>
<p>
<b>Create Multiple Users Using PowerShell</b><br />
<ul>
  <li>Login to DC-1 as <b>jane_admin</b></li>
  <li>Open <b>PowerShell ISE</b> as Administrator</li>
  <li>Paste and run your bulk user creation script to create 1,000 users</li>
  <pre>
<code class="language-powershell">
# ----- Edit these Variables for your own Use Case ----- #
$PASSWORD_FOR_USERS   = "Password1"
$NUMBER_OF_ACCOUNTS_TO_CREATE = 10000
# ------------------------------------------------------ #

Function generate-random-name() {
    $consonants = @('b','c','d','f','g','h','j','k','l','m','n','p','q','r','s','t','v','w','x','z')
    $vowels = @('a','e','i','o','u','y')
    $nameLength = Get-Random -Minimum 3 -Maximum 7
    $count = 0
    $name = ""

    while ($count -lt $nameLength) {
        if ($($count % 2) -eq 0) {
            $name += $consonants[$(Get-Random -Minimum 0 -Maximum $($consonants.Count - 1))]
        }
        else {
            $name += $vowels[$(Get-Random -Minimum 0 -Maximum $($vowels.Count - 1))]
        }
        $count++
    }

    return $name
}

$count = 1
while ($count -lt $NUMBER_OF_ACCOUNTS_TO_CREATE) {
    $firstName = generate-random-name
    $lastName = generate-random-name
    $username = $firstName + '.' + $lastName
    $password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force

    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    New-AdUser -AccountPassword $password `
               -GivenName $firstName `
               -Surname $lastName `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_EMPLOYEES,$(([ADSI]'').distinguishedName)" `
               -Enabled $true
    $count++
}
</code>
</pre>
  <li>Observe user accounts being created in <b>_EMPLOYEES</b> OU</li>
  <li>Try logging into Client-1 as one of the new users (check the password in the script)</li>
</ul>
</p>
<br />

<h2>💾 Saving Costs</h2>

- VMs are used in future labs — do not delete
- To avoid charges, stop the VMs in the Azure Portal when not in use
