# 🔐 End-to-End Identity Lifecycle Automation and Security  

This lab demonstrates how I automated **user onboarding, offboarding, access reviews, and MFA enforcement** in Azure Active Directory using **PowerShell + Microsoft Graph**.  

---


🖼️ *Azure Active Directory Overview Page — showing tenant name + AAD blade open*  
<img width="1920" height="958" alt="Screenshot (45)" src="https://github.com/user-attachments/assets/f977f72c-1f96-4ba2-a7cc-2f9a374b308c" />


---

## 2️⃣ Install Required PowerShell Modules  

✅ I installed the required PowerShell modules (`AzureAD` and `Microsoft.Graph`) to interact with Azure AD.  

📸 **Screenshot 2:** PowerShell window showing successful installation  
<img width="1920" height="956" alt="Screenshot (46)" src="https://github.com/user-attachments/assets/a4615676-678b-44cd-915f-83e295401f89" />


---

## 3️⃣ Register Azure AD App for Automation  

✅ I registered an app called **IdentityFlow** in Azure AD, which I used for scripting.  
I granted it permissions to manage users and groups so my automation scripts could run securely.  

📸 **Screenshot 3:** App Registration Overview (IdentityFlow + Client ID)  
<img width="1920" height="960" alt="Screenshot (47)" src="https://github.com/user-attachments/assets/b0824adb-beb3-42a0-9948-f940162f0c90" />

📸 **Screenshot 4:** API Permissions showing Graph permissions with admin consent  
<img width="1920" height="954" alt="Screenshot (48)" src="https://github.com/user-attachments/assets/938a077f-61b7-4ebc-9172-80b5dae96a6c" />

---

## 8️⃣ Create Groups for RBAC  

✅ I set up **dynamic groups** for Sales, IT, and HR so membership was handled automatically.  
This way, I didn’t need to manually assign users to groups — Azure AD handled it for me.  

📸 **Screenshot 10:** Groups blade showing Sales, IT, and HR groups  
<img width="1920" height="954" alt="Screenshot (49)" src="https://github.com/user-attachments/assets/649df1bb-d0a7-4d69-9071-fb379cea0d63" />

---

## 4️⃣ Create Excel File with User Data  

✅ I built an **Excel file (`users.csv`)** that contained user details such as UPN, display name, department, job title, manager, and status.  
This served as the input for both onboarding and offboarding.  

📸 **Screenshot 5:** Excel file with user records  
<img width="763" height="378" alt="Screen Shot 2025-09-08 at 7 30 37 PM" src="https://github.com/user-attachments/assets/fb891bcd-00a2-4ea0-9536-505b70bf9cca" />

---

## 5️⃣ Automate User Onboarding  

✅ Using my script, I onboarded new accounts from the Excel file where `Status = Active`.  
Dynamic group rules automatically placed them into their correct RBAC groups.  

📸 **Screenshot 6:**  PowerShell script execution creating users  
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


📸 **Screenshot 7:** Azure AD Users view showing new accounts (Jane, Alice, Mike, Bob)  
<img width="1920" height="950" alt="Screenshot (54)" src="https://github.com/user-attachments/assets/520f2786-c0e3-46a8-ac82-2cda7e42fe96" />


---

## 6️⃣ Automate User Offboarding  

✅ I disabled accounts where `Status = Inactive` in the Excel file.  
Since groups were dynamic, those users were automatically removed from group membership.  

📸 **Screenshot 8:** Confirmation after automating Inactive accounts(Bob Smith)  
<img width="1920" height="951" alt="Screenshot (59)" src="https://github.com/user-attachments/assets/8f2ec78f-dd98-4bb9-a14d-13449f281436" />

---

## 7️⃣ Generate Access Review Report  

✅ I exported a report of all users, their status, and their group memberships into a CSV for auditing.  

📸 **Screenshot 9:** Using Script to export `access_review_report.csv` 
<img width="1920" height="962" alt="Screenshot (61)" src="https://github.com/user-attachments/assets/1f66f29d-275e-4d5e-86e6-cec0cd8ffa57" />

📸 **Screenshot 10:** Excel view of `access_review_report.csv` showing accounts and groups
<img width="820" height="400" alt="Screen Shot 2025-09-08 at 7 31 58 PM" src="https://github.com/user-attachments/assets/03df3225-4232-48e3-b0a0-94949ccb1b99" />

---

## 9️⃣ Configure Conditional Access + MFA  

✅ I built a Conditional Access policy requiring **MFA for members of the Sales group**.  

📸 **Screenshot 11:** Conditional Access policy setup with “Require MFA”  
<img width="1920" height="956" alt="Screenshot (63)" src="https://github.com/user-attachments/assets/612fb928-59d2-485b-841f-0e2c3fe27dcc" />

---

## 🔟 Test Conditional Access / MFA  

✅ I signed in as **Jane Doe** to verify that MFA was correctly triggered.  

📸 **Screenshot 12:** Login screen showing MFA challenge  
<img width="1920" height="956" alt="Screenshot (66)" src="https://github.com/user-attachments/assets/9850d657-ceda-454d-a3e8-ab24b056b06b" />

<img width="1920" height="956" alt="Screenshot (67)" src="https://github.com/user-attachments/assets/eed3c3a5-343c-45ee-a0ab-4a908db0ad21" />

---

# 🎯 Final Outcome  

By the end of this lab, I had:  
- 👤 Automated **user onboarding & offboarding** with Excel + PowerShell  
- 👥 Implemented **dynamic RBAC groups** for HR, IT, and Sales  
- 📊 Generated an **access review report** of users and group memberships  
- 🔐 Enforced **MFA** with Conditional Access  

