# Architecture

## Purpose

This architecture introduces centralized logging into an existing bastion-based administrative environment while preserving strict access control, network segmentation, and least-privilege principles.

Rather than redesign the foundation, the logging capability was layered onto the platform to maintain architectural stability and reinforce controlled operational pathways.

The result is an infrastructure pattern that emphasizes survivability, observability, and predictable system behavior under stress.

---

## Design Goals

- Centralize system telemetry for audit and operational visibility  
- Ensure reliable log transport  
- Prevent logging workloads from impacting operating system stability  
- Maintain bastion-mediated administration  
- Preserve internal network isolation  
- Enforce default-deny security posture  
- Operate with SELinux in enforcing mode  

---

## High-Level Topology

Administrative access remains unchanged from the foundation architecture:

Remote Administrator → VPN → Bastion → Internal Network

Within the internal network, a dedicated logging server aggregates telemetry from participating systems.

```text
Administrative Access Path:
Remote Admin -> VPN -> jump01 -> Internal Network

Internal Network (10.10.20.0/24)
--------------------------------
srv01  ----TCP/514---->  log01
 (forwarder)              (collector)
