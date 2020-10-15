---
title: Errors occur when bringing a disk resource
description: Describes a problem in which various errors messages may be logged for a resource on a server that is running Windows Server 2008. Provides how to work around this problem.
ms.date: 09/15/2020
author: Deland-Han 
ms.author: delhan
manager: dscontentpm
audience: itpro
ms.topic: troubleshooting
ms.prod: windows-server
localization_priority: medium
ms.reviewer: kaushika, stevenek
ms.prod-support-area-path: Data corruption and disk errors
ms.technology: BackupStorage
---
# A physical disk resource remains in the Online Pending status, or the Chkdsk utility automatically starts to run on a server that is running Windows Server 2008

This article helps fix various errors messages may be logged when you bring a physical disk resource online.

_Original product version:_ &nbsp; Windows Server 2012 R2  
_Original KB number:_ &nbsp; 977516

## Symptoms

When you bring a physical disk resource online on a server that is running Windows Server 2008, you may experience one of the following symptoms:

### Symptom 1

When you view the physical disk resource in the Failover Cluster Management snap-in, the resource may show a status of **Online Pending**. Additionally, the following error message is logged in the System log:

> Log Name: System  
Source: Microsoft-Windows-FailoverClustering  
Event ID: 1066  
Task Category: Physical Disk Resource  
Level: Warning  
Description:  
Cluster disk resource 'Cluster Disk 3' indicates corruption for volume '\\\\?\Volume{ec2fa15d-b438-11de-88bc-00155dd99d36}'. Chkdsk is being run to repair problems. The disk will be unavailable until Chkdsk completes. Chkdsk output will be logged to file 'C:\Windows\Cluster\Reports\ChkDsk_ResCluster Disk 3_Disk2Part1.log'.  
Chkdsk may also write information to the Application Event Log.

Additionally, the following error message is logged in the Cluster log:

> ERR [RES] Physical Disk <Cluster Disk 3>: VerifyFS: Failed to open file \\\\?\GLOBALROOT\Device\Harddisk2\Partition1\TextDocument.txt Error: 5.

### Symptom 2

When you view the physical disk resource in the Microsoft Cluster Administrator utility, you may experience one or more of the following symptoms:

- The resource may not come online, or it may come online after a brief delay.
- The Chkdsk utility together with the /F switch automatically starts to run on the shared hard disk.
- Event ID 1066 that has a description similar to the following appears in the System log of the Event Viewer:
    > Cluster resource Disk Y:: is corrupt. Running ChkDsk /F to repair problems.

## Cause

These issues occur for one of the following reasons.

### Cause of Symptom 1

This issue occurs because a read-only file is located in the root directory of the resource. When a shared physical disk resource is brought online, the cluster service enumerates the files of the root directory and tries to open each file together with full access. This behavior occurs to make sure that the file system is consistent and that the volume is not corrupted. If any of the files are read-only in the root directory of the resource, the volume is considered corrupted and Chkdsk is started. To work around this problem, use the workaround that is mentioned in the [Workaround 1](#workaround-1) section.

### Cause of Symptom 2

This issue occurs because the "dirty" flag for the volume is set. When a shared physical disk resource is brought online, the cluster service enumerates the files of the root directory and tries to open each file together with full access. This behavior occurs to make sure that the file system is consistent and that the volume is not corrupted. If any of the files have the "dirty" flag set in the root directory of the resource, the volume is considered corrupted and Chkdsk is started. To work around this problem, use the workaround that is mentioned in the [Workaround 2](#workaround-2) section.

## Workaround

### Workaround 1

To work around this problem, do one of the following:

- Clear the read-only attribute from the files by either viewing the file properties or by using the **attrib -r** command at a command prompt.
- Move the files that have the read-only attribute from the root directory of the resource to an appropriate subfolder.

> [!NOTE]
> If you fail to bring the disk online and perform any further check on the files, please set the Physical Disk private property DiskRunChkDsk to a property of 4 (ChkDskDontRun). This will disable volume mount checks.

### Workaround 2

To work around this problem, first determine whether the "dirty" flag for the specified volume is set.

To determine whether the "dirty" flag is set for the volume in Windows Server 2008, use the Chkntfs tool.

For more information about the Chkntfs tool, visit the following Microsoft TechNet Web site:  
[https://technet.microsoft.com/library/cc731298(WS.10).aspx](https://technet.microsoft.com/library/cc731298%28ws.10%29.aspx) 

To determine whether the "dirty" flag is set for the volume in Windows Server 2008 R2, use the Validate a Configuration Wizard.

For more information about the Validate a Configuration Wizard, visit the following Microsoft TechNet Web site:  
[https://technet.microsoft.com/library/cc772055.aspx](https://technet.microsoft.com/library/cc772055.aspx)

If the "dirty" flag is set for the volume, run the Chkdsk utility together with the /F switch.

For more information about the Chkdsk utility, visit the following Microsoft TechNet Web site:  
[https://technet.microsoft.com/library/cc730714(WS.10).aspx](https://technet.microsoft.com/library/cc730714%28ws.10%29.aspx) 

## Status

Microsoft has confirmed that this is a problem in the Microsoft products that are listed in the Applies to section.