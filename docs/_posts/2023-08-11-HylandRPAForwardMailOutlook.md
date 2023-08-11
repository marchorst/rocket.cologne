---
layout: post
title: Hyland RPA - Forward mail with outlook interop
tags: Hyland RPA - Forward mail with outlook interop
---
Hyland RPA: Forward mail with outlook interop
```csharp
    #r "C:\Program Files\Microsoft Office\root\Office16\ADDINS\Microsoft Power Query for Excel Integrated\bin\Microsoft.Office.Interop.Outlook.dll"
    using System;
    using Microsoft.Office.Interop.Outlook;
    var mailId = mailMessage.Headers["UID"];
    var outlookApp = new Microsoft.Office.Interop.Outlook.Application();
    Microsoft.Office.Interop.Outlook.MailItem outlookOriginalMessage = null;
    try
    {
        Microsoft.Office.Interop.Outlook.NameSpace outlookNamespace = outlookApp.GetNamespace("MAPI");
        outlookOriginalMessage = outlookNamespace.GetItemFromID(mailId) as Microsoft.Office.Interop.Outlook.MailItem;

    }
    catch (System.Exception ex)
    {
        Console.WriteLine("Error: " + ex.Message);
        throw ex;
    }

    var newItem = outlookOriginalMessage.Forward();
    newItem.Recipients.Add(receiver);
    newItem.HTMLBody = addMessageInFront+newItem.HTMLBody;
    newItem.Send();

```
