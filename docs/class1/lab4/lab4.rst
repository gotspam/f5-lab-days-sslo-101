LAB 4 – DELETE AN SSLO CONFIGURATION
====================================

One of the benefits of the new SSLO architecture is that
configurations can be edited, deployed and re-deployed without
affecting existing traffic flows. For this capability, the SSLO
packaging is now broken into separate independent components. When
deleting a defined topology, most of the attached components are
also deleted. However, some objects, particularly those that can be
consumed by multiple topologies, are not automatically deleted. This
lab explores the different methods for deleting SSL Orchestratorobjects.

Step 1: Deleting a topology
---------------------------

Deleting a topology will also delete any relying Interception Rules.
The deletion process performs a complex set of REST-based tasks,
therefore only one topology can be deleted at a time. In the SSLO
UI, select a topology and click the Delete button. Confirm that both
the topology and respective interception rules are removed.

Step 2: Deleting other objects
------------------------------

While deleting a topology also removes its respective interception
rules, it does not remove the other objects - services, service
chains, security policies and SSL settings. These can all be removed
individually, however must be deleted in a hierarchical order. Once
the topology and interception rules have been deleted,

-  SSL Settings can be deleted any time

-  Delete any unused Security Policies

-  Delete any unused Service Chains

-  Delete any unused Services

Step 3: Deleting everything
---------------------------

To completely remove the SSLO configuration and start from scratch,

-  In the SSLO UI, click Delete Configurations and then click OK. This
   process will take some time as SSLO walks through all of the
   objects and dependencies to remove all configurations.

-  Under the iApps menu, Application Services, Applications LX –
   un-deploy any remaining SSL orchestrator objects. If using any
   other Guided Configuration engine (ex. Access GC), ensure that
   only SSLO objects are deleted here.

-  Under the iApps menu, Templates, Templates LX – delete all of the SSL
   Orchestrator templates.

-  Under the iApps menu, Package management LX – delete the SSL
   Orchestrator package.

   The next time the SSL Orchestrator configuration menu is accessed,
   SSLO will automatically restore the on-box package.

Optional: Deleting everything…the hard way
------------------------------------------

In the unlikely event that the above steps do not work, and some
SSLO objects remain and cannot be deleted, one of the following
steps can be used,

-  If the topology and interception rules are gone but other objects
   remain and will not uninstall in the SSL Orchestrator UI, in the
   BIG-IP UI navigate to iApps -> Application Services -> Applications
   LX. The remaining objects will all be here in states of deployed
   (green), undeployed (gray), and error (red). Delete any objects in an
   error state and toggle the other objects from deployed to undeployed
   and back until they enter an error state and can also be deleted.

-  If the above fails, the following script can be used to automate
   destruction of SSLO objects.

   -  Copy the script to the BIG-IP (ex. cleaner.sh)

   -  Chmod the script to give it execute privileges: chmod +x
      cleaner.sh

   -  Execute the script: ./cleaner.sh

   -  It will typically be necessary to execute the script several times
      to get through dependencies. It is completely done when the script
      returns quickly with no additional output. Validate that all SSLO
      objects are gone from the BIG-IP UI under the Local Traffic and
      Network sections.

   -  Under the iApps menu, Application Services, Applications LX –
      un-deploy any remaining SSL orchestrator objects. If using any
      other Guided Configuration engine (ex. Access GC), ensure that
      only SSLO objects are deleted here.

   -  Under the iApps menu, Templates, Templates LX – delete all of the
      SSL Orchestrator templates.

   -  Under the iApps menu, Package management LX – delete the SSL
      Orchestrator package.

-  If the above fails, manually clear the REST database from the command
   line,

-  Break any HA configuration

-  Issue the ‘clear-rest-storage [options]’ command, where the options
   are “-l” (lowercase L) to delete the restjavad log files as well as
   the stored state, and “-d” to reset the system configuration to
   default. This command will remove all SSL Orchestrator objects from
   the restnoded database. After issuing this command, follow with
   ‘bigstart restart restnoded’ and ‘bigstart restart restjavad’, clear
   the browser cache, log out and back in.

