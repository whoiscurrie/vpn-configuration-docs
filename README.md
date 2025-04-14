# VPN Server Configuration Guide

> **Domain:** `fakecompany.jm`  
> **VPN Types:** SSTP and L2TP/IPSec  
> **Server Role:** Windows Server 2019+ (Domain Member)  
> **Goal:** Configure secure remote access via VPN for authorized users.

---
## Overview

This guide outlines the configuration process for deploying VPN services on a Windows Server integrated into a domain. Both **SSTP** and **L2TP/IPSec** are implemented to support different client capabilities. Certificate Services (CA) and RRAS are used to support secure connectivity.

---

## Prerequisites

- Domain-joined server (e.g., `vpnserver.fakecompany.jm`)
- Windows Server 2019 or later
- Static public IP for VPN clients to connect
- Internal DNS & DHCP
- Internet access
- Administrator privileges

---

## Step 1: Join Server to Domain

Open PowerShell and run:

```powershell
Add-Computer -DomainName "fakecompany.jm" -Credential fakecompany\Administrator
Restart-Computer
```

---

## Step 2: Install Roles and Features

Install Remote Access and Certificate Services:
```powershell
Install-WindowsFeature RemoteAccess, RSAT-RemoteAccess, RemoteAccess-PowerShell, Web-Server
Install-WindowsFeature AD-Certificate, ADCS-Cert-Authority -IncludeManagementTools
```

Configure AD Certificate Services via the Server Manager wizard. Choose:

    Role: Certification Authority

    Type: Enterprise CA

    CA Type: Root CA (or Subordinate if you already have a Root)

---

## Step 3: Configure Certificates

Generate an SSL certificate (for SSTP):

    Open Certification Authority

    Create a new certificate template (e.g., VPN Server Auth)

    Include Server Authentication in EKU

    Request certificate via MMC > Certificates (Computer) > Personal > All Tasks > Request New Certificate

## Step 4: RRAS Setup
Open Server Manager > Tools > Routing and Remote Access:

    Right-click server > Configure and Enable Routing and Remote Access

    Choose Custom Configuration

    Select:

        VPN Access

        NAT

    Start the RRAS service

## Step 5: Configure SSTP

    Go to RRAS > Server Properties > Security tab

    Select the SSL certificate created earlier

    Ensure SSTP is enabled

    Port 443 should be open in the firewall and router

## Step 6: Configure L2TP/IPSec
In RRAS:

    Go to Ports

    Right-click WAN Miniport (L2TP) > Properties

    Set maximum number of ports

    Under IPSec Settings, enter a Pre-shared Key

Example:

PSK: fakecompany2025!

## Step 7: Configure Firewall and Routing

On Server:

netsh advfirewall firewall add rule name="VPN - SSTP" protocol=TCP dir=in localport=443 action=allow
netsh advfirewall firewall add rule name="VPN - L2TP" protocol=UDP dir=in localport=1701 action=allow
netsh advfirewall firewall add rule name="VPN - IPSec" protocol=UDP dir=in localport=500 action=allow
netsh advfirewall firewall add rule name="VPN - NAT-T" protocol=UDP dir=in localport=4500 action=allow

Enable NAT:

    Go to RRAS > IPv4 > NAT

    Right-click External Interface > Properties

    Check Enable NAT on this interface

## Step 8: User and Group Management

    Create a security group: VPN_Users

    Add users who should have VPN access

    In RRAS, set Remote Access Policy to allow connections from members of VPN_Users

Testing and Validation

    SSTP Test: Connect using Windows built-in VPN client

        Server: vpn.acmecorp.jm

        Type: SSTP

    L2TP Test:

        Use same server

        Set Pre-shared Key in the client

        Enable CHAP/MS-CHAPv2
