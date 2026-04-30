# Born2beroot – DevDoc

## 1. Project goal

Born2beroot is a system administration project where the objective is to build a secure Linux server and explain every decision behind it.

The documentation should show:

* the analysis before starting
* the reasoning behind each technical choice
* the order in which the system was built
* the final result and why it is valid

---

## 2. Starting analysis

The first step is understanding the subject and identifying the constraints.

The first decision is whether the project will include bonus features or stay mandatory only.

This matters because the VM planning changes depending on that choice.

If bonus is planned, the machine needs more space for extra services, storage, and flexibility.

If bonus is not planned, the VM can stay smaller and focused only on the mandatory requirements.

The system must be:

* inside a virtual machine
* minimal and server-oriented
* protected by proper access rules
* built with LVM
* monitored with a script

From this, the design becomes clearer.

The project is not just “install Linux”.

It is “design a secure server with controlled behavior”.

---

## 3. Why the VM comes first

The virtual machine is the starting point because it defines the environment where everything else happens.

It is chosen because it gives:

* isolation from the host
* reproducibility
* easy rollback if a mistake happens

This is useful for experimentation, but also for evaluation.

The VM size is not the same in every case.

It should be planned according to the decision made at the start:

* mandatory only = smaller and simpler VM
* mandatory + bonus = bigger VM with more room

---

## 4. Why a minimal system is better

A minimal Linux installation is preferred because unnecessary software makes the system more complex.

The more software is installed, the more:

* services may run in the background
* ports may be open
* maintenance becomes harder

So the minimal approach reduces noise and improves clarity.

---

## 5. Storage analysis and LVM choice

LVM is one of the most important architectural decisions.

The idea is to separate physical storage management from logical usage.

The chain is:

Physical Volume → Volume Group → Logical Volume

### Why this is the correct choice

It makes storage more flexible than static partitions.

It also allows better planning for:

* root system files
* user data
* logs
* swap

### Why this matters in practice

If the system grows or needs adjustments, LVM gives more room to adapt.

That is why it is a stronger design choice than a fixed layout.

---

## 6. Service and boot reasoning

During boot, only the necessary services should be enabled.

The reason is simple:

* every service increases system complexity
* every service can become a security concern
* every service must have a clear purpose

This is why the system should start in a predictable state.

### Bonus services

If bonus services are added, they should be treated separately from the mandatory setup.

They usually require:

* more planning
* more resources
* more firewall rules
* more access control decisions

That is why bonus services should only be introduced after the mandatory system is stable.

---

## 7. SSH reasoning

SSH is used because remote access is part of server administration.

However, SSH must be restricted carefully.

The design choices here are:

* allow authentication
* forbid direct root login
* use the required port configuration

This keeps access useful without making the machine too exposed.

---

## 8. Firewall reasoning

The firewall is configured with a deny-by-default mindset.

That means:

* only required services are allowed
* all unnecessary traffic is blocked
* the configuration stays easy to audit

The logic is that the safest port is the one that is not open.

---

## 9. User model and sudo logic

The project should not rely on root for daily administration.

Instead, a regular user is created and given limited sudo access.

This is better because:

* actions are traceable
* permissions are controlled
* mistakes are less dangerous

It also matches the principle of least privilege.

---

## 10. Authentication and password policy

Password policy is part of the security design, not an afterthought.

It exists to enforce:

* stronger credentials
* periodic changes when needed
* fewer brute-force opportunities

The goal is to make weak authentication harder.

---

## 11. Logging and traceability

Logs are necessary because they show what happened on the system.

They help to:

* understand user actions
* diagnose failures
* verify system behavior

Good logs support both troubleshooting and accountability.

---

## 12. Monitoring script reasoning

The monitoring script is used to summarize the machine state quickly.

It can include:

* CPU usage
* memory usage
* disk usage
* user count
* network information

### Why it exists

It shows whether the system is healthy and gives a fast overview during evaluation.

### What makes it good

* simple output
* accurate values
* low overhead
* easy to read

---

## 13. Full project flow

The project can be explained as a sequence:

1. analyze the subject and constraints
2. choose the VM setup
3. plan storage with LVM
4. install a minimal system
5. configure users and sudo
6. configure SSH
7. configure the firewall
8. enforce password policy
9. verify logging
10. create the monitoring script
11. test everything
12. explain why each decision was taken

This is the cleanest way to present the process from beginning to end.

---

## 14. How to explain the project clearly

When presenting the work, focus on three things:

* what was needed
* why each choice was made
* how the final system matches the requirements

That shows real understanding instead of just memorization.

---

## 15. Final conclusion

Born2beroot is a strong system administration project because it teaches how to build a secure Linux server and how to justify the design behind it.

The important part is not only that it works.

The important part is that the whole process can be analyzed, explained, and defended.