-  Issue the ‘tmsh delete sys application service recursive’ command to
   also delete any remaining SSL Orchestrator application service
   objects.

-  Once all SSLO objects have been removed, also uninstall the SSLO RPM
   package under the iApps menu, Package management LX – delete the SSL
   Orchestrator package.

-  Rebuild HA and redeploy SSLO by navigating to the SSL Orchestrator
   configuration UI. On first visit it will automatically restore the
   on-box package.

TROUBLESHOOT SSLO
=================

While the SSL Orchestrator product has certainly evolved, as with
anything in the computing world, problems are usually inevitable and
poorly timed. In the event that an SSL Orchestrator configuration
has failed, or that it has succeeded but not behaving as expected,
the following troubleshooting tools should be useful.

Step 1: Test the configuration
------------------------------

It is important to first define “normal” behavior. If the SSL
Orchestrator deployment process was successful, it will be possible
to access remote Internet sites from the client workstation without
issue, and HTTPS sites appear to have a locally-trusted, re-issued
server certificate. This would be considered normal behavior. If any
of these do not happen, use the tools below to troubleshoot.

Step 2: Troubleshoot
--------------------

Below is a reasonably-ordered list of troubleshooting steps.

-  If the SSL Orchestrator deployment process fails, review the ensuing
   error message. It would be impossible to list here all of the
   possible error messages and their meanings, but often enough the
   messages will reveal the issue.

-  Re-review the lab steps for any missing or misconfigured settings.

-  Enable debug logging in the SSL Orchestrator configuration. Tail the
   APM log from a BIG-IP command line or from the logs page in the
   management UI. Debug logging will very often reveal important
   issues. Specifically, it will indicate traffic classification
   matches, mismatches or deployment issues.

   **tail -f /var/log/apm**
   **tail -f /var/log/restnoded/restnoded.log**
   **tail -f /var/log/restjavad.0.log**

-  If the SSL Orchestrator deployment process succeeds, but traffic
   isn’t flowing through the environment made evident by lack of
   access to remote sites from the client:

   -  Ensure that the client is properly configured to either default
      route to the ingress VLAN and self- IP of the BIG-IP for
      transparent proxy access or has the correct browser proxy
      settings defined for explicit proxy access.

   -  Ensure that traffic is flowing to the BIG-IP from the client with
      a tcpdump capture at the ingress interface.

   -  Review the LTM configuration created by the SSL Orchestrator.
      Specifically, look at the pools and respective monitors for
      any failures.

   -  Isolate service chain services. If at least one service chain has
      been created, and debug logging indicates that traffic is
      matching this chain, remove all but one service from that
      chain and test. Add one service back at a time until traffic
      flow stops. If a single added service breaks traffic flow,
      this service will typically be the culprit.

-  If a broken service is identified, insert probes to verify inbound
   and outbound traffic flow. Inline services will have a source (S)
   VLAN and destination (D) VLAN, and ICAP and receive only services
   will each have a single source VLAN. Insert a tcpdump capture at each
   VLAN in order to determine if traffic is getting to the device, and
   if traffic is leaving the device through its outbound interface.

-  If no service chains are defined, it may be necessary to remove all
   of the defined services and re- create them one-by-one to validate
   flow through the built-in All chain. If a broken service is
   identified, insert tcpdump probes as described above.

-  If traffic is flowing through all of the security devices, insert a
   tcpdump probe at the egress point to verify traffic is leaving the
   BIG-IP to the gateway router.

   tcpdump -I 0.0:nnn -nn -Xs0 -vv -w <file.pcap> <any additional filters>

-  If traffic is flowing to the gateway router, perform a more extensive
   packet analysis to determine if SSL if failing between the BIG-IP
   egress point and the remote server.

   Then either export this capture to WireShark are send to ssldump:

   ssldump -nr <file.pcap> -H -S crypto > text-file.txt

