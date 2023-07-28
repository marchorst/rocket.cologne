---
layout: post
title: Hyland RPA - Clear DB and files (pdf)
tags: Hyland RPA - Clear DB and files (pdf)
---
Hyland RPA: Hyland RPA - Clear DB and files (pdf)
```powershell

# Define your database connection string
$connectionString = "Data Source=SERVERNAME;Initial Catalog=heart_local;Integrated Security=true;"

# Define the folder where you want to search for files
$folderPath = "D:\HylandRPA\repository\Storage"


# Function to execute SQL Query
function Execute-SqlQuery {
    param(
        [string]$connectionString,
        [string]$query
    )

    try {
        $connection = New-Object System.Data.SqlClient.SqlConnection
        $connection.ConnectionString = $connectionString
        $connection.Open()

        $command = $connection.CreateCommand()
        $command.CommandText = $query

        $dataAdapter = New-Object System.Data.SqlClient.SqlDataAdapter
        $dataAdapter.SelectCommand = $command

        $dataset = New-Object System.Data.DataSet
        $dataAdapter.Fill($dataset)

        return $dataset.Tables[0]
    }
    catch {
        Write-Host "Error executing SQL query: $_" -ForegroundColor Red
    }
    finally {
        $connection.Close()
    }
}


# Execute the SQL query
$sqlQuery = @"
SELECT TOP 1000 Run.StoppedAt, rd.DocumentId, f.FileId, f.Name
FROM heart_local.dbo.Run
JOIN heart_local.dbo.RunDocument rd on rd.RunId = Run.RunId
JOIN repository_local.dbo.[File] f on rd.DocumentId = f.DocumentId
WHERE Run.StoppedAt < DATEADD(day, -30, CAST(GETDATE() AS date))
AND f.Name LIKE '%.pdf'
ORDER BY Run.StoppedAt DESC
"@
$results = Execute-SqlQuery -connectionString $connectionString -query $sqlQuery

# Loop through the results and process each file
foreach ($row in $results) {
   
    if(-not ([string]::IsNullOrEmpty($row["FileId"]))) {
     Write-Host ($row | Format-Table | Out-String)
    $fileId = $row["FileId"]
    $fileName = $row["FileId"]
    $documentId = $row["DocumentId"]

    $filePath = Join-Path -Path $folderPath -ChildPath $fileName

    $child = Get-ChildItem -Path $folderPath -Filter $fileName -Recurse -ErrorAction SilentlyContinue -Force
    Write-Host ($child.Directory | Format-Table | Out-String)
    $filePath = Join-Path -Path $child.Directory -ChildPath $fileName

    # Check if the file exists in the specified folder
    if (Test-Path $filePath) {
        # Delete the file
        Remove-Item -Path $filePath -Force

    }
    else {
        Write-Host "File not found: $filePath" -ForegroundColor Yellow
    }
        # Execute SQL query to remove specific lines from the database
        $deleteQuery = "DELETE FROM repository_local.dbo.[File] WHERE FileId = '$fileId';"
        Execute-SqlQuery -connectionString $connectionString -query $deleteQuery
        
        $deleteQuery = "DELETE FROM repository_local.dbo.Documents WHERE DocumentId = '$documentId';"
        Execute-SqlQuery -connectionString $connectionString -query $deleteQuery
        
        $deleteQuery = "DELETE FROM heart_local.dbo.RunDocument WHERE DocumentId = '$documentId';"
        Execute-SqlQuery -connectionString $connectionString -query $deleteQuery


     }
}
```
