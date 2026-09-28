# Ansible LDAP Server Automation

Automated deployment and configuration of an OpenLDAP server using Ansible.

The goal of this project is to convert a fresh Linux machine into a
functional LDAP server without manually installing and configuring LDAP
on the target machine.

The deployment includes:

- OpenLDAP installation
- LDAP domain configuration
- LDAP administrator configuration
- LDAP directory structure
- LDAP users
- LDAP groups
- Group membership
- LDAP service account
- Internal TLS / LDAPS
- LDAP service configuration
- Automated verification
- Secret management using Ansible Vault

---

# Table of Contents

1. [Project Overview](#project-overview)
2. [Why Ansible](#why-ansible)
3. [What is LDAP](#what-is-ldap)
4. [What is OpenLDAP](#what-is-openldap)
5. [What is Ansible](#what-is-ansible)
6. [Architecture](#architecture)
7. [Project Structure](#project-structure)
8. [LDAP Directory Structure](#ldap-directory-structure)
9. [Prerequisites](#prerequisites)
10. [Preparing a Fresh Server](#preparing-a-fresh-server)
11. [Adding a New Server to Inventory](#adding-a-new-server-to-inventory)
12. [Testing SSH Connectivity](#testing-ssh-connectivity)
13. [Ansible Vault](#ansible-vault)
14. [Configuration Variables](#configuration-variables)
15. [Running the Deployment](#running-the-deployment)
16. [What Happens During Deployment](#what-happens-during-deployment)
17. [TLS / LDAPS](#tls--ldaps)
18. [Users](#users)
19. [Groups](#groups)
20. [Service Account](#service-account)
21. [Verification](#verification)
22. [Testing Authentication](#testing-authentication)
23. [Idempotency](#idempotency)
24. [Adding Another LDAP Server](#adding-another-ldap-server)
25. [Troubleshooting](#troubleshooting)
26. [Security Notes](#security-notes)
27. [Future Improvements](#future-improvements)

---

# Project Overview

This project automates the deployment of an OpenLDAP server using Ansible.

The intended workflow is:

    Fresh Linux Server
            |
            | SSH
            v
    Ansible Controller
            |
            | Ansible Playbook
            v
    LDAP Server
            |
            +--> OpenLDAP
            |
            +--> LDAP Directory
            |
            +--> Users
            |
            +--> Groups
            |
            +--> Service Account
            |
            +--> TLS / LDAPS
            |
            +--> Verification

The target server does not need to have OpenLDAP manually configured.

Ansible performs the installation and configuration.

---

# Why Ansible?

Without automation, LDAP deployment normally involves manually performing
multiple steps:

1. Install OpenLDAP
2. Configure the LDAP domain
3. Configure the administrator
4. Configure the directory
5. Create organizational units
6. Create users
7. Create groups
8. Configure group membership
9. Generate certificates
10. Configure TLS
11. Configure LDAPS
12. Verify the installation

This becomes difficult to reproduce consistently.

Ansible allows the entire process to be described as code.

The same playbook can therefore be used to deploy the LDAP server again on
another machine.

---

# What is LDAP?

LDAP stands for:

    Lightweight Directory Access Protocol

LDAP is a protocol used to access and manage directory information.

A directory can contain objects such as:

- Users
- Groups
- Computers
- Service accounts
- Organizational units

LDAP stores information in a hierarchical structure.

Example:

    dc=ldap,dc=example,dc=internal
    |
    +-- ou=People
    |     |
    |     +-- uid=alice
    |     |
    |     +-- uid=bob
    |
    +-- ou=Groups
    |     |
    |     +-- cn=developers
    |     |
    |     +-- cn=admins
    |
    +-- ou=ServiceAccounts
          |
          +-- cn=django-bind

---

# What is OpenLDAP?

OpenLDAP is an open-source implementation of the LDAP protocol.

This project uses:

    slapd

as the OpenLDAP server.

LDAP clients communicate with the server using LDAP operations such as:

- Bind
- Search
- Add
- Modify
- Delete

---

# What is Ansible?

Ansible is an automation and configuration-management tool.

Ansible uses:

- Inventory
- Playbooks
- Roles
- Variables
- Handlers
- Templates
- Modules
- Vault

The machine running Ansible is called the:

    Controller

The machine being configured is called the:

    Managed Node

In this project:

    Kali Linux
        |
        | SSH
        v
    LDAP Server

---

# Architecture

Example deployment:

    +---------------------------+
    |       Ansible Controller  |
    |                           |
    |       Kali Linux          |
    |                           |
    |       ~/ldap-ansible      |
    +-------------+-------------+
                  |
                  | SSH
                  |
                  v
    +---------------------------+
    |       LDAP Server         |
    |                           |
    |       ldap01              |
    |                           |
    |       OpenLDAP             |
    |       LDAPS :636          |
    +---------------------------+

The controller does not need LDAP installed.

It only needs Ansible and the required Ansible collections/modules.

---

# Project Structure

    ldap-ansible/
    |
    +-- README.md
    |
    +-- .gitignore
    |
    +-- inventory
    |
    +-- site.yml
    |
    +-- group_vars/
    |   |
    |   +-- ldap_servers/
    |       |
    |       +-- vars.yml
    |       +-- vault.yml
    |
    +-- playbooks/
    |
    +-- roles/
        |
        +-- ldap_server/
            |
            +-- defaults/
            +-- files/
            +-- handlers/
            |   +-- main.yml
            |
            +-- tasks/
                |
                +-- main.yml
                +-- install.yml
                +-- configure.yml
                +-- tls.yml
                +-- directory.yml
                +-- users.yml
                +-- groups.yml
                +-- membership.yml
                +-- service_account.yml
                +-- verify.yml

---

# Inventory

The inventory defines which machines Ansible manages.

Example:

    [ldap_servers]
    ldap01 ansible_host=10.9.98.103

For another server:

    [ldap_servers]
    ldap01 ansible_host=10.9.98.103
    ldap02 ansible_host=10.9.99.92

The hostname on the left is an Ansible inventory name.

The IP address is specified using:

    ansible_host

---

# Prerequisites

## Controller

The Ansible controller should have:

- Ansible
- SSH client
- Required Ansible collections
- SSH access to the target server

Check Ansible:

    ansible --version

---

## Managed Server

The target server should have:

- Linux
- SSH server
- Python
- A user with sudo privileges

OpenLDAP does not need to be manually installed.

The playbook installs it.

---

# Preparing a Fresh Server

When using a completely fresh server, first make sure:

1. The server has network connectivity.
2. SSH is running.
3. You know the server IP.
4. You have a user that can use sudo.
5. The controller can SSH into the server.

Example:

    ssh kali@10.9.99.92

If SSH works, exit:

    exit

---

# Adding a Fresh Server

Edit:

    inventory

Example:

    [ldap_servers]
    ldap02 ansible_host=10.9.99.92

If the server uses a different SSH username:

    [ldap_servers]
    ldap02 ansible_host=10.9.99.92 ansible_user=kali

---

# Test Ansible Connectivity

Before running the full deployment, always test connectivity.

Run:

    ansible ldap_servers -i inventory -m ping

Expected result:

    ldap02 | SUCCESS => {
        "changed": false,
        "ping": "pong"
    }

If this does not work, do not run the LDAP deployment yet.

Fix SSH/Ansible connectivity first.

---

# Become / Sudo Access

The LDAP installation requires root privileges.

The main playbook contains:

    become: true

Therefore Ansible will use sudo on the managed server.

If the sudo password is required, run:

    ansible-playbook -i inventory site.yml -K

The `-K` option asks for the privilege-escalation password.

---

# Ansible Vault

Passwords must not be stored in plaintext in Git.

Sensitive values are stored in:

    group_vars/ldap_servers/vault.yml

The file should be encrypted using Ansible Vault.

Check:

    head -n 1 group_vars/ldap_servers/vault.yml

Expected:

    $ANSIBLE_VAULT;1.1;AES256

---

# Creating the Vault

If creating the project from scratch:

    mkdir -p group_vars/ldap_servers

    ansible-vault create group_vars/ldap_servers/vault.yml

Example contents:

    ldap_admin_password: "CHANGE_ME"

    ldap_users:
      - username: alice
        first_name: Alice
        last_name: Admin
        uid_number: 10001
        gid_number: 10000
        password: "CHANGE_ME"

      - username: bob
        first_name: Bob
        last_name: User
        uid_number: 10002
        gid_number: 10000
        password: "CHANGE_ME"

    ldap_service_account:
      username: django-bind
      password: "CHANGE_ME"

Save and exit.

---

# Viewing the Vault

To view encrypted variables:

    ansible-vault view group_vars/ldap_servers/vault.yml

To edit:

    ansible-vault edit group_vars/ldap_servers/vault.yml

Never commit an unencrypted copy of this file.

---

# Configuration Variables

Normal non-secret configuration is stored in:

    group_vars/ldap_servers/vars.yml

Current example:

    ldap_domain: "ldap.example.internal"

    ldap_base_dn: "dc=ldap,dc=example,dc=internal"

    ldap_organization: "LDAP Example Organization"

    ldap_admin_dn: "cn=admin,dc=ldap,dc=example,dc=internal"

TLS configuration:

    ldap_tls_dir: "/etc/ldap/tls"

    ldap_tls_ca_key: "/etc/ldap/tls/ca.key"
    ldap_tls_ca_cert: "/etc/ldap/tls/ca.crt"

    ldap_tls_server_key: "/etc/ldap/tls/ldap-server.key"
    ldap_tls_server_csr: "/etc/ldap/tls/ldap-server.csr"
    ldap_tls_server_cert: "/etc/ldap/tls/ldap-server.crt"

    ldap_tls_common_name: "ldap.example.internal"

    ldap_tls_subject_alt_names:
      - "DNS:ldap.example.internal"
      - "DNS:ldap01"

---

# Main Playbook

The main playbook is:

    site.yml

It contains:

    - name: Deploy LDAP Server
      hosts: ldap_servers
      become: true
      roles:
        - ldap_server

This means the LDAP role is applied to every machine in:

    [ldap_servers]

---

# Role Execution Flow

The main role executes tasks in this order:

    1. Install prerequisites
            |
            v
    2. Configure LDAP installation parameters
            |
            v
    3. Install OpenLDAP
            |
            v
    4. Start slapd
            |
            v
    5. Configure TLS
            |
            v
    6. Restart / reload LDAP
            |
            v
    7. Create directory structure
            |
            v
    8. Create users
            |
            v
    9. Create groups
            |
            v
    10. Configure membership
            |
            v
    11. Create service account
            |
            v
    12. Verify deployment

---

# Task Files

## install.yml

Responsible for installing the LDAP packages.

Typical packages include:

    slapd
    ldap-utils

---

## configure.yml

Preconfigures the LDAP installation.

It sets:

- LDAP domain
- Organization name
- Administrator password
- Initial slapd configuration

The administrator password is obtained from Ansible Vault.

---

## directory.yml

Creates the LDAP directory structure.

Example:

    dc=ldap,dc=example,dc=internal

with:

    ou=People
    ou=Groups
    ou=ServiceAccounts

---

## users.yml

Creates LDAP users.

Current example users:

    alice
    bob

User information includes:

- Username
- First name
- Last name
- UID
- GID
- Password

Passwords are stored in Ansible Vault.

---

## groups.yml

Creates LDAP groups.

Current groups:

    developers
    admins

Example:

    developers -> gidNumber 10000
    admins     -> gidNumber 10001

---

## membership.yml

Configures group membership.

Current configuration:

    Alice -> developers
    Bob   -> developers
    Alice -> admins

---

## service_account.yml

Creates a service account for applications that need to
search the LDAP directory.

Current service account:

    cn=django-bind,ou=ServiceAccounts,...

The service account is intended for applications such as Django.

---

# LDAP Directory Structure

The resulting directory is approximately:

    dc=ldap,dc=example,dc=internal
    |
    +-- ou=People
    |   |
    |   +-- uid=alice
    |   |
    |   +-- uid=bob
    |
    +-- ou=Groups
    |   |
    |   +-- cn=developers
    |   |
    |   +-- cn=admins
    |
    +-- ou=ServiceAccounts
        |
        +-- cn=django-bind

---

# TLS / LDAPS

The project configures encrypted LDAP communication.

LDAP normally uses:

    ldap://

LDAPS uses:

    ldaps://

This project uses:

    ldaps://ldap.example.internal:636

---

# Certificate Architecture

The automation creates a small internal CA:

    LDAP Lab Root CA
            |
            | signs
            v
    LDAP Server Certificate
            |
            v
    ldap.example.internal

Files are stored on the LDAP server under:

    /etc/ldap/tls/

Important files:

    ca.key
    ca.crt

    ldap-server.key
    ldap-server.csr
    ldap-server.crt

The CA private key must be protected.

---

# LDAPS Verification

From a client that trusts the CA:

    openssl s_client \
      -connect ldap.example.internal:636 \
      -servername ldap.example.internal \
      -CAfile ldap-ca.crt \
      -verify_return_error

A successful verification should end with:

    Verify return code: 0 (ok)

---

# LDAP Authentication Test

Admin bind:

    LDAPTLS_CACERT=/path/to/ca.crt ldapsearch -x \
      -H ldaps://ldap.example.internal:636 \
      -D "cn=admin,dc=ldap,dc=example,dc=internal" \
      -W \
      -b "dc=ldap,dc=example,dc=internal" \
      "(objectClass=*)" \
      dn

---

# Test a User

Test Alice:

    LDAPTLS_CACERT=/path/to/ca.crt ldapwhoami -x \
      -H ldaps://ldap.example.internal:636 \
      -D "uid=alice,ou=People,dc=ldap,dc=example,dc=internal" \
      -W

Successful authentication should return:

    dn:uid=alice,ou=People,dc=ldap,dc=example,dc=internal

---

# Test the Django Service Account

Example:

    LDAPTLS_CACERT=/path/to/ca.crt ldapsearch -x \
      -H ldaps://ldap.example.internal:636 \
      -D "cn=django-bind,ou=ServiceAccounts,dc=ldap,dc=example,dc=internal" \
      -W \
      -b "ou=People,dc=ldap,dc=example,dc=internal" \
      "(uid=alice)" \
      dn uid cn

Expected result includes:

    uid: alice

    cn: Alice Admin

---

# Running the Full Deployment

Once:

- SSH works
- Inventory is correct
- Variables are configured
- Vault is available

run:

    ansible-playbook -i inventory site.yml --ask-vault-pass

If sudo requires a password:

    ansible-playbook -i inventory site.yml \
      --ask-vault-pass \
      -K

---

# What Happens During Deployment?

Ansible connects to the target server through SSH.

It then:

    SSH
     |
     v
    Install prerequisites
     |
     v
    Preconfigure slapd
     |
     v
    Install slapd
     |
     v
    Start slapd
     |
     v
    Generate TLS certificates
     |
     v
    Configure LDAPS
     |
     v
    Create LDAP OUs
     |
     v
    Create users
     |
     v
    Create groups
     |
     v
    Configure memberships
     |
     v
    Create service account
     |
     v
    Run verification

---

# Idempotency

One of Ansible's important properties is idempotency.

The goal is that running:

    ansible-playbook -i inventory site.yml

multiple times should not recreate existing resources unnecessarily.

For example:

    First run:
        changed > 0

    Second run:
        changed = 0
        failed = 0

The second run should report that the system is already in the
desired state.

This is important when using the project in real environments.

---

# Adding Another LDAP Server

Suppose a new machine has:

    IP: 10.9.99.92

Add it to:

    inventory

Example:

    [ldap_servers]
    ldap02 ansible_host=10.9.99.92

Test:

    ansible ldap02 -i inventory -m ping

Then deploy:

    ansible-playbook -i inventory site.yml --ask-vault-pass -K

---

# Fresh Server Procedure

For a completely fresh server, use this checklist.

## 1. Find the server IP

Example:

    10.9.99.92

## 2. Verify SSH

    ssh kali@10.9.99.92

## 3. Add it to inventory

    [ldap_servers]
    ldap02 ansible_host=10.9.99.92

## 4. Test Ansible

    ansible ldap02 -i inventory -m ping

## 5. Check Vault

    head -n 1 group_vars/ldap_servers/vault.yml

Expected:

    $ANSIBLE_VAULT;1.1;AES256

## 6. Run deployment

    ansible-playbook -i inventory site.yml \
      --ask-vault-pass \
      -K

## 7. Verify LDAP

    systemctl status slapd

## 8. Test LDAPS

    ldapsearch ...

## 9. Test user authentication

    ldapwhoami ...

---

# Troubleshooting

## Ansible cannot connect

Test:

    ssh user@SERVER_IP

Then:

    ansible ldap_servers -i inventory -m ping

Check:

- IP address
- SSH service
- username
- SSH key
- firewall
- sudo permissions

---

## Permission denied

If Ansible reports permission problems, verify that the playbook
is being run with:

    -K

and that the remote user has sudo access.

---

## LDAP service not running

Check:

    sudo systemctl status slapd

Logs:

    sudo journalctl -u slapd

---

## LDAP port 636 unavailable

Check:

    sudo ss -lntp | grep 636

Expected:

    LISTEN ... :636

Check:

    /etc/default/slapd

and verify that LDAPS is enabled.

---

## LDAP invalid credentials

Test directly:

    ldapwhoami -x \
      -H ldaps://ldap.example.internal:636 \
      -D "uid=alice,ou=People,dc=ldap,dc=example,dc=internal" \
      -W

If direct LDAP authentication fails, troubleshoot LDAP first.

If direct LDAP authentication works but an application fails,
troubleshoot the application configuration.

---

## TLS certificate verification failure

Check:

    openssl s_client \
      -connect ldap.example.internal:636 \
      -servername ldap.example.internal \
      -CAfile ca.crt

Verify that:

- The CA is correct
- The hostname matches the certificate
- The client trusts the CA
- DNS or `/etc/hosts` resolves the hostname to the correct server

---

# Important Hostname Requirement

The LDAP server certificate uses:

    ldap.example.internal

Therefore clients must resolve:

    ldap.example.internal

to the LDAP server's IP address.

For a lab environment this can be done using:

    /etc/hosts

Example:

    10.9.99.92 ldap.example.internal ldap02

For production environments, use proper DNS instead.

---

# Security Notes

## Do not commit plaintext passwords

Never commit:

- LDAP administrator passwords
- User passwords
- Service account passwords
- Private keys
- `.env` files
- Vault passwords

Use Ansible Vault for Ansible secrets.

---

## Protect the Vault password

The Ansible Vault password is separate from the encrypted Vault file.

Anyone who has both:

    vault.yml

and the:

    Vault password

can decrypt the secrets.

---

## Protect the CA private key

The LDAP CA private key is sensitive.

The CA private key should never be distributed unnecessarily.

---

# Current Test Environment

The project has been tested in a lab environment using:

    Ansible Controller:
        Kali Linux

    LDAP Server:
        Kali Linux

    LDAP software:
        OpenLDAP / slapd

    LDAP protocol:
        LDAP + LDAPS

    LDAP port:
        636

    Directory base:
        dc=ldap,dc=example,dc=internal

    LDAP hostname:
        ldap.example.internal

The environment is intended as an automation and learning lab.

---

# Current LDAP Objects

Users:

    alice
    bob

Groups:

    developers
    admins

Service account:

    django-bind

Organizational Units:

    ou=People
    ou=Groups
    ou=ServiceAccounts

---

# Future Improvements

Possible future improvements include:

- Automating /etc/hosts or DNS
- External DNS integration
- Production CA integration
- Certificate renewal
- LDAP user-management playbooks
- LDAP group-management playbooks
- User deletion
- User password rotation
- SSH public-key attributes
- POSIX groups
- SSSD integration
- Django LDAP integration
- FreeRADIUS integration
- Monitoring
- Backup and restore automation
- LDAP replication
- Multi-server deployment
- Production hardening
- Automated CI testing

---

# Learning Notes

This project is also intended to document the relationship between
Ansible and LDAP.

Ansible is responsible for:

    Installing
    Configuring
    Creating
    Modifying
    Verifying

LDAP is responsible for:

    Storing directory information
    Authenticating users
    Organizing users and groups
    Providing directory services

OpenLDAP is the LDAP server implementation.

Ansible does not replace LDAP.

Ansible automates the deployment and management of LDAP.

---

# Basic Command Reference

Check inventory:

    ansible-inventory -i inventory --graph

Ping all LDAP servers:

    ansible ldap_servers -i inventory -m ping

Run an ad-hoc command:

    ansible ldap_servers -i inventory -m command \
      -a "hostname"

Check service:

    ansible ldap_servers -i inventory -b \
      -m command \
      -a "systemctl status slapd"

Run playbook:

    ansible-playbook -i inventory site.yml

Run with Vault:

    ansible-playbook -i inventory site.yml \
      --ask-vault-pass

Run with Vault and sudo:

    ansible-playbook -i inventory site.yml \
      --ask-vault-pass \
      -K

Check syntax:

    ansible-playbook -i inventory site.yml --syntax-check

Dry-run:

    ansible-playbook -i inventory site.yml \
      --check \
      --ask-vault-pass

---

# Author / Project

LDAP Server Automation using Ansible.

Built as a cybersecurity infrastructure automation project
for learning and reusable lab deployment.