-  If the WireShark or ssldump analysis verifies an SSL issue:

   -  Plug the site’s address into the SSLLabs.com server test site at:
      `https://wwww.ssllabs.com/ssltest <http://www.ssllabs.com/ssltest/>`__

   This report will indicate any specific SSL requirements that this site has.

-  Verify that the SSL Orchestrator server SSL profiles (two of them)
   have the correct cipher string to match the requirements of this
   site. To do that, perform the following command at the BIG-IP command
   line:

   tmm --clientciphers ‘cipher string as displayed in server ssl profiles’

-  Further SSL/TLS issues are beyond the depth of this lab guide. Seek
   assistance.

-  If all else fails, seek assistance.

APPENDIX – COMMON TESTING COMMANDS
==================================

The following are some simple, but powerful commands that are useful
in developing and troubleshooting SSL visibility projects.

Control the SSLFWD certificate cache
------------------------------------

The behavior of the SSL Forward Proxy changes after a certificate is
cached, which will make it difficult to troubleshoot some issues.
The following allows you to both list and delete the certificates in
the cache.

tmsh show ltm clientssl-proxy cached-certs clientssl-profile [CLIENTSSLPROFILE] virtual INGRESSTCPVIP]
tmsh delete ltm clientssl-proxy cached-certs clientssl-profile [CLIENTSSLPROFILE] virtual INGRESSTCPVIP]

Isolate SSLO traffic
--------------------

Any given website will be full of images, scripts, style sheets, and
very often references to document objects on other sites (like a
CDN). This can make troubleshooting very complex. The following cURL
commands allow you to isolate traffic to a single request and response.

curl -vk https://www.bing.com

curl -vk --proxy 10.30.0.150:3128 https://www.bing.com

curl -vk --proxy 10.30.0.150:3128 - -location https://www.bing.com

Optionally, between each cURL test, delete the certificate cache and
start logging:

tmsh delete ltm clientssl-proxy cached-certs clientssl-profile [CLIENTSSLPROFILE] virtual INGRESSTCPVIP] && tail -f /var/log/apm

Debugging
---------

There is simply nothing better than debug logging for
troubleshooting SSL intercept issues. The SSL Orchestrator in debug
mode pumps out an enormous set of logs, describing every step along
a connection’s path. Remember to never leave debug logging enabled.

Packet capture
--------------

Second only to debug logging, packet captures are crucial to
troubleshooting any network-dependent issue.

tcpdump -lnni [VLAN] [-Xs0]

In-line services create “source” (S) and “destination” (D) VLANs,
and ICAP and receive-only services attach to existing VLANs. Drop a
probe at each point in the path and observe flow.

SSL inspection
--------------
ssldump -AdNd -i [VLAN] port 443 <add additional filters>

tcpdump -i 0.0:nnn -nn -Xs0 -vv -w <file.pcap> <and additional filters>

ssldump -nr <file.pcap> -H -S crypto > text-file.txt

TLS is rarely the issue, but a sight or configuration error may
render some sites inaccessible.

Control the URL Filtering database
----------------------------------

To show the current status of the database:
tmsh list sys url-db download-result

To initiate (force) the URL DB to update:
tmsh modify sys url-db download-schedule all stats true download-now true

To verify that the URL DB is actively updating:
tcpdup -lnni 0.0 port 80 and host 204.15.67.80

External testing
----------------

Plug the site’s address into SSLLabs.com server test site at
`https://wwww.ssllabs.com/ssltest <http://www.ssllabs.com/ssltest/>`__ to see if the site has any
unusual SSL/TLS requirements.

APPENDIX – ROUTING CONSIDERATIONS FOR LAYER 3 DEVICES
=====================================================

SSL Orchestrator sends all traffic through an inline layer 3 or HTTP
device in the same direction – entering through the inbound
interface. It is likely, therefore, that the layer 3 device may not
be able to correctly route both outbound (forward proxy) and inbound
(reverse proxy) traffic at the same time. Please see the appendix,
“Routing considerations for layer 3 devices” for more details. For
example, in a simple Linux-type environment there would be two
routes needed for SSLO:

