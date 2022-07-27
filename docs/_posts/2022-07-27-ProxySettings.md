---
layout: post
title:  Get Proxy Config - Powershell
tags: proxy powershell activatation
---
To find out the ProxyData
```powershell
Get-ItemProperty -Path "Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
```
```powershell
([System.Net.WebRequest]::GetSystemWebproxy()).GetProxy("https://google.de")
```