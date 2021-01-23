A full stack LAN management and monitoring system

- Device details and polling (Python, SQLite)
- DHCP discover monitor (C++)
- SSH monitor (C++)
- Web UI (Javascript, Pug)
- Server back end (Node JS, Express, Socket.io, ws)


<b> LAN device details </b>
- Sqlitedb is a table with IP, MAC, hostname, last check time, last seen time and status (up/down)
- Python app can probe hosts from IPs found in table and then pings them from localhost, if the host doesn't respond, it's status is "D" and a time stamp of the check is added to the db.
- The --c flag simply dumps output from the text.
- The service runs once per minute 
- You can add or remove a device using --add or --del and this requires an IP, MAC and hostname to be input. This takes the data, adds it to the sqlite db but also adds static entries in /etc/dhcpd.manual and then restarts dhcpd so that any new devices with that MAC will get the desired IP.

<b> DHCP Monitor </b>
- Written in C++ and uses libtins and libcurl to sniff on an interface.
- Filter is applied to sniffer to listen or UDP 67/68 
- If found, the packet headers are extracted 
- The data is converted into JSON in string format using a series of stringstreams (native instead of relying on libraries)
- This data is then sent via a protected API to the Node server.

<b> SSH Monitor </b>
- As above, except our filter is altered to listen for ports 22 instead of 67/68.

<b> WebUI </b>
- The front end of this service is Javascript and templated by Pug.
- The LAN service details simply run the Python checker with the --c flag and then dumps the std out nicely onto the page.
- You can add or remove a device using the form and it will act as a front end for the methodology listed in the LAN device details section.
- DHCP and SSH info are stored in an array size 100 on the back end (to keep memory usage low) and then passed through to pug in one full string with linebreaks (I would've gone back to make this using Vue so that I can use dynamic text but it works for what I need it to do)

<b> Back end </b>
- The part of the service that glues all other parts together


<b> Services </b>
Managing and remembering the name of each service is slightly tricky - so I made a script with each module of the whole project to start or stop these.


<b>Future Additions</b>
- I previously created a tool that allows you to poll or alter the configuration of a switch without having to telnet/SSH to it using Netmiko. I intend at some point to create a webform that will allow you to select a port and then either get or set a number of parameters (IP, VLAN information, routes, etc) much like the old style Cisco Catalyst switches and HP switches. Displaying running config may be slightly more difficult to streamline as the possibilities are quite vast and so it may require a single row of inputs as such to build the full command:
<i>(get/set) (interface/arp/route) (GigabitEthernet/mac) (<Port Number>)</i>
 This style would need to be generic on the web form side but hardcoded depending on device type to use the appropriate syntax. It would also need a complex array of options to use and offer some form of input validation.
- Netmetrix - a tool I created previously to measure latency and loss. Does a ping check every second for 60 seconds before resetting stats and calculates average latency and loss in percent. While this is ready now, I would need dynamic text to show the refreshed data every second. 
