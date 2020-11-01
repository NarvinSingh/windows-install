Windows Install
===============

Resources and instructions to perform unattended installations of Windows.

Download
--------

[ESD Decrypter](httpsl://github.com/abbodi1406/WHD/raw/master/scripts/esd-decrypter-wimlib-56.7z)

### Windows 10 20H2 19042.572

[Education/Pro/Enterprise Volume Edition x64 ESD](http://dl.delivery.mp.microsoft.com/filestreamingservice/files/b65f7de0-5fd8-415b-92ed-cccd44632062/19042.572.201009-1947.20h2_release_svc_refresh_CLIENTBUSINESS_VOL_x64FRE_es-es.esd)

Create ISO
----------

Drag an ESD file to the ESD Decrypter `decrypt.cmd` and follow the prompts to create an ISO.

Create UEFI Bootable USB
------------------------

Format the first partition of the USB disk as FAT32 and copy the contents of the ISO file to the root of that partition. The USB should then be bootable under UEFI with secure Boot turned on.

Alternatively, you can create a general purpose Windows PE UEFI/BIOS bootable USB disk by installing the Windows ADK with Windows PE and running `MakeWinPEMedia`. See the instructions [here](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/winpe-create-usb-bootable-drive).

It is necessary to boot under UEFI to install Windows on a GPT partitioned hard drive.

Install
-------

1. Boot from the USB disk.
2. *Optional*: parition the hard drive. See the `DiskPart` script on [this page](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/configure-uefigpt-based-hard-drive-partitions) for instructions on how to partition the hard drive.
3. Run `setup.exe /unattend:<answerfile.xml>`

Keys
----

These keys can be used in an answer file or to manually install Windows, but Windows won't be activated by using them.

| Edition | Key |
| :------ | :-- |
| Windows 10 Enterprise Volume | NPPR9-FWDCX-D2C8J-H872K-2YT43 |
| Windows 10 Pro Volume | W269N-WFGWX-YVC9B-4J6C9-T83GX |

Activate
--------

1. Copy the files in `activate.wim` during installation by adding a `DataImage` section like this to your answer file:
```xml
<DataImage wcm:action="add">
    <Order>1</Order>
    <InstallTo>
        <DiskID>0</DiskID>
        <PartitionID>3</PartitionID>
    </InstallTo>
    <InstallFrom>
        <!-- This path is relative to the directory install.wim is in -->
        <Path>..\..\activate.wim</Path>
    </InstallFrom>
</DataImage>
```
Alternatively, you can manually extract the activation files from `activate.wim` after installation is complete.

2. Run the following commands as Administrator by adding `SynchronousCommand` sections like these to your answer file:
```xml
<SynchronousCommand wcm:action="add">
    <Order>1</Order>
    <CommandLine>powershell Set-MpPreference -DisableRealtimeMonitoring $true</CommandLine>
</SynchronousCommand>
<SynchronousCommand wcm:action="add">
    <Order>2</Order>
    <CommandLine>C:\Activate\AutoRenewal-Setup.cmd /u</CommandLine>
</SynchronousCommand>
<SynchronousCommand wcm:action="add">
    <Order>3</Order>
    <CommandLine>C:\Activate\Activate.cmd /u /w</CommandLine>
</SynchronousCommand>
<SynchronousCommand wcm:action="add">
    <Order>4</Order>
    <CommandLine>powershell Add-MpPreference -ExclusionPath C:\Windows\System32\SppExtComObjHook.dll</CommandLine>
</SynchronousCommand>
<SynchronousCommand wcm:action="add">
    <Order>5</Order>
    <CommandLine>rmdir /s /q C:\Activate</CommandLine>
</SynchronousCommand>
<SynchronousCommand wcm:action="add">
    <Order>6</Order>
    <CommandLine>powershell Set-MpPreference -DisableRealtimeMonitoring $false</CommandLine>
</SynchronousCommand>
```
Alternatively, you can manually run the preceding commands from an elevated prompt after installation is complete. If you do this, and you manually extraced the activation files to a custom location, be sure to use the correct paths in the commands that reference the activation files.

