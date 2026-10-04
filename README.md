# Hospital Network Architecture

This document outlines the network and system architecture for the hospital, including the deployment of key services, security measures, and hardware configuration.

## **System Overview**

The proposed network architecture is designed to provide a secure, efficient, and scalable infrastructure tailored for a hospital environment. It uses VLAN segmentation to optimize traffic and isolate critical resources, with dedicated networks for servers, staff, visiting doctors, and wireless access points. A pfSense firewall ensures secure external communication and mitigates threats.

As the network and system administrator, I managed the firewall and servers, implemented access control, safeguarded sensitive data, and monitored network traffic. The design includes centralized backup systems and Active Directory for data integrity and operational continuity, ensuring compliance with healthcare IT standards.

The architecture follows a hybrid design, hosting application services in the cloud while maintaining on-premises database storage for enhanced security and reliability.


## Network design
<p align="center">
  <img src="Network%20Design.jpg" width=75%>
</p>

___

## Network Segmentations
* Staff Network | VLAN 17 | Network : 192.168.17.0/24
* Visiting Doctor's Network | VLAN 27 | Network : 192.168.27.0/24
* Access Point Network | VLAN 47 | Network : 192.168.47.0/24
* Server Network | VLAN 37 | Network : 192.168.37.0/24

## Hardware & Software
### Swtich
  * Model : Cicso IOU

In the hospital network, managed switches with VLAN segmentation are used to optimize network performance and manage traffic efficiently. By dividing the network into separate VLANs for different departments or devices, the network can reduce congestion, improve bandwidth utilization and ensure smoother communication.

This segmentation isolates traffic, reducing disruptions and creating a more organized and reliable infrastructure. While not directly focused on security, VLANs help prioritize critical traffic and prevent bottlenecks, making the network more efficient and better suited for the hospital's operational needs.

### Servers
* Operatiting System: Windows Server 2019
  * User Management: Managed through the Active Directory Domain Controller, this service handles the authentication and authorization of users, ensuring centralized management of user accounts, permissions, and policies across the network.

  * Database: Managed by SQL Server, this server is responsible for hosting the hospital's critical databases, ensuring that the data is stored securely and efficiently, and providing fast access for authorized users.

  * Backup: Handled by Veeam backup software, this service ensures the hospital's data is regularly backed up and can be restored in case of a failure or disaster, providing an additional layer of data security and business continuity.

Each service is running on a separate hardware instance to ensure optimal performance, reliability, and security.

### Firewall
  * **pfSense** is deployed to manage network traffic and ensure security across the internal and external communications.

___

## Firewall Configuration Goals
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/b/b9/PfSense_logo.png/799px-PfSense_logo.png" width=30%>

1. Traffic Isolation

To keep hospital network traffic separate from the wireless access point (AP) traffic, we can use VLANs to create separate networks for each. This ensures that traffic from the hospital’s internal network doesn't mix with traffic from the visitors or patients using the Wi-Fi. The pfSense firewall helps control this separation by blocking unnecessary communication between the networks. The Wi-Fi for visitors and patients is set up on its own network, keeping the hospital's systems secure and running smoothly.

#### Steps to isolate traffic: in this scenario traffic from wifi network has to isolated from internal LAN network.
- By leveraging Rules in the pfSense firewall, we can easily isolate traffic by selecting the respective interface, setting the action to Block, and specifying the destination as the network that should be isolated from the respective interface.
- Since there are multiple VLAN networks in the respective infrastructure, I am grouping all internal network using `Aliases`.
<p align="center">
  <img src="PFsense%20Screenshots/ISOLATION/ISO%20-%201.png" width=80%>
  <img src="PFsense%20Screenshots/ISOLATION/ISO%20-%203.png" width=80%>
  <img src="PFsense%20Screenshots/ISOLATION/ISO%20-%202.png" width=80%>
</p>

2. Bandwith Control

For bandwidth control, we can use the pfSense firewall to manage traffic by setting specific rules that prioritize hospital network traffic over Wi-Fi traffic. By allocating dedicated bandwidth for the hospital's internal network and the visitors' Wi-Fi, we ensure the critical applications and systems receive the necessary resources. This prevents network congestion and ensures optimal performance, while keeping the traffic separate to maintain security and smooth operations.

