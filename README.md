<h1>Ticket 005 — DNS Misconfiguration and Active Directory Connectivity Troubleshooting</h1>

<b>1. Scenario</b>

A simulated IT support ticket was created to investigate a Windows 11 workstation experiencing DNS resolution failures when attempting to locate resources within the jakehomelab.test Active Directory domain.

The lab environment consisted of a Windows Server domain controller (JH-DC01) and a domain-joined Windows 11 client (MAN-PC-001). The objective was to introduce a DNS configuration fault, identify the root cause, implement a fix, and verify that domain services were accessible again.

<b>2. Environment</b>

- Domain: jakehomelab.test

- Domain controller: JH-DC01

- Client workstation: MAN-PC-001

- Client operating system: Windows 11

- Domain controller IP address: 192.168.193.10

- Incorrect DNS server: 192.0.2.53

<b>3. Procedure</b>

<b>Step 1 — Record the original network configuration</b>

The ipconfig /all command was executed on the client workstation to record its existing network configuration, including its IPv4 address, subnet mask, DHCP settings, DNS suffix, and DNS server.

<img src="Screenshots/Original ipconfig.png"/>


The original configuration confirmed that the workstation was using 192.168.193.10 as its DNS server. This provided a reference point for the troubleshooting exercise.

<b>Step 2 — Introduce the DNS fault</b>

To simulate a real-world configuration error, the preferred DNS server was manually changed to 192.0.2.53 through the network adapter's IPv4 properties.

<img src="Screenshots/Incorrect DNS settings.png"/>


The client retained its automatically assigned IP configuration, ensuring that the test isolated DNS configuration rather than introducing an unrelated IP addressing problem.

<b>Step 3 — Investigate the connectivity issue</b>

Several command-line tools were used to identify the cause of the failure.

- ping 192.168.193.10 confirmed that the client could communicate with the domain controller by IP address, with no packet loss.

- nslookup jakehomelab.test failed because the configured DNS server did not respond.

- nslookup -type=SRV _ldap._tcp.dc._msdcs.jakehomelab.test also failed, preventing the client from retrieving the Active Directory service-location record through the incorrect DNS server.

- nltest /dsgetdc:jakehomelab.test successfully discovered the domain controller, demonstrating that the discovery command could still succeed despite the failed direct DNS queries.

 <img src="Screenshots/Before fix evidence.png"/>


These results indicated that basic IP connectivity was available, while DNS resolution through the configured server was failing.

<b>Step 4 — Identify the root cause</b>

The IPv4 configuration was inspected and the incorrect preferred DNS server, 192.0.2.53, was identified.

The issue was caused by the client querying an incorrect DNS server instead of the DNS service hosted by the lab domain controller. This prevented the affected DNS queries from resolving the domain and its Active Directory service records.

<b>Step 5 — Restore the correct DNS configuration</b>

The preferred DNS server was changed back to 192.168.193.10, the IP address of JH-DC01.

The workstation's IP address configuration was left unchanged to avoid introducing additional network issues.

 <img src="Screenshots/Corrected DNS settings.png"/>


<b>Step 6 — Verify the resolution</b>

After restoring the DNS configuration, the following commands were executed:

- nslookup jakehomelab.test — successfully resolved the domain to 192.168.193.10.

- nslookup -type=SRV _ldap._tcp.dc._msdcs.jakehomelab.test — successfully returned the LDAP service-location record for the domain controller.

- nltest /dsgetdc:jakehomelab.test — successfully discovered JH-DC01.

- gpupdate /force — confirmed that the computer Group Policy update completed successfully.

 <img src="Screenshots/After fix evidence.png"/>


The successful results confirmed that DNS resolution and the tested domain services were functioning again.

<b>4. Root Cause</b>

The Windows 11 client was configured to use an incorrect DNS server (192.0.2.53) rather than the lab domain controller's DNS server (192.168.193.10). As a result, direct DNS queries for the Active Directory domain and its LDAP service-location records failed.

<b>5. Resolution</b>

The preferred DNS server was restored to 192.168.193.10. DNS resolution, Active Directory domain controller discovery, and the computer Group Policy refresh were then tested successfully.

<b>6. Skills Demonstrated</b>

- Windows 11 network configuration and troubleshooting

- DNS resolution and DNS server identification

- Active Directory domain controller discovery
  
- DNS SRV records and LDAP service discovery

- Command-line diagnostics using ipconfig, ping, nslookup, nltest, and gpupdate

- Root-cause analysis and corrective action

- Post-resolution verification and technical documentation

<b>7. Outcome</b>

The simulated incident was resolved successfully. This exercise demonstrated how an incorrect DNS server setting can disrupt domain name resolution even when the client can still communicate with the domain controller by IP address. It also demonstrated the importance of testing the fix and recording evidence before closing an IT support ticket.
