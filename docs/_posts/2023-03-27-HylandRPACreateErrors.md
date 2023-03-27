---
layout: post
title:  Hyland RPA: Create Error Entries from last 100 Tasks
tags: Create Error Entries from last 100 Tasks hylandrpa
---
Install/set the german keyboard layout
```powershell
$auth = "Bearer XXXX"
$errorUrl = 'http://heart.rpa.local/api/Errors'
$taskUrl = 'http://heart.rpa.local/api/tasks?sorts=-closedAt&page=1&pageSize=100'

$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("Authorization", $auth)

$response = Invoke-RestMethod $taskUrl -Method 'GET' -Headers $headers


$list = New-Object Collections.Generic.List[String]

foreach ($elem in $response.PsObject.Properties.Value)
{
    If($elem.remark -ne "") {
        If(!$list.Contains($elem.remark)){
            $list.Add($elem.remark);
        }
    }
    #echo  $elem.remark
}

foreach($e in $list | Get-Unique) {
    $d = $e.Split(':')
    If($d[0] -ne "0" -and $d[0] -ne "4000" -and $d[1] -ne $undefinedVariable) {
        $headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
        $headers.Add("Authorization", $auth)
        $headers.Add("Content-Type", "application/json")
        $body = "{`"code`": "
        $body += $d[0];
        $body += ",`"name`": `"";
        $body += $d[1].Trim();
        $body += "`"}"

        try {
            $response = Invoke-RestMethod $errorUrl -Method 'POST' -Headers $headers -Body $body
            echo "SUCCESS", $body;
        } catch { "ERROR", $body }
    }
}
```