-  The default gateway to send traffic back to SSLO through the
   service’s outbound interface

-  A static return route to allow client traffic to return through the
   service’s inbound interface Example:

   In the above, configured for an outbound traffic flow, the default
   gateway is on the outbound side interface (eth2), with a static
   route for 10.1.10.0/24 (client-sourced) traffic flowing back through
   the inbound interface (eth1). An inbound flow, however, would
   require the opposite:

+---------------------+-----------------------+-----------------------+---------------+----------------+---------------+
|     *Destination*   |     *Gateway*         |     *Genmask*         |     *Flags*   |     *Metric*   |     *iFace*   |
+=====================+=======================+=======================+===============+================+===============+
|     *default*       |     *198.19.64.7*     |     *0.0.0.0*         |     *UG*      |     *0*        |     *eth1*    |
+---------------------+-----------------------+-----------------------+---------------+----------------+---------------+
|     *10.1.10.0*     |     *198.19.64.245*   |     *255.255.255.0*   |     *UG*      |     *o*        |     *eth2*    |
+---------------------+-----------------------+-----------------------+---------------+----------------+---------------+

There are generally a few options for handling inbound and outbound
traffic flows:

-  Do not use the same layer 3 device for inbound and outbound flows –
   the simplest option, but not always possible in some environments.

-  Create a policy route, if the device supports it, to create multiple
   gateways. We will explore the second and second options below.

Configuring a policy route on the layer 3 device
------------------------------------------------

If a service supports it, policy routing allows you to create
multiple gateways on a layer 3 (routed) device. In lieu of creating
separate inbound and outbound services, and service chains for a
single L3 device, all traffic in this use case still flows through
the inbound side interface, but the policy route will effectively
steer traffic in the correct direction. Policy routing can be a
complex topic in and of itself, and each security product will have
its own way of configuring policy routing anyway, so it cannot be
covered in total in this guide. Please refer to product-specific
documentation to learn more about your policy routing options.

The following is an example script to enable a policy route on a
generic Linux device (most of which have iproute2 installed by
default). In the script, it is only necessary to modify the top
eight variables, defining attributes of the inbound and outbound
networks. Once complete, chmod the script to make it executable,
test it, and then call it from a startup process like /etc/rc.local
or /etc/init.d/rc.local. If the script is successful, you should be
able to send inbound and outbound SSLO traffic flows through this device.

.. Code::

   #!/bin/bash

   ## Inbound interface inbound\_interface=eth1.10
   inbound\_ip=198.19.64.65 inbound\_mask=25 inbound\_gw=198.19.64.7

   ## Outbound interface outbound\_interface=eth1.20
   outbound\_ip=198.19.64.130 outbound\_mask=25
   outbound\_gw=198.19.64.245

   ### ---------------------------------------------- ###

   ### ---------------------------------------------- ###

   ## static table names inbound\_table=av\_in outbound\_table=av\_out

   ## function to get network from mask and IP get\_network () {

   IFS=. read -r io1 io2 io3 io4 <<< "$2"

   set -- $(( 5 - ($1 / 8) )) 255 255 255 255 $(( (255 << (8 - ($1 %8))) & 255 )) 0 0 0
   [ $1 -gt 1 ] && shift $1 \|\| shift
   NET\_ADDR="$((${io1} & ${1-0})).$((${io2} & ${2-0})).$((${io3} &
   ${3-0})).$((${io4} & ${4-0}))"

   echo "$NET\_ADDR"

   }

   ## stop if iproute2 isn not installed if ! [ -d "/etc/iproute2/" ];
   then

   echo "iproute2 policy routing is not available on this system -
   exiting" exit

   fi

   ## create the ipproute2 tables

   if ! grep -q ${inbound\_table} /etc/iproute2/rt\_tables; then echo
   "200 ${inbound\_table}" >> /etc/iproute2/rt\_tables

   fi

   if ! grep -q ${outbound\_table} /etc/iproute2/rt\_tables; then echo
   "201 ${outbound\_table}" >> /etc/iproute2/rt\_tables

   fi

   ## get the inbound and outbound networks from function
   inbound\_net=$(get\_network ${inbound\_mask} ${inbound\_ip})

