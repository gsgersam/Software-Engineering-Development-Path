# Born2beroot

## Overview

Born2beroot is a system administration project focused on building a secure Linux server inside a virtual machine.

The important part is not only that the system works, but that every decision can be explained:

* what was needed
* why a specific choice was made
* how that choice affects security and maintainability
* what the result looks like at the end

---

## Project analysis

Before doing anything, the project has to be analyzed as a whole.

The main questions are:

* Will the project stay mandatory only, or include bonus features?
* What kind of machine is required?
* Which services are really necessary?
* How should the storage be structured?
* How can access be secured?
* How will the system be monitored?

This analysis is important because the project is not about random installation steps.

It is about building a controlled server with clear decisions.

The first decision is whether to do bonus or not, because that changes the whole design.

If bonus is planned, the VM should be sized with more room for services, storage, and future growth.

If there is no bonus, the VM can stay smaller and simpler, because only the mandatory requirements need to fit.

---

## Why the project is designed this way

The system is intentionally minimal because a smaller system is easier to secure, easier to maintain, and easier to defend during evaluation.

That leads to three main priorities:

* reduce unnecessary services
* restrict access as much as possible
* keep the architecture understandable

The final system must be practical, but also explainable.

---

## Main decisions and why they were taken

### Virtual machine

A virtual machine is used because it isolates the project from the host system.

This gives:

* a safe environment for testing
* easier recovery if something breaks
* reproducible results

The VM size should match the plan from the beginning.

If bonus is included, the machine should be larger so it can handle the extra services and storage requirements.

If bonus is not included, a smaller VM is enough and keeps the setup simpler.

### Minimal Linux server

A server-style installation is chosen instead of a desktop environment.

This is better because:

* it uses fewer resources
* it removes unnecessary software
* it reduces the attack surface

### LVM storage

LVM is used because simple fixed partitions are too rigid for this kind of project.

LVM helps to:

* organize storage more cleanly
* separate system areas logically
* allow future resizing if needed

### SSH access

SSH is used for remote access, but it is restricted.

The reason is simple: remote access is useful, but it must not become an open door.

### Firewall

The firewall is configured to allow only required traffic.

This is done because every open port is a possible risk.

### Bonus services

If bonus features are included, they must be treated as a separate service layer.

This is important because bonus services:

* add extra network exposure
* increase system complexity
* require additional firewall and access decisions

The bonus part should only be added after the mandatory system is stable and understood.

### User and sudo policy

Root should not be used for daily work.

Administration is done through a regular user with controlled sudo permissions because this creates accountability and limits mistakes.

### Monitoring script

The monitoring script exists to give a fast overview of the system state.

It proves that the administrator can read the system, not just install it.

---

## What the project shows

This project demonstrates that I understand:

* Linux server basics
* disk and partition planning
* access control
* service restriction
* password policy
* logging and monitoring

It also shows that I can justify technical decisions instead of just following commands.

---

## Final result

At the end, the machine should be:

* secure
* minimal
* stable
* easy to explain
* easy to evaluate

Born2beroot is successful when the setup is not only functional, but logically well designed from beginning to end.
