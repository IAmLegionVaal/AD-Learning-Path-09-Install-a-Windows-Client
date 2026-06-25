# AD Learning Path 09 — Install a Windows Client

> **Level:** Beginner  
> **Series:** Activity 9 of 64

## Objective
Create and install a Windows 11 Pro/Enterprise VM named `CL01` for domain membership, Group Policy, authentication, and troubleshooting activities.

## Prerequisites
- Activity 02 switch
- Windows 11 ISO
- 2 vCPU, 4 GB RAM, 64 GB storage
- Validated `DC01`

## Procedure
1. Create a Generation 2 VM with Secure Boot and virtual TPM, connect it to `AD-LAB-NAT`, and disable automatic checkpoints.
2. Install Windows 11 Pro or Enterprise; Home cannot join an on-prem AD domain.
3. Create a temporary local administrator, install updates, rename the computer to `CL01`, and restart.
4. Before DHCP exists, assign `10.10.10.101/24`, gateway `10.10.10.1`, and DNS `10.10.10.10`.
5. Create a clean pre-domain checkpoint.

## Implementation
```powershell
$VMName = 'CL01'
New-VM -Name $VMName -Generation 2 -MemoryStartupBytes 4GB `
    -NewVHDPath "C:\Hyper-V\Virtual Hard Disks\$VMName.vhdx" `
    -NewVHDSizeBytes 64GB -SwitchName 'AD-LAB-NAT'
Set-VMProcessor -VMName $VMName -Count 2
Set-VM -Name $VMName -AutomaticCheckpointsEnabled $false
Set-VMKeyProtector -VMName $VMName -NewLocalKeyProtector
Enable-VMTPM -VMName $VMName
```

## Validation
```powershell
Get-ComputerInfo | Select-Object CsName,WindowsProductName,WindowsVersion,OsBuildNumber
Get-Tpm
Confirm-SecureBootUEFI
Get-NetIPConfiguration
Resolve-DnsName dc01.corp.lab
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.corp.lab
```

## Evidence
Capture VM configuration, Windows edition/build, TPM and Secure Boot state, client network/DNS, pre-domain checkpoint, DC reachability, SRV lookup, deviations, remediation, and final pass/fail result under `evidence/`.

## Troubleshooting
- Domain option absent: confirm Pro/Enterprise edition.
- TPM requirement failure: enable VM TPM and Secure Boot.
- DNS lookup failure: use `10.10.10.10`, not the router or public DNS.

## Security notes
The temporary local administrator is later governed by Windows LAPS. Do not reuse domain Administrator credentials locally. Avoid personal cloud accounts in the lab.

## Rollback
Remove the VM only if rebuilding; keep the pre-domain checkpoint short-lived.

## References
- Microsoft Learn: Windows 11 Hyper-V VM requirements
- Microsoft Learn: Windows edition domain-join support

## Next activity
`AD-Learning-Path-10-Join-a-Windows-Client-to-the-Domain`