APPENDIX – DEMO SCRIPTS
=======================

Lab 1 demo script
-----------------

**Configuration review and prerequisites**

1. Optionally define DNS, NTP and gateway route

2. Click Next

**Topology Properties**

1. Name - some name

2. Protocol: Any

3. IP Family: IPv4

4. Topology: L3 Outbound

5. Click Save & Next

**SSL Configuration**

1. Create a New SSL Profile

2. Client-side SSL (Cipher Type): Cipher String

3. Client-side SSL (Cipher String): DEFAULT

4. Client-side SSL (Certificate Key Chain): default.crt and default.key

5. Client-side SSL (CA Certificate Key Chain): subca.f5demolabs.com

6. Server-side SSL (Cipher Type): Cipher String

7. Server-side SSL (Cipher String): DEFAULT

8. Server-side SSL (Trusted Certificate Authority): ca-bundle.crt

9. Click Save & Next

**Service List**

1. **Inline Layer 2 service**

   a. Name: some name (ex. FireEye)

   b. Network Configuration

      -  Ratio: 1

      -  From BIGIP VLAN: Create New, name (ex. FireEye\_in), int 1.6

      -  To BIGIP VLAN: Create New, name (ex. FireEye\_out), int 1.7

      -  Click Done

   c. Service Action Down: Ignore

   d. Enable Port Remap: Enable, 8080

   e. Click Save

1. **Inline layer 3 service**

   a. Name: some name (ex. IPS)

   b. IP Family: IPv4

   c. Auto Manage: Enabled

   d. To Service Configuration

      - To Service: 198.19.64.7/25

      -  VLAN: Create New, name (ex. IPS\_in), interface 1.3, tag 50

   e. Service Action Down: Ignore

   f. L3 Devices: 198.19.64.64

   g. From Service Configuration

      - From Service: 198.19.64.245/25

      - VLAN: Create New, name (ex. IP\_out), interface 1.3, tag 60

   h. Enable Port Remap: Enabled, 8181

   i. Manage SNAT Settings: None

   j. Click Save

1. **Inline HTTP service**

   a. Name: some name (ex. Proxy)

   b. IP Family: IPv4

   c. Auto Manage: Enabled

   d. Proxy Type: Explicit

   e. To Service Configuration

      -  To Service: 198.19.96.7/25
      -  VLAN: Create New, name (ex. Proxy\_in), interface 1.3, tag 110

   f. Service Action Down: Ignore

   g. HTTP Proxy Devices: 198.19.96.66, Port 3128

   h. From Service Configuration

      -  From Service: 198.19.96.245/25
      -  VLAN: Create New, name (ex. Proxy\_out), interface 1.3, tag 120

   i. Manage SNAT Settings: None

   j. Authentication Offload: Disabled

   k. Click Save

1. **ICAP Service**

   a. name: some name (ex. DLP)

   b. IP Family: IPv4

   c. ICAP Devices: 10.70.0.10, Port 1344

   d. Request URI Path: /squidclamav

   e. Response URI Path: /squidclamav

   f. Preview Max Length(bytes): 524288

   g. Service Action Down: Ignore

   h. Click Save

1. **TAP Service**

   a. Some Name (ex. TAP)

   b. Mac Address: 12:12:12:12:12:12

   c. VLAN: Create New, name (ex. TAP\_in)

   d. Interface: 1.4

   e. Service Action Down: Ignore

   f. Click Save

   g. Click Save & Next

**Service Chain List**

1. Add

   a. Name: some name (ex. my-service-chain)

   b. Services: all of the services

   c. Click Save

