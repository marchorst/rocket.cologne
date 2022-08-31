---
layout: post
title:  German Keyboard Layout - Powershell
tags: german powershell keyboard layout language
---
Install/set the german keyboard layout
```powershell
$LanguageList = Get-WinUserLanguageList
$LanguageList.Add("de-de")
$LanguageList.Add("en-de")
Set-WinUserLanguageList de-DE -force
Set-WinUserLanguageList $LanguageList
```