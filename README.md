# rhel9-centralized-logging-resilient-storage
Centralized logging architecture built on top of a bastion-based administrative foundation. This project focuses on secure log transport, storage failure-domain isolation, and live LVM migration following a controlled disk exhaustion incident.

This repository extends the platform established in:

**RHEL-9-Bastion-Foundation-Lab**  https://github.com/HeltonDG/RHEL-9-Bastion-Foundation-Lab
The logging infrastructure extends that environment without altering its administrative control model.

Rather than redesigning the network, this project enhances the environment by introducing a dedicated logging server (`log01`) and validating system behavior under storage pressure.

---

## Project Overview

This project implements centralized logging using **rsyslog over TCP (514)** to ensure reliable log delivery while preventing packet loss associated with UDP transport.

During stress testing, uncontrolled log growth exhausted the root filesystem of the logging server. The system was then **re-architected live** to isolate `/var/log` onto a dedicated logical volume, significantly improving system survivability.

The result is a logging platform that better reflects production infrastructure patterns.

---

## Architecture Overview

This design introduces a dedicated observability tier while preserving bastion-mediated administration.

```text
                        Administrative Access Path
Remote Admin -> VPN -> Bastion (jump01) -> Internal Network

                        Internal Network (10.10.20.0/24)
        ---------------------------------------------------
        |                                                 |
     srv01 ------------------------------------> log01
   (log forwarder)                          (central collector)
                                             TCP/514