2. Add

   a. name: some name (ex. my-sub-service-chain)

   b. Services: L2 and TAP services

   c. Click Save

3. Click Save & Next

**Security Policy**

1. Add a new rule

   a. Name: some name (ex. urlf\_bypass)

   b. Conditions

      -  Category Lookup (All)

      -  SNI Category: Financial Data and Services, Health and Medicine

   c. Action: Allow

   d. SSL Forward Proxy Action: bypass

   e. Service Chain: L2/TAP service chain

   f. Click OK

2. Modify the All rule

   a. Service Chain: all services chain

   b. Click OK

3. Click Save & Next

**Interception Rule**

a. Select Outbound Rule Type: Default

b. Ingress Network (VLANs): client-side

c. L7 Interception Rules: apply FTP and email protocols as required

d. Click Save & Next

**Egress Setting**

a. Manage SNAT Settings: Auto Map

b. Gateways: New, ratio 1, 10.30.0.1

**Summary**

a. Review configuration

b. Click Deploy

Lab 2 demo script
-----------------

**Configuration review and prerequisites**

1. Optionally define DNS, NTP and gateway route

2. Click Next

**Topology Properties**

1. Name: some name (ex. sslo-inbound-1)

2. Protocol: TCP

3. IP Family: IPv4

4. Topology: L3 Inbound

5. Click Save & Next

**SSL Configuration**

1.  Show Advanced Setting

2.  Client-side SSL (Cipher Type): Cipher String

3.  Client-side SSL (Cipher String): DEFAULT

4.  Client-side SSL (Certificate Key Chain): default.crt and default.key

5.  Server-side SSL (Cipher Type): Cipher String

6.  Server-side SSL (Cipher String): DEFAULT

7.  Server-side SSL (Trusted Certificate Authority): ca-bundle.crt

8.  Advanced (Expire Certificate Control): Ignore

9.  Advanced (Untrusted Certificate Authority): Ignore

10. Click Save & Next

**Services List**

1. Click Save & Next

**Service Chain List**

1. Click Save & Next

**Security Policy**

1. Remove Pinners\_Rule

2. Edit All Traffic rule and add L2/TAP service chain

3. Click Save & Next

**Interception Rule**

1. Gateway-mode

   a. Hide Advanced Setting

   b. Source Address: 0.0.0.0/0

   c. Destination Address/Mask: 0.0.0.0/0

   d. Port: 443

   e. VLANs: outbound

2. Targeted-mode

   a. Show Advanced Setting

   b. Source Address: 0.0.0.0/0

   c. Destination Address: 10.30.0.200

   d. Port: 443

   e. VLANs: outbound

   f. Pool: webserver-pool

3. Click Save & Next

**Egress Settings**

1. Manage SNAT Settings: Auto Map

2. Gateways: Default Route

**Summary**

1. Review configuration

2. Click Deploy

Lab 3 demo script
-----------------

**Configuration review and prerequisites**

1. Optionally define DNS, NTP and gateway route

2. Click Next

**Topology Properties**

1. Name: some name (ex. sslo-explicit)

2. Protocol: TCP

3. IP Family: IPv4

4. Topology: L3 Explicit Proxy

5. Click Save & Next

**SSL Configuration**

1. SSL Profile: Use Existing, existing outbound SSL settings

2. Click Save & Next

**Services List**

1. Click Save & Next

**Service Chain List**

1. Click Save & Next

**Security Policy**

1. Type: Use Existing, existing outbound security policy

2. Click Save & Next

**Interception Rule**

1. IPV4 Address: 10.20.0.150

2. Port: 3128

1. VLANs: client-net

2. Click Save & Next

**Egress Settings**

1. Manage SNAT Settings: Auto Map

2. Gateways: Existing Gateway Pool, -ex-pool-4 pool

**Summary**

1. Review configuration

2. Click Deploy

**System Settings**

1. DNS Query Resolution: Local Forwarding Nameserver

2. Local Forwarding Nameserver(s): 10.1.20.1

3. Click Deploy
