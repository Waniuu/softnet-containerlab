# Containerlab Routing & Stateful Firewall Lab

This repository contains a virtual networking laboratory built using **Docker** and **Containerlab**. The project demonstrates advanced networking concepts in Linux containers, including static routing, stateful firewall configurations, and dynamic script injection via Docker bind-mounts.

## 🏗️ Network Topology

The lab consists of three nodes: two end-hosts (`hs1`, `hs2`) and one central router (`rt1`).

```text
       Subnet 1 (10.0.1.0/24)                 Subnet 2 (10.0.2.0/24)
  ┌──────────┐              ┌───────────┐              ┌──────────┐
  │          │ eth1    eth1 │           │ eth2    eth1 │          │
  │   hs1    ├──────────────┤    rt1    ├──────────────┤   hs2    │
  │          │ .2        .1 │ (Router)  │ .1        .2 │          │
  └──────────┘              └───────────┘              └──────────┘

Key Features Demonstrated

    Bind-Mounts instead of COPY: The initialization script (entrypoint.sh) is not baked into the Docker image. Instead, it is dynamically mounted into the containers at runtime using the binds directive in the topology file. This allows for rapid script iteration without rebuilding the image.

    Static Routing: The end-hosts do not have a default gateway for the entire network. Specific static routes are injected via the exec command to instruct them on how to reach remote subnets via the rt1 router.

    Stateful Firewall (iptables):
    The router (rt1) has IPv4 forwarding enabled and uses iptables to enforce strict traffic rules:

        hs1 can initiate traffic (e.g., ICMP Ping) to hs2.

        hs2 can reply to established connections.

        hs2 is blocked from initiating new connections to hs1 (simulating a NAT or public-facing perimeter).

📁 Repository Structure

    basic-lab.clab.yml
    The core Containerlab topology definition. It defines the nodes, the Docker image to use, the physical links (virtual wires) between interfaces, the bind-mounts, and the post-startup execution commands (like ip route add and iptables).

    Dockerfile
    Custom image definition. Based on ubuntu:24.04, modified to install essential networking tools (iproute2, iputils-ping, and iptables) required for the routing and firewall rules to function properly.

    configs/
    Environment variables for nodes. Contains hs1.cfg, hs2.cfg, and rt1.cfg. These files store IP addresses, prefixes, and peer IPs. They are sourced by the entrypoint script.

    bin/entrypoint.sh
    The startup script. It reads the .cfg files, brings up the eth1 interfaces, assigns the correct IPv4/IPv6 addresses, and tests basic neighbor reachability.

    deploy.sh / destroy.sh
    Automation scripts. Wrappers for Containerlab commands to easily build the image, deploy the lab, and tear it down.

🛠️ How to Run & Test
1. Deploy the lab

Ensure you have Docker and Containerlab installed. Run the deployment script:
Bash

sudo ./deploy.sh

2. Verify Allowed Traffic (Hs1 -> Hs2)

From hs1, ping hs2. This traffic is permitted by the firewall.
Bash

sudo docker exec -it clab-basic-lab-hs1 ping -c 4 10.0.2.2

Expected result: 0% packet loss.
3. Verify Blocked Traffic (Hs2 -> Hs1)

From hs2, try to ping hs1. This traffic is dropped by the stateful firewall rules on rt1.
Bash

sudo docker exec -it clab-basic-lab-hs2 ping -c 4 10.0.1.2

Expected result: 100% packet loss.
4. Teardown

To clean up the environment:
Bash

sudo containerlab destroy -t basic-lab.clab.yml --cleanup

