Intune Proactive Remediation: Dynamic Application Grouping
This repository contains a set of scripts and a detailed guide for creating "dynamic-like" device groups in Microsoft Intune based on installed applications. This solution leverages Proactive Remediations and the Microsoft Graph API to automate group membership, addressing a common gap in native Intune functionality.

The Problem
Microsoft Intune's native dynamic groups are powerful for grouping devices based on properties like OS version, manufacturer, or device name. However, there is no built-in attribute to create a dynamic group based on a device's discovered applications.

This means that tasks like deploying an update to all devices that have "Google Chrome" installed, or creating a report for all machines with "Visual Studio Code," often require manual intervention or a separate, complex process.

The Solution
This guide provides a robust workaround using a combination of Proactive Remediations and the Microsoft Graph API. The process works as follows:

A detection script runs on all targeted devices to check for the presence of a specific application.

If the application is found, the device is marked as "non-compliant," and the remediation script is triggered.

The remediation script uses a secure App Registration and the Microsoft Graph API to add the device to a pre-defined Microsoft Entra ID security group.

This remediation package is scheduled to run regularly (e.g., daily) to automatically add or remove devices as applications are installed or uninstalled.

Prerequisites
To implement this solution, you will need the following:

Microsoft Entra ID P1/P2 or Windows 10/11 Enterprise E3/E5 licenses for the targeted users.

An administrative role in Microsoft Entra ID with permissions to manage App Registrations and security groups (e.g., Global Administrator, Application Administrator, or Groups Administrator).

An understanding of PowerShell scripting.

Implementation Guide
Follow these steps carefully to set up the solution in your environment.

Step 1: Create a Microsoft Entra App Registration
This App Registration will be used to authenticate your remediation script with the Microsoft Graph API.

Sign in to the Microsoft Entra admin center.

Go to Identity > Applications > App registrations and click New registration.

Name the application (e.g., Intune-Remediation-Group-Updater).

For Supported account types, select Accounts in this organizational directory only.

Click Register.

Navigate to API permissions > Add a permission > Microsoft Graph.

Select Application permissions and grant the following:

Device.Read.All

Group.ReadWrite.All

Click Grant admin consent for your organization.

Step 2: Create a Client Secret
On the App Registration page, go to Certificates & secrets > Client secrets.

Click New client secret, provide a description, and choose an expiration date.

Immediately copy the Value of the secret. This is your only chance to do so.

Step 3: Create the Security Group
This is the standard, empty group that your script will populate.

In the Microsoft Entra admin center, go to Groups > All groups.

Click New group.

Choose Security for Group type and set Membership type to Assigned.

Give it a name like SG-All-Devices-with-Chrome.

Copy the Group's Object ID.

Step 4: Prepare the PowerShell Scripts
Copy the following scripts. You will need to update the variables in the remediation script with the IDs and secret you collected in the previous steps.

Detect-Chrome.ps1 (Detection Script)
This script checks for the presence of the Chrome executable. It will return a 1 if Chrome is found, triggering the remediation.

Step 5: Deploy the Proactive Remediation in Intune
In the Microsoft Intune admin center, go to Devices > Scripts and remediations and click Create script package.

Give it a descriptive name and description.

On the Settings page, upload your two scripts:

Detection script file: Detect-Chrome

Remediation script file: AddChromeGroup

Leave other settings as default unless you have a specific reason to change them.

On the Assignments page, select the group of devices you want to run this on (e.g., All Windows devices).

Set the Schedule to a frequent interval, such as Daily or Hourly, to ensure group membership is kept up-to-date.

Complete the wizard by reviewing and creating the script package.

How to Use for Other Applications
To adapt this solution for a different application (e.g., Visual Studio Code), you only need to modify the detection script:

Create a new security group for the new application (e.g., SG-All-Devices-with-VSCode).

Create a new remediation package in Intune.

For the new package, use a new detection script that checks for Visual Studio Code, and update the remediation script's $groupId variable to the new group's Object ID.

Security and Best Practices
Client Secret Management: Treat your client secret as a password. Do not check it into public repositories. Use a secure method to store it. The remediation script is deployed via Intune, which is a trusted channel, but be mindful of who has access to the script content in the Intune console.

Principle of Least Privilege: The permissions (Device.Read.All, Group.ReadWrite.All) are broad. For a production environment, consider creating separate App Registrations for different scripts and scoping permissions more tightly if possible.

Troubleshooting: The Proactive Remediation feature provides excellent reporting. Check the Device status tab for your script package to view script results and error messages.
