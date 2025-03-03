# Setting Up A Remote Lab

## Introduction

### What are remote labs?
Remote labs are a LAVA development allowing CI with a single centralised dashboard server and multiple distributed board farms - some of which can be in hardware vendor or other partner premises. the result is a large distributed board farm with LAVA testing invoked and reported via a central LAVA server.

Remote labs provide the following benefits:
 * DUT management in the hands of originators/owners 
 * No need to educate the central lab personnel on every board from every vendor
 * Minimise hardware shipping/return cycle with problematic board debug
 * Can reach “consortium scale” with no single bottleneck/resource sink

See [1] for more information.

### About this guide
This document outlines the requirements and process needed to deploy a remote lab that will connect to a master LAVA instance managed by Linaro. 

### Reasons to set up a remote lab
Remote labs are intended to support partners who have or expect a long-term relationship with an established project (e.g. LKFT, trusted Firmware) and are who looking to increase testing on a specific supported devices where they have interest and/or expertise. For example, various Linaro Members are working toward setting up a their own remote labs that will connect to Linaro's LKFT master LAVA server and increase the number of tests and configurations for devices they are interested in.

### Out of scope for remote labs
Remote labs are an efficient way to scale board farms. They are not intended as a way to develop device or new feature support. The following are contra-indications for working as a remote lab:
* Using a custom version of LAVA
* Working with devices-under-test (DUTs) that are not supported upstream in LAVA
* Using an installation as a testbed to upstream integration support for new devices

### Assumptions
* Contact with Linaro Lab team has been established
* Ability to carry out a native install of LAVA components on local hardware running Debian Stretch
* Power and USB control scripts can be tested outside of LAVA to confirm that they are working correctly
* There is a supported communication path between the remote lab and the central server (see below for "Proving the Connection")

### Other Documentation
The comprehensive LAVA documentation [2] also provides documentation which covers installing and configuring LAVA components.

## Set Up Tasks

## Infrastructure requirements
* There are minimum requirements (CPU, memory, I/O robustness) for the hardware that hosts the Worker instance.
The typical guidance for the CPU is that it should have more available cores than the number of devices connected to the worker.
As to RAM, a rough guide would be 8GB + 1GB per device over 8, in multiples of 8
Hard drives are less problematic for workers. They are effectively scratch space, but reasonable performance and capacity is advised. 256GB is usually more than adequate. A 1Gb LAN connection is preferred.
* DUT interface hardware (USB, power control, relay control) needs to be robust
The Lab team have recommendations for PDU, USB Hubs anc Relay control based on our own experience. Scripts supporting our preferred vendors are available in the lava-lab repo (https://git.linaro.org/lava/lava-lab.git/) specifically in the shared/lab-scripts sub-ditectory.
* All infrastructure control should be via scripts invoked in device dictionary entries rather than any temptation towards dispatcher hard coding
* For a many-Worker Remote Lab, scalable administration, deployment & configuration tools should be used
The Cambridge Lab uses "salt" as the admin tool, and the lava-lab repo contains all the salt configuration which can act as a guide.

### Proving the Connection
Whilst the LAVA dispatcher's communication with the central server is based on standard protocols, and all communication originates at the dispatcher, It's not possible to guarantee that a remote LAVA dispatcher will be able to communicate with the central server in all possible situations. This can be because of corporate or other firewalls and other infrastructure constraints.
In order to test the connection, Linaro will supply a containerised test dispatcher which should be run at the remote site to prove the connection capability.
If the connection test is not successful, it will be necessary either to debug the corporate firewall infrastructure, or, more likely, install a dedicated external connection for the remote lab.

### Installing lava-dispatcher
This installation assumes a native install on Debian Stretch

```
# Activate the backports
apt-get update
apt-get install --no-install-recommends --yes wget gnupg ca-certificates apt-transport-https
echo "deb https://deb.debian.org/debian stretch-backports main" > /etc/apt/sources.list.d/backports.list

# Add the lava debian repository
wget https://apt.lavasoftware.org/lavasoftware.key.asc
apt-key add lavasoftware.key.asc
echo "deb http://apt.lavasoftware.org/release stretch-backports main" > /etc/apt/sources.list.d/lava.list

# Install lava-dispatcher
apt-get update
apt-get install -t stretch-backports lava-dispatcher

```

### Connecting the dispatcher
Connecting the remote dispatcher to the central server is the final step and key piece of setting up a remote lab. The dispatcher which is to be connected can be installed stand-alone as in the previous section, or it can be part of a full LAVA install. 
It is likely that some of the steps below will be scripted or supported in terms of Linaro supplying a complete lava-slave configuration file. However, because Linaro does not have admin access to the remote lab dispatcher, the actual configuration will need to be actioned by remote lab staff. 

#### Connection steps
1. *(Do we need to stop the lava-slave service first?)*
1. Create an encryption certificate:
/usr/share/lava-dispatcher/create_certificate.py <dispatcher-name>
1. Send the public key (to LAB admin
/etc/lava-dispatcher/certificates.d/<dispatcher-name>.key
1. Install the master certificate:
cp master.key /etc/lava-dispatcher/certificates.d/master.key
1. Configure the slave by editing /etc/lava-dispatcher/lava-slave and uncommenting the variables you want to define:
MASTER_URL=<master_url>
LOGGER_URL=<logger_url>
ENCRYPT="--encrypt"
MASTER_CERT="--master-cert /etc/lava-dispatcher/certificates.d/master.key"
SLAVE_CERT=--slave-cert /etc/lava-dispatcher/certificates.d/<dispatcher-name>.key_secret
HOSTNAME="--hostname <dispatcher-name>"
1. Start lava-slave service
service lava-slave start
1. Watch the logs in /var/log/lava-dispatcher/lava-slave.log
1. Check that the worker is online in the lava web interface.

## Maintenance
 
### Point-of-Contact/Support
The Linaro Lab team need contact details for the primary point-of-contact person for the remote lab and another name for illness/vacation contact.
A mailing list has been created specifically for remote lab users to distribute information and for posting support questions. 
*(needs details on how to join the mailing list)*

### Upgrades
It's assumed that a remote lab will closely follow the central server in terms of LAVA version upgrades. Because Linaro does not have admin access to the remote lab dispatcher, the actual upgrade will need to be actioned by remote lab staff. The remote lab point-of-contact person will be contacted by the Linaro Lab team when an upgrade is planned. 
The LAVA dispatcher does not store any state and all local DUT configuration should be stored in Device dictionary entries.
It's assumed that the upgrade of the remote lab will be done in a timely manner. A mismatch of versions between the remote and the server is not a supported configuration. 

### Downgrades
In the event of a problem with an upgrade, a temporary downgrade is possible.

## References
[1] *(link to the PDF of the remote lab slide deck)*

[2] LAVA documentation *what's the official site?*
