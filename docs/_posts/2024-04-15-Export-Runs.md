---
layout: post
title:  Export last runs
tags: rpa export runs platform
---
Export the last runs with a powershell script
```
# Define the URI and headers for the HTTP GET request
$uri = 'http://heart.rpa.local/api/Tasks'
$headers = @{
    'accept' = 'text/plain'
    'Authorization' = 'Bearer xxx'
}

# Make the HTTP GET request
$response = Invoke-RestMethod -Uri $uri -Method Get -Headers $headers

# Check if the response contains any tasks and if they have 'runs'
if ($response -and ($response.runs -ne $null)) {
    # Extract all runs from each task (Assuming multiple tasks could be returned)
    $allRuns = $response | ForEach-Object { $_.runs } | Where-Object { $_ -ne $null }
    
    $allRuns = $allRuns | ForEach-Object { 
        $_.remark = ($_.remark -split "`n")[0]  # Splits the remark at new lines and takes the first
        return $_
    }

    # Export the 'runs' data to a CSV file
    $allRuns | Export-Csv -Path "output.csv" -NoTypeInformation
    Write-Host "Data exported to output.csv successfully."
} else {
    Write-Host "No 'runs' data found in the response."
}

```
