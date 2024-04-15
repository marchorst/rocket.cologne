---
layout: post
title:  Export last runs
tags: rpa export runs platform
---
Export the last runs with a powershell script
```
# Define the URI and headers for the HTTP GET request
$params = ''
$uri = 'http://heart.rpa.local/api/Tasks'+ $params
$headers = @{
    'accept' = 'text/plain'
    'Authorization' = 'Bearer xxx'
}

# Make the HTTP GET request
$response = Invoke-RestMethod -Uri $uri -Method Get -Headers $headers

# Check if the response contains any tasks and if they have 'runs'
if ($response -and ($response.runs -ne $null)) {
    # Extract all runs from each task (Assuming multiple tasks could be returned)
    $allRuns = $response | ForEach-Object { 
        $processName = $_.process.name
        $runs = $_.runs | ForEach-Object {
            Add-Member -InputObject $_ -MemberType NoteProperty -Name "process" -Value $processName -Force
            return $_
        };
        return $runs 
    } | Where-Object { $_ -ne $null }
    
    $allRuns = $allRuns | ForEach-Object { 
        $_.remark = ($_.remark -split "`n")[0].Trim()  # Splits the remark at new lines and takes the first
        return $_
    }

    # Export the 'runs' data to a CSV file
    $allRuns | Export-Csv -Path "output.csv" -NoTypeInformation -Delimiter ";"
    Write-Host "Data exported to output.csv successfully."
} else {
    Write-Host "No 'runs' data found in the response."
}


```

JSON Export
```
# Define the URI and headers for the HTTP GET request
$params = ''
$tasksUri = 'http://heart.rpa.local/api/Tasks' + $params
$agentsUri = 'http://heart.rpa.local/api/Agents'
$headers = @{
    'Accept' = 'application/json'  # Correctly expect JSON data from the API
    'Authorization' = 'Bearer xxxx'
}

# Make the HTTP GET request to fetch tasks
$taskResponse = Invoke-RestMethod -Uri $tasksUri -Method Get -Headers $headers
# Make the HTTP GET request to fetch agents
$agentResponse = Invoke-RestMethod -Uri $agentsUri -Method Get -Headers $headers

# Check if the task response contains any tasks and if they have 'runs'
if ($taskResponse -and $taskResponse.runs) {
    $allRuns = $taskResponse | ForEach-Object { 
        $processName = $_.process.name
        
        $runs = $_.runs | ForEach-Object {
            $agentId = $_.agentId
            $agentDetails = $agentResponse | Where-Object {$_.agentId -eq $agentId} | Select-Object -Property instance
       
            Add-Member -InputObject $_ -MemberType NoteProperty -Name "processName" -Value $processName -Force
            Add-Member -InputObject $_ -MemberType NoteProperty -Name "agentDetails" -Value $agentDetails -Force
            return $_
        }
        return $runs 
    } | Where-Object { $_ -ne $null }

    $allRuns = $allRuns | ForEach-Object { 
        $_.remark = ($_.remark -split "`n")[0].Trim()  # Splits the remark at new lines and takes the first, trims any extra space
        return $_
    }

    # Convert the 'runs' data to JSON and save to a file
    $json = $allRuns | ConvertTo-Json -Depth 5  # Increase the depth to ensure nested objects are properly converted
    Set-Content -Path "output.json" -Value $json
    Write-Host "Data exported to output.json successfully."
} else {
    Write-Host "No 'runs' data found in the response or the response is null."
}

```