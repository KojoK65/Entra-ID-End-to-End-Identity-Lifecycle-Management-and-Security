# 🔐 End-to-End Identity Lifecycle Automation and Security  

<p align="center">  
This lab demonstrates how I automated <strong>user onboarding, offboarding, access reviews, and MFA enforcement</strong> in Azure Active Directory using <strong>PowerShell + Microsoft Graph</strong>.  
</p>  
<p align="center">
<img width="675" height="375" alt="image" src="https://github.com/user-attachments/assets/99011947-86c4-4066-9603-9fb9c75af7b2" />
</p>

## 1️⃣ Azure AD Overview 
🖼️ *Azure Active Directory Overview Page — showing tenant name + AAD blade open*  
<p align="center">
  <img width="675" height="375" alt="Screenshot (45)" src="https://github.com/user-attachments/assets/f977f72c-1f96-4ba2-a7cc-2f9a374b308c" />
</p>

## 2️⃣ Install Required PowerShell Modules  

✅ I installed the required PowerShell modules (<code>AzureAD</code> and <code>Microsoft.Graph</code>) to interact with Azure AD.  

<p align="center">
  <img width="675" height="375" alt="Screenshot (46)" src="https://github.com/user-attachments/assets/a4615676-678b-44cd-915f-83e295401f89" />
</p>

## 3️⃣ Register Azure AD App for Automation  

<p align="center">  
✅ I registered an app called <strong>IdentityFlow</strong> in Azure AD, which I used for scripting.  
I granted it permissions to manage users and groups so my automation scripts could run securely.  
</p>

<p align="center"><em>App Registration Overview (IdentityFlow + Client ID)</em></p>
<p align="center">
  <img width="675" height="375" alt="Screenshot (47)" src="https://github.com/user-attachments/assets/b0824adb-beb3-42a0-9948-f940162f0c90" />
</p>  

<p align="center"><em>API Permissions with admin consent</em></p>
<p align="center">
  <img width="675" height="375" alt="Screenshot (48)" src="https://github.com/user-attachments/assets/938a077f-61b7-4ebc-9172-80b5dae96a6c" />
</p>  

## 4️⃣ Create Excel File with User Data  

<p align="center">  
✅ I built an <strong>Excel file (<code>users.csv</code>)</strong> that contained user details such as UPN, display name, department, job title, manager, and status.  
This served as the input for both onboarding and offboarding.  
</p>

<p align="center">  
  <img width="675" height="375" alt="Screen Shot 2025-09-08 at 7 31 58 PM" src="https://github.com/user-attachments/assets/03df3225-4232-48e3-b0a0-94949ccb1b99" />  
</p>

## 5️⃣ Automate User Onboarding  

<p align="center">  
✅ Using my script, I onboarded new accounts from the Excel file where <code>Status = Active</code>.  
Dynamic group rules automatically placed them into their correct RBAC groups.  
</p>

<p align="center"><em>PowerShell script execution creating users</em></p>

```powershell
# Bulk create users from CSV (automated onboarding)
Import-Csv "users.csv" | ForEach-Object {
    # Create new user account
    New-AzureADUser `
        -UserPrincipalName $_.UserPrincipalName `
        -DisplayName $_.DisplayName `
        -AccountEnabled $true `
        -MailNickname ($_.DisplayName.Split(' ')[0]) `
        -PasswordProfile @{Password = "TempP@ssw0rd"; ForceChangePasswordNextLogin = $true}

    # Add to department-based group (example: Sales)
    if ($_.Department -eq 'Sales') {
        $user = Get-AzureADUser -ObjectId $_.UserPrincipalName
        Add-AzureADGroupMember `
            -ObjectId "<SalesGroupObjectId>" `
            -RefObjectId $user.ObjectId
    }
}
```
<p align="center"><em>Azure AD Users view showing new accounts (Jane, Alice, Mike, Bob)</em></p>  
<p align="center">  
  <img width="675" height="375" alt="Screenshot (54)" src="https://github.com/user-attachments/assets/520f2786-c0e3-46a8-ac82-2cda7e42fe96" />  
</p>

## 6️⃣ Automate User Offboarding

<p align="center">  
✅ I disabled accounts where <code>Status = Inactive</code> in the Excel file.  
Since groups were dynamic, those users were automatically removed from group membership.  
</p>

<p align="center"><em>Confirmation after automating inactive accounts (Bob Smith)</em></p>  
<p align="center">  
  <img width="675" height="375" alt="Screenshot (59)" src="https://github.com/user-attachments/assets/8f2ec78f-dd98-4bb9-a14d-13449f281436" />  
</p>

## 7️⃣ Generate Access Review Report

<p align="center">  
✅ I exported a report of all users, their status, and their group memberships into a CSV for auditing.  
</p>

<p align="center"><em>Exporting <code>access_review_report.csv</code></em></p>  
<p align="center">  
  <img width="675" height="375" alt="Screenshot (61)" src="https://github.com/user-attachments/assets/1f66f29d-275e-4d5e-86e6-cec0cd8ffa57" />  
</p>  

<p align="center"><em>Excel view of <code>access_review_report.csv</code></em></p>  

<p align="center">
  <img width="675" height="375" alt="Screen Shot 2025-09-08 at 7 30 37 PM" src="https://github.com/user-attachments/assets/fb891bcd-00a2-4ea0-9536-505b70bf9cca" />
</p>

## 8️⃣ Create Groups for RBAC

<p align="center">  
✅ I set up <strong>dynamic groups</strong> for Sales, IT, and HR so membership was handled automatically.  
This way, I didn’t need to manually assign users to groups — Azure AD handled it for me.  
</p>

<p align="center"><em>Groups blade showing Sales, IT, and HR groups</em></p>  
<p align="center">  
  <img width="675" height="375" alt="Screenshot (49)" src="https://github.com/user-attachments/assets/649df1bb-d0a7-4d69-9071-fb379cea0d63" />  
</p>

## 9️⃣ Configure Conditional Access + MFA

<p align="center">  
✅ I built a Conditional Access policy requiring <strong>MFA for members of the Sales group</strong>.  
</p>

<p align="center"><em>Conditional Access policy setup with “Require MFA”</em></p>  
<p align="center">  
  <img width="675" height="375" alt="Screenshot (63)" src="https://github.com/user-attachments/assets/612fb928-59d2-485b-841f-0e2c3fe27dcc" />  
</p>

## 🔟 Test Conditional Access / MFA

<p align="center">  
✅ I signed in as <strong>Jane Doe</strong> to verify that MFA was correctly triggered.  
</p>

<p align="center"><em>Jane Doe signing in + MFA challenge</em></p>  
<p align="center">  
  <img width="675" height="375" alt="Screenshot (66)" src="https://github.com/user-attachments/assets/9850d657-ceda-454d-a3e8-ab24b056b06b" />  
</p>  

<p align="center">  
  <img width="675" height="375" alt="Screenshot (67)" src="https://github.com/user-attachments/assets/eed3c3a5-343c-45ee-a0ab-4a908db0ad21" />  
</p>

# 🎯 Final Outcome

By the end of this lab, I had:  

- 👤 Automated **user onboarding & offboarding** with Excel + PowerShell  
- 👥 Implemented **dynamic RBAC groups** for HR, IT, and Sales  
- 📊 Generated an **access review report** of users and group memberships  
- 🔐 Enforced **MFA** with Conditional Access  
