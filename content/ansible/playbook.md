---
title: "Playbook"
date: 2018-09-10T13:30:00+02:00
order: 2
---

# Playbook

Playbook are another way to define execution of commands on hosts and the principal way of executing `Ansible` on your infrastructure.  
They defines the configuration to be executed and manage on your machines.  

Principal diff with simple command execution are :

 * Templating
 * Idempotency
 * Reusability

These 3 features are really interesting for managing cofigruation of your infrastructure, coupled with the `ansible` behavior of push instead of pull.

# Introduction

Follow the official introduction to Playbook [here](https://docs.ansible.com/ansible/2.6/user_guide/playbooks_intro.html)

# Exercice

Create a VM and install nginx on it with a custom user .  
You should restart multiple times the provisionning with `vagrant up --provision` and only install the first time.