#### Steps to control Bandwidth:
- Initla step to control bandwidth is creating `Traffic Limiter`, can be find from `Firewall ---> Traffic Shaper ---> Limiters.
- After creating Limites, set the limiters `In/Out pipe` from firewall rules.

<p align="center">
  <img src="PFsense%20Screenshots/BW%20Control/Step%20-%201.png" width=80%>
  <img src="PFsense%20Screenshots/BW%20Control/Step%20-%203.png" width=80%>
  <img src="PFsense%20Screenshots/BW%20Control/Step%20-%204.png" width=80%>
  <img src="PFsense%20Screenshots/BW%20Control/Step%20-%205.png" width=80%>
</p>

3. Traffic Filtering
For traffic filtering, since the database server is hosted on-premises and the application server is in the cloud, we want to ensure secure communication between the two while preventing malicious connections. To achieve this, we are using pfBlockerNG in pfSense. This tool helps block unwanted or harmful IP addresses, domains, or networks from accessing the servers, thereby preventing potential threats. It enhances the firewall's capabilities by filtering out malicious traffic, ensuring that only legitimate connections are allowed, protecting both the database and application servers from external attacks.

<img src="https://c10.patreonusercontent.com/4/patreon-media/p/campaign/1320964/1d0a355ee00a477dbeb8f7db0d276fb2/eyJoIjoxMDgwLCJ3IjoxMDgwfQ%3D%3D/1.png?token-time=1738368000&token-hash=T9JIowCaLQmzVkMIGIeWoUtG5SaUx7RXXFj9d8wFHJA%3D" width=10%>

#### Steps to set up _pfBlockerNG_
- Install pfBlockerNG by Going to System ---> Package Manager ---> Available Packages and install pfBlockerNG.
- Enable pfBlockerNG in Firewall ---> pfBlockerNG and adjust logging settings.
- Add IP block lists or DNS block lists in the IP and DNSBL tabs. And configure custom or public threat lists to block malicious sources.
- If needed, enable GeoIP blocking in the GeoIP tab and select countries to block or allow.
- Go to Firewall > Rules to define custom rules that control traffic inbound for database server.

<p align="center">
  <img src="PFsense%20Screenshots/Filtering/pfngblocker.png" width=80%>
</p>

___

## Veeam Backup Configuration and Disaster Recovery Simulation
<img src="https://logos-world.net/wp-content/uploads/2024/07/Veeam-Logo.jpg" width=30%>
1. Veeam Backup Configuration
For ensuring the integrity and availability of the hospital's data, Veeam Backup is used to back up critical data, including patient records and application data, to safeguard against potential data loss. By creating backup jobs in Veeam, regular snapshots of the on-premises server, which stores the database, are made. This enables us to quickly recover from any disaster scenario and ensure minimal downtime.

#### Steps to Configure Veeam Backup:
- Install Veeam Backup & Replication on a dedicated Windows Server.
- Add the on-premises server as a backup target in Veeam.
- Create backup jobs for the hospital's critical data, such as the database and Active Directory, to ensure regular snapshots.
- Schedule backup jobs to run at predefined intervals (e.g., daily, weekly) to maintain up-to-date backups.
- Test backup integrity regularly to ensure that the backup process is functioning correctly and that data can be restored.

<p align="center">
  <img src="Backup%20Screenshots/backup_conf/Step%20-%201.png" width=90%>
  <img src="Backup%20Screenshots/backup_conf/Step%20-%202.png" width=90%>
  <img src="Backup%20Screenshots/backup_conf/Step%20-%203.png" width=30%>
  <img src="Backup%20Screenshots/backup_conf/Step%20-%204.png" width=30%>
  <img src="Backup%20Screenshots/backup_conf/Step%20-%205.png" width=30%>
  <img src="Backup%20Screenshots/backup_conf/Step%20-%205.5.png" width=30%>
  <img src="Backup%20Screenshots/backup_conf/Step%20-%206.png" width=30%>
  <img src="Backup%20Screenshots/backup_conf/Step%20-%207.png" width=30%>
  <img src="Backup%20Screenshots/backup_conf/Step%20-%208.png" width=30%>
  <img src="Backup%20Screenshots/backup_conf/Step%20-%209.png" width=30%>
  <img src="Backup%20Screenshots/backup_conf/Step%20-%2010.png" width=30%>
  <img src="Backup%20Screenshots/backup_conf/Step%20-%2011.png" width=30%>
  <img src="Backup%20Screenshots/backup_conf/Step%20-%2012.png" width=30%>
  <img src="Backup%20Screenshots/backup_conf/Step%20-%2013.png" width=30%>
  <img src="Backup%20Screenshots/backup_conf/Step%20-%2014.png" width=30%>
  <img src="Backup%20Screenshots/backup_conf/Step%20-%2017.png" width=60%>
</p>

2. Simulating a disaster scenario when the database files are deleted. But since all the files are backed up it is easy to reduce the downtime by just restore the files from Backup Server
- Open the Veeam Backup & Replication Console and navigate to the Backups view.
- Find the backup that contains the files you want to restore. And right-click the backup and select Restore guest files.
- Select the restore point and specify a reason for the restore and browse to the file you want to restore.
- Right-click the file and select Restore to overwrite the original file, or use Copy to to save it to a new location.

<p align="center">
  <img src="Backup%20Screenshots/restore_backup/R%20Step%20-%201.png" width=80%>
  <img src="Backup%20Screenshots/restore_backup/R%20Step%20-%202.png" width=40%>
  <img src="Backup%20Screenshots/restore_backup/R%20Step%20-%203.png" width=40%>
  <img src="Backup%20Screenshots/restore_backup/R%20Step%20-%204.png" width=40%>
  <img src="Backup%20Screenshots/restore_backup/R%20Step%20-%205.png" width=40%>
  <img src="Backup%20Screenshots/restore_backup/R%20Step%20-%206.png" width=40%>
  <img src="Backup%20Screenshots/restore_backup/R%20Step%20-%207.png" width=40%>
  <img src="Backup%20Screenshots/restore_backup/R%20Step%20-%208.png" width=40%>
  <img src="Backup%20Screenshots/restore_backup/R%20Step%20-%209.png" width=40%>

</p>


___

___
