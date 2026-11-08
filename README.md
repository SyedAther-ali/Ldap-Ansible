# 🔐 Automated LDAP Infrastructure with Ansible

<p align="center">

**Infrastructure-as-Code for Automated OpenLDAP Deployment, Configuration & Security**

</p>

<p align="center">

![Ansible](https://img.shields.io/badge/Ansible-Automation-EE0000?style=for-the-badge&logo=ansible&logoColor=white)
![OpenLDAP](https://img.shields.io/badge/OpenLDAP-LDAP-003366?style=for-the-badge)
![Linux](https://img.shields.io/badge/Linux-Server-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![TLS](https://img.shields.io/badge/TLS-LDAPS-2E8B57?style=for-the-badge)
![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=for-the-badge&logo=python&logoColor=white)

</p>

---

## 📌 Overview

This project automates the deployment and configuration of an **OpenLDAP server using Ansible**.

The objective is simple:

> **Take a fresh Linux machine and turn it into a functioning, secured LDAP server using a repeatable Ansible deployment.**

Instead of manually installing OpenLDAP, configuring the LDAP domain, creating users and groups, generating TLS certificates, configuring LDAPS, and verifying the deployment, these tasks are organized into an Ansible role and executed from a central controller.

The project is designed as a reusable infrastructure automation project that can be extended for larger identity-management and cybersecurity environments.

---

# 🧭 Project Goals

The main goals of this project are:

- Automate OpenLDAP installation
- Eliminate repetitive manual LDAP configuration
- Create a consistent LDAP directory structure
- Automate LDAP user creation
- Automate LDAP group creation
- Automate group membership
- Create dedicated LDAP service accounts
- Configure TLS certificates
- Enable LDAPS
- Store sensitive passwords using Ansible Vault
- Verify the LDAP deployment automatically
- Make the deployment repeatable on fresh machines
- Provide a foundation for future identity and security automation

---

# 🏗️ Architecture

The project follows a central-controller architecture.

```text
                         ┌─────────────────────────┐
                         │     Ansible Controller  │
                         │                         │
                         │        Kali Linux       │
                         │                         │
                         │  Inventory + Playbooks  │
                         │  Roles + Variables      │
                         │  Ansible Vault          │
                         └────────────┬────────────┘
                                      │
                                      │ SSH
                                      │
                         ┌────────────▼────────────┐
                         │      Managed Node       │
                         │                         │
                         │       LDAP Server       │
                         │                         │
                         │      OpenLDAP/slapd     │
                         └────────────┬────────────┘
                                      │
                    ┌─────────────────┼─────────────────┐
                    │                 │                 │
                    ▼                 ▼                 ▼
              ┌──────────┐      ┌──────────┐     ┌───────────────┐
              │  People  │      │  Groups  │     │ServiceAccounts│
              └────┬─────┘      └──────────┘     └───────────────┘
                   │
             ┌─────┴─────┐
             │           │
             ▼           ▼
          Alice         Bob

                     LDAPS : 636
                          │
                          ▼
                   TLS Certificates
                          │
                          ▼
                       Root CA
🔄 Deployment Flow

A typical deployment follows this sequence:

Fresh Linux Server
       │
       ▼
Ansible Connectivity Test
       │
       ▼
Install Prerequisites
       │
       ▼
Preconfigure slapd
       │
       ▼
Install OpenLDAP
       │
       ▼
Start LDAP Service
       │
       ▼
Generate TLS Certificates
       │
       ▼
Configure LDAPS
       │
       ▼
Create LDAP Directory
       │
       ▼
Create Users
       │
       ▼
Create Groups
       │
       ▼
Configure Group Membership
       │
       ▼
Create Service Account
       │
       ▼
Run Verification
       │
       ▼
       ✅ LDAP Server Ready


🧠 Understanding the Project

The project is built around three important technologies:

              ┌─────────────────────┐
              │       Ansible       │
              │                     │
              │ Infrastructure      │
              │ Automation          │
              └──────────┬──────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │      OpenLDAP       │
              │                     │
              │ Identity Directory  │
              └──────────┬──────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │       LDAPS         │
              │                     │
              │ TLS-secured LDAP    │
              └─────────────────────┘

Ansible

Ansible is responsible for:

Connecting to the server
Installing packages
Applying configuration
Creating LDAP objects
Generating certificates
Configuring services
Running verification
OpenLDAP

OpenLDAP provides the directory service.

It stores:

Users
Groups
Organizational Units
Service accounts
Directory information
LDAPS

LDAPS provides encrypted LDAP communication using TLS.

The deployment uses:

LDAPS
Port: 636
🗃️ LDAP Directory Structure

The LDAP directory uses the following base:

dc=ldap,dc=example,dc=internal

The structure is organized as:

dc=ldap,dc=example,dc=internal
│
├── ou=People
│   ├── uid=alice
│   └── uid=bob
│
├── ou=Groups
│   ├── cn=developers
│   └── cn=admins
│
└── ou=ServiceAccounts
    └── cn=django-bind

This separation makes it easier to manage different types of LDAP objects.

👤 LDAP Users

The current test environment contains users such as:

uid=alice,ou=People,...
uid=bob,ou=People,...

Users are created automatically by Ansible.

The user attributes include information such as:

Username
First name
Last name
UID
GID
Password
Object classes
👥 LDAP Groups

Groups are also created automatically.

Current groups include:

developers
admins

Membership is configured separately from group creation.

For example:

developers
│
├── alice
└── bob

admins
│
└── alice
🤖 Service Account

The deployment creates a dedicated LDAP service account:

cn=django-bind,ou=ServiceAccounts,dc=ldap,dc=example,dc=internal

The purpose of a service account is to allow applications such as Django to perform LDAP searches without using the LDAP administrator account.

This follows the principle of using a dedicated application identity instead of exposing the LDAP administrator credentials to applications.

🔐 TLS / LDAPS

The project automatically generates a local certificate hierarchy for the LDAP deployment.

LDAP Lab Root CA
       │
       │ signs
       ▼
LDAP Server Certificate
       │
       ▼
ldap.example.internal
       │
       ▼
LDAPS : 636

The automation creates:

ca.key
ca.crt

ldap-server.key
ldap-server.csr
ldap-server.crt

The certificates are then configured in OpenLDAP.

🔑 Secure Password Management

Sensitive credentials are not stored directly in the normal variable files.

Instead, Ansible Vault is used.

group_vars/
└── ldap_servers/
    ├── vars.yml
    └── vault.yml

The normal variables contain configuration such as:

ldap_domain: "ldap.example.internal"
ldap_base_dn: "dc=ldap,dc=example,dc=internal"

Sensitive values are stored in the encrypted Vault.

To edit the Vault:

ansible-vault edit group_vars/ldap_servers/vault.yml

To view it:

ansible-vault view group_vars/ldap_servers/vault.yml

Run the deployment with:

ansible-playbook -i inventory site.yml --ask-vault-pass


🧠 Project Learning

This project provides hands-on experience with:

Infrastructure-as-Code
Ansible
Linux administration
LDAP
OpenLDAP
Identity management
TLS / PKI
LDAPS
Secret management
Configuration management
Service accounts
Authentication infrastructure
Automated verification

The main objective is not only to deploy LDAP, but to understand how manual infrastructure configuration can be transformed into a repeatable and maintainable automation workflow.

👨‍💻 Author

Syed

Cybersecurity SOC Intern
IIT Hyderabad
