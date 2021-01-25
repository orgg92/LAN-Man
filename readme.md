A full stack LAN management and monitoring system

- Device details and polling (Python, SQLite)
- DHCP discover monitor (C++)
- SSH monitor (C++)
- Host finder (C++)
- Web UI (Javascript, Pug)
- Server back end (Node JS, Express, ws)


<b> LAN device details </b>
- Sqlitedb is a table with IP, MAC, hostname, last check time, last seen time and status (up/down)
- Python app can probe hosts from IPs found in table and then pings them from localhost, if the host doesn't respond, it's status is "D" and a time stamp of the check is added to the db.
- The --c flag simply dumps output from the text.
- The service runs once per minute 
- You can add or remove a device using --add or --del and this requires an IP, MAC and hostname to be input. This takes the data, adds it to the sqlite db but also adds static entries in /etc/dhcpd.manual and then restarts dhcpd so that any new devices with that MAC will get the desired IP. <b> Note: </b> A DHCP server on the box will need configuring to include the listening interface, a subnet declaration and the /etc/dhcpd.manual. Without this, items will only be added to the db for monitoring and then it will fail.

<b> DHCP Monitor </b>
- Written in C++ and uses libtins and libcurl to sniff on an interface.
- Filter is applied to sniffer to listen or UDP 67/68 
- If found, the packet headers are extracted 
- The data is converted into JSON in string format using a series of stringstreams (native instead of relying on libraries)
- This data is then sent via a protected API to the Node server.

<b> SSH Monitor </b>
- As above, except our filter is altered to listen for ports 22 instead of 67/68.

<b> Host finder </b>
- A ping sweep utility that scans a /24 based off the specified interface.
- If a host does not respond, we also try an arping to check or any responses, if none we consider the host offline (granted this does not consider ICMP being filtered but this could always be enhanced using an nmap scan per host to check for open ports which would inevitably be a slower process).

<b> WebUI </b>
- The front end of this service is Javascript and templated by Pug.
- The LAN service details simply run the Python checker with the --c flag and then dumps the std out nicely onto the page.
- You can add or remove a device using the form and it will act as a front end for the methodology listed in the LAN device details section.
- DHCP and SSH info are stored in an array size 100 on the back end (to keep memory usage low) and then passed through to pug in one full string with linebreaks (I would've gone back to make this using Vue so that I can use dynamic text but it works for what I need it to do)

<b> Back end </b>
- The part of the service that glues all other parts together
- JSON data sent using the API is interpreted based on the api route and then pushed into the relevant array
- When each array gets to a length of 100, the first element will be shifted out of the array (here we could implement permanent storage and store events in blocks of 100)
- Express is used to create the routes and as previously mentioned the templating engine is Pug.
- So far the project uses Basic Auth with a generic username/pass combo and the API end points protected with a token.

<b> Services </b>
- Managing and remembering the name of each service is slightly tricky - so I made a script with each module of the whole project to start or stop these.


<b>Future Additions</b>
- I previously created a tool that allows you to poll or alter the configuration of a switch without having to telnet/SSH to it using Netmiko. I intend at some point to create a webform that will allow you to select a port and then either get or set a number of parameters (IP, VLAN information, routes, etc) much like the old style Cisco Catalyst switches and HP switches. Displaying running config may be slightly more difficult to streamline as the possibilities are quite vast and so it may require a single row of inputs as such to build the full command:
<i>(get/set) (interface/arp/route) (GigabitEthernet/mac) ([Port Number])</i>

- This style would need to be generic on the web form side but hardcoded depending on device type to use the appropriate syntax. It would also need a complex array of options to use and offer some form of input validation.
- Netmetrix - a tool I created previously to measure latency and loss. Does a ping check every second for 60 seconds before resetting stats and calculates average latency and loss in percent. While this is ready now, I would need dynamic text to show the refreshed data every second. 

- Using websockets for 2-way communication and IPC. I currently have a working prototype that works between Node ws module and C++'s websocketpp - I just haven't needed to implement bidirectional communication between services just yet.


I will eventually provide a full installer with all dependencies that can be used to build the project from source and provide periodic updates once I add things such as HTTPS and refactor the code in a more object oriented fashion.
