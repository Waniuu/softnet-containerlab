# Containerlab Routing Lab (IPv4 & IPv6)

This repository contains a virtual networking laboratory built using **Docker** and **Containerlab**. The project demonstrates core networking concepts in Linux containers, including dual-stack routing (IPv4/IPv6), default gateway configuration, and dynamic script injection via Docker bind-mounts.

## 🏗️ Network Topology

The lab consists of three nodes: two end-hosts (`hs1`, `hs2`) and one central router (`rt1`).

```text
       Subnet 1 (10.0.1.0/24)                 Subnet 2 (10.0.2.0/24)
       IPv6 (fd00:1::/64)                     IPv6 (fd00:2::/64)
  ┌──────────┐              ┌───────────┐              ┌──────────┐
  │          │ eth1    eth1 │           │ eth2    eth1 │          │
  │   hs1    ├──────────────┤    rt1    ├──────────────┤   hs2    │
  │          │ .2        .1 │ (Router)  │ .1        .2 │          │
  └──────────┘              └───────────┘              └──────────┘

# 🚀 Key Features Demonstrated

    Bind-Mounts instead of COPY: The initialization script (entrypoint.sh) is not baked into the Docker image. Instead, it is dynamically mounted into the containers at runtime using the binds directive in the topology file. This allows for rapid script iteration without rebuilding the image.

    Dual-Stack IP Forwarding: The router (rt1) has both IPv4 and IPv6 forwarding enabled via sysctl commands, allowing traffic to flow seamlessly between the two distinct subnets.

    Default Gateway Configuration: The end-hosts are completely unaware of the broader network topology. They are configured with simple default routes pointing to rt1, satisfying the requirement that hosts must not know any route other than their own subnet.

📁 Repository Structure

    basic-lab.clab.yml
    The core Containerlab topology definition. It defines the nodes, the Docker image to use, the physical links (virtual wires) between interfaces, the bind-mounts, and the post-startup execution commands (like ip route add default and sysctl).

    Dockerfile
    Custom image definition. Based on ubuntu:24.04, modified to install essential networking tools (iproute2, iputils-ping) required for the routing to function properly.

    configs/
    Environment variables for nodes. Contains hs1.cfg, hs2.cfg, and rt1.cfg. These files store IP addresses, prefixes, and peer IPs. They are sourced by the entrypoint script.

    bin/entrypoint.sh
    The startup script. It reads the .cfg files, brings up the interfaces, assigns the correct IPv4/IPv6 addresses, and tests basic neighbor reachability.

    deploy.sh / destroy.sh
    Automation scripts. Wrappers for Containerlab commands to easily deploy the lab and tear it down.

🛠️ How to Run & Test
1. Deploy the lab

Ensure you have Docker and Containerlab installed. Run the deployment script:
Bash

sudo ./deploy.sh

2. Verify IPv4 Traffic (Bidirectional)

Hosts can reach each other in both directions.
Bash

# Test hs1 -> hs2
sudo docker exec -it clab-basic-lab-hs1 ping -c 2 10.0.2.2

# Test hs2 -> hs1
sudo docker exec -it clab-basic-lab-hs2 ping -c 2 10.0.1.2

Expected result: 0% packet loss for both commands.
3. Verify IPv6 Traffic (Bidirectional)

The network also fully supports IPv6 routing.
Bash

# Test hs1 -> hs2 (IPv6)
sudo docker exec -it clab-basic-lab-hs1 ping6 -c 2 fd00:2::2

# Test hs2 -> hs1 (IPv6)
sudo docker exec -it clab-basic-lab-hs2 ping6 -c 2 fd00:1::2

Expected result: 0% packet loss for both commands.
4. Teardown

To clean up the environment:
Bash

sudo containerlab destroy -t basic-lab.clab.yml --cleanup

