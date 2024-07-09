# Backup SOP Automation Script

This repository contains a PowerShell script to automate the backup Standard Operating Procedure (SOP) for a server with an agent. The script handles incidents related to backup errors, restarts necessary services, resubmits jobs via the Commvault console, and handles ticket reassignment based on job success or failure.

## Table of Contents
- [Overview](#overview)
- [Script Steps](#script-steps)
- [Dependencies](#dependencies)
- [Usage](#usage)
- [License](#license)

## Overview
The script is designed to:
1. Handle incidents received from ServiceNow (SNOW).
2. Check the status of backup files for errors.
3. Restart specific services if an error is found.
4. Authenticate to the Commvault console using REST API.
5. Resubmit backup jobs and handle the outcome.
6. Process other tickets with the same error code and restart services on affected systems.

## Script Steps
1. **Define Variables**:
    - Set necessary variables including incident number, URLs, Commvault credentials, error descriptions, and services to restart.

2. **Authenticate to Commvault**:
    - The `Authenticate-Commvault` function logs into the Commvault console using REST API.

3. **Check Backup File Status**:
    - The `Check-BackupFileStatus` function checks the status of the backup file based on the incident number.

4. **Restart Services**:
    - The `Restart-Services` function restarts specified services if an error is found.

5. **Resubmit Job in Commvault**:
    - The `Resubmit-CommvaultJob` function resubmits a job in Commvault using the REST API.

6. **Main Script Logic**:
    - The script checks for errors, restarts services, authenticates to Commvault, and resubmits the job if necessary.
    - It prints appropriate messages and handles ticket reassignment if needed.

7. **Additional Logic**:
    - The script checks for other tickets with the same error code and handles them based on the CI type (Windows/Linux).

## Dependencies
- PowerShell 5.0 or higher.
- Access to the Commvault REST API.
- Necessary permissions to restart services and interact with the Commvault console.

## Usage
1. **Clone the Repository**:
    ```bash
    git clone https://github.com/yourusername/backup-sop-automation.git
    cd backup-sop-automation
    ```

2. **Set Variables**:
    - Update the script with the actual server, user details, Commvault credentials, and other necessary variables.

3. **Run the Script**:
    - Open PowerShell with the necessary permissions.
    - Execute the script:
    ```powershell
    .\backup_sop_automation.ps1
    ```

## Script Code

Here is the complete PowerShell script for reference:

```powershell
# Define variables
$incidentNumber = "INC2338692"
$snowProdUrl = "https://your-snow-prod-instance.com"
$commvaultConsoleUrl = "https://your-commvault-console.com"
$commvaultApiUrl = "$commvaultConsoleUrl/api"
$commvaultUsername = "your_commvault_username"
$commvaultPassword = "your_commvault_password"
$errorDescription = "Error:616"
$servicesToRestart = @("Service1", "Service2", "Service3")

# Function to authenticate to Commvault
function Authenticate-Commvault {
    $body = @{
        username = $commvaultUsername
        password = $commvaultPassword
    } | ConvertTo-Json

    $response = Invoke-RestMethod -Uri "$commvaultApiUrl/Login" -Method Post -Body $body -ContentType "application/json"
    return $response.token
}

# Function to check the status of the backup file
function Check-BackupFileStatus {
    param (
        [string]$incidentNumber
    )

    # Sample logic to check the status from the incident number (this should be replaced with actual logic)
    if ($incidentNumber -eq "INC2338692") {
        return $errorDescription
    } else {
        return "No Error"
    }
}

# Function to restart services
function Restart-Services {
    param (
        [array]$services
    )

    foreach ($service in $services) {
        Restart-Service -Name $service -Force
    }
}

# Function to resubmit the job in Commvault
function Resubmit-CommvaultJob {
    param (
        [string]$jobId,
        [string]$token
    )

    $headers = @{
        "Authtoken" = $token
    }

    $body = @{
        jobId = $jobId
    } | ConvertTo-Json

    $response = Invoke-RestMethod -Uri "$commvaultApiUrl/Job/$jobId/Resubmit" -Method Post -Headers $headers -Body $body -ContentType "application/json"
    return $response
}

# Main script logic
$errorStatus = Check-BackupFileStatus -incidentNumber $incidentNumber

if ($errorStatus -eq $errorDescription) {
    # Restart services if error is found
    Restart-Services -services $servicesToRestart

    # Authenticate to Commvault
    $token = Authenticate-Commvault

    if ($token) {
        # Resubmit the job (replace with actual job ID retrieval logic)
        $jobId = "12345"
        $response = Resubmit-CommvaultJob -jobId $jobId -token $token

        if ($response.status -eq "Success") {
            Write-Output "The job ID $jobId started, hence closing the ticket."
        } else {
            Write-Output "Failed to resubmit the job ID $jobId. Assigning to the Backup Team."
            # Logic to assign to the Backup Team (e.g., update SNOW ticket)
        }
    } else {
        Write-Output "Failed to authenticate to Commvault."
    }
} else {
    Write-Output "No error found or different error."
}

# Additional logic to check for other tickets with the same error code and handle accordingly
# Replace with actual logic to retrieve and process tickets
$affectedCIs = @("CI1", "CI2")

foreach ($ci in $affectedCIs) {
    if ($ci -like "*Windows*") {
        # Logic to login to Windows CI and restart services
        Restart-Services -services $servicesToRestart
    } elseif ($ci -like "*Linux*") {
        # Logic to login to Linux CI and restart services
        Write-Output "Login to Linux CI and restart services."
    }
}

# This `README.md` file provides a comprehensive guide to the script, including an overview, detailed script steps, dependencies, usage instructions, and the full script code for reference. Adjust the repository URL and any specific details as needed.
