Overview:
Please make a list of what you think needs to be planned for the firewall migration?  In principle, we can divide this in to 4 phases:
1. Analysis
2. Planning
3. Doing
4. Operation


## Analysis
* Current hardware:
	* How many APs used for space?
	* Number of devices/users?
	* Average FW throughput (to measure utilization)
	* VPN tunnel activity utilization
* Current FW configuration:
	* FW rules
		* Is there any segmentation between VLANs, are there FW rules that are specific to restricting cross-VLAN communication?
	* VPN tunnels
	* Remote User VPN setup
	* Other configured services (user auth, dhcp/dns, UPnP, etc...)
	* Application FW rules? (These aren't super practical on Unifi as it doesn't fully support true application-level FW rules)
* Is the FW doing L3 routing or is that on the core switch?
	* VLAN configuration?
	* Subnet configuration/utilization
* Wireless Configuration:
	* Is the new FW going to manage Unifi wireless (some models have built in controllers to do so if needed, otherwise can be managed with part of centralized CloudKey)
	* SSID configuration
	* Are SSIDs on specific VLANs?
	* Is there guest access wireless?  How is it configured?

## Planning
Things to consider:
1. Available downtime window to swap hardware
2. How long would downtime take?
3. Verification procedure that everything works post migration:
	1. Check that VPN tunnels are up/operational
	2. Watch firewall logs to see if operational traffic is getting blocked by missed firewall rules
	3. Verify Remote user VPN works (if necessary)
4. Testing policies for business operation:
	1. Applications are still working
	2. Phones are still operational (if necessary)
3. Testing procedure of firewall ACLs
	1. Is it permitting what needs to be permitted?
	2. Is remote logging functioning (if required?)
	3. Is it blocking what needs to be blocked?
4. If using redundant internet connections, is that properly configured?
	1. Test failover to verify proper configuration
5. Who's doing what:
	1. Who is doing the hardware installation before cutover?
	2. Who is doing the actual cutover of network connectivity?
	3. Who is going to do on-site verification?  HQ verification?
6. Rollback plan
	1. If it doesn't work, what is the time window before rolling back to old setup?
	2. Have a proper documented plan to revert changes
7. (Optional) Time out every step of move and how long each part should take.  That way, you can have a more effective planned downtime window
8. If doing central monitoring, alert monitoring team of planned downtime/maintenance window
9. Depending on hardware, are you going to do centralized device management with a single CloudKey, or is every site going to be centrally managed (dependant on chosed firewall hardware)

For VPN configuration, if HQ is also using Unifi hardware, you can easily setup site-to-site vpn configuration from the unifi portal, no manual configuration really needed

#### Configuration steps:
* Mirror FW rules to new hardware from old hardware, have second person verify configuration
* Prep VPN tunnel configruation on new hardware
	* Have configuration in place
	* Determine if going to use new PSKs or generate new ones
		* If generating new ones, have configured on new hardware but prepped/documented to update on existing hardware
* Configure Other services:
	* DNS settings
		* Upstream dns servers
		* domain name suffix
	* DHCP scopes
		* If there are reservations currently in place, configure those in the scope configuration on new FW
		* TFTP configuration (if applicable)
		* Proxy settings (if applicable)
		* domain name suffix
	* VLAN configurations
* Wireless configuration (if applicable)
	* Configure SSIDs, addressing, VLAN configuration
	* FW ACLs & Guest Policies configuration
	* Guest access walled garden splash page? (User agreement, email verification, etc...)
* (If applicable) RADIUS authentication configuration
	* Verify that new FW can access the radius server and auth for user support (maybe for remote user vpn authentication)
* Pre-rack new hardware (if possible) and get cables ready to be quicky swapped over
* Document network cabling configuration
* Color-code network patch cables based on usage.  For example:
	* Red = Internet connection to ISP modem
	* Yellow = From FW to core switch

On existing hardware:
* Shorten dhcp lease times to as short as possible (makes for an easier dhcp cutover)
	* This will reduce the likelihood of duplicate IPs when coming up on new dhcp scope usage

Document process to create new site, what is needed to be done at HQ for standing up a new site

## Doing
Potential Order of Procedure:
1. Verify everyone required is ready to go at time of cutover
2. At go time:
	1. Drop VPN tunnels, disable remote user VPN on existing hardware
	2. Remove internet connections and connect to new FW
	3. Configure/cut over VPN tunnels.  Verify they come up properly
	4. Once things are verified operational, move over LAN connection to new FW
	5. Verify that VLAN routing configuration is operational
3. Post physical cutover work:
	1. Verify internet access for every VLAN with required Internet access
	2. Verify core applications are operational
	3. Verify other core network services are operational (backups, radius auth, dns queries, etc...)
4. Verify with monitoring team that they are able to see new hardware/properly connect to for monitoring purposes (if applicable)

For Wireless:
* Disable existing HW Radios, enable new ones
* Verify SSID connectivity on new SSIDs
* Verify network connectivity on all SSIDs

## Operation
* Perk of Unifi hardware - offers unified/centralized configuration capabilities of all site configurations
* Go over basic procedure for firewall rule CRUD actions (creation, update, deletion)
* Review the built-in montiroing dashboard
* Review process to view firewall logs
