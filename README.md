# kismetTakFlow
Node-Red Flow to forward Kismet Alerts and/or GPS location (as CoT icon) to ATAK via multicast or TAKServer. (used in conjunction with Ampledatas node-red-contrib-tak).

*it is recommended to use kismetAtakCompanion (kismet web plugin) over kismetTakFlow!

![flow](/kismetTakFlow.png?raw=true "Node Red Flow")

![flow](/kismetTakFlowUi.png?raw=true "Node Red Flow")

![flow](/kismetAlertsATAK.png?raw=true "ATAK Alerts")

![flow](/kismetTargetsATAK.png?raw=true "ATAK Targets")

REQUIREMENTS:

-Node-Red

-node-red-contrib-tak https://github.com/snstac/node-red-contrib-tak

-TAKServer (or Multicast supported devices)

-Kismet Server (actively running with GPS and WiFi Card)

SETUP NODE-RED:

-https://nodered.org/docs/getting-started/

-options include installing node-red on your local PC, a Raspberry Pi or similar single board computer or mini PC, deploying node-red on a cloud server or virtual private server (VPS), Docker Container, or local virtual environment. After installing node-red you should be able to go to the node-red dashboard at http://*nodeRedIPaddress:1880 you may have to open appropriate ports (1880) to allow devices to access the node-red dashboard.

-ensure you install the "node-red-contrib-tak" node. If not there should be a prompt when you import kismetTakFlow in Node-Red that there are nodes that need to be installed and will automatically install them for you if you allow. In the event that isnt the case, in Node-Red: Menu (3 horizontal lines) > Manage palette > Install > Search "node-red-contrib-tak" > Install > Install

-----------------------------

IMPORT .JSON FLOW TO NODE RED:

-in GitHub: click on "kismetTakFlow.json" > click on the download icon "Download raw file" > note where the "kismetTakFlow.json" file downloaded to, default is in your Downloads folder.

-in Node-Red: click on menu icon (3 horizontal lines top right) > click on "Import" > click on "select a file to import" > go to Downloads folder and click on "kismetTakFlow.json" > Upload > Import

ALTERNATIVELY..

-you can just copy the whole "kismetTakFlow.json" code from GitHub and paste it into the Node-Red Import Clipboard.

-------------------------------

CONFIGURE TAKSERVER:

-TAKServer can be hosted on various platforms: Raspberry Pi, Mini PCs, your local PC, Cloud Servers or Virtual Private Servers (VPS), Docker Containers, or Virtual Environments. Regardless, if you're looking for instructions on setting up your TAKServer I recommend ATAKHQs guides https://www.atakhq.com/en/tak-server.

-Configuring the TCP (TAKServer) node in Node-Red kismetTakFlow, Input your TAKServer IP and Port, and checkbox whether you're using SSL/TLS or not. If you are using SSL/TLS, ensure you upload you TAKServer certificates and key that are located in the directory of your TAKServer that stores all client certificates/keys (ie: ubuntu docker container is "/opt/tak/certs/files"). I Suggest creating a TAK client "node-red" to be used specifically for Node-Reds handling of forwarding data to TAK. In your certificates file you will want to copy and upload "node-red.pem" as the "Certificate", "node-red.key" as the "Private Key" and "ca.pem" as the "CA Certificate" in the TLS Configuration Properties in the TCP (TAKServer) node in Node-Red. You will also need to enter the Passphrase for your TAKServer truststore, this can be found in your TAKServers CoreConfig.xml file. Default is "atakatak".

----------------------------------

CONFIGURE PING NODE:

-in Node-Red: configure ping node "repeat" > "interval" and set the frequency of how often the kismet alerts are requested and then forwarded to TAK. The http GET request flow should just be used to get alerts that were missed before subscribing to kismet with websocket (eventbus)

-----------------------------------

CONFIGURE BOTH HTTP NODES:

-in Node-Red: input your kismet device IP address and port (default port is 2501)

-----------------------------------

CONFIGURE BOTH WS (IN/OUT) NODES:

-in Node-Red: input your login credentials for kismet device followed by IP address and port (default port is 2501). ie: ws://username:password@kismet-ip-address:2501

-----------------------------------

CONFIGURE TGT DEVICES:

-in Node-Red: input your target MACs and SSIDs in the tgt devices node. Ensure to follow the format of how its written JUST.. LIKE.. THE.. EXAMPLE.

------------------------------------

USING THE KISMETTAKFLOW:

-short answer: make sure kismets UI is actively running and just click "sub alert" in the flow.

-once your TCP TAKServer node is connected, your http nodes are configured, and your ws nodes are configured, you're ready to start having node-red listen in for kismet alerts to popup and get forwarded to TAK. Open ATAK/iTAK and test to make sure you're receiving CoT either from multicast or the TAKServer.

-ping node repeat/intervals aren't necessary for this flow since the ws node (websocket/eventbus) will automatically send CoT to TAK once alerts (or tgt devices) are detected from kismet in realtime. The http GET request flow is mostly just to catch any alibi alerts missed that are still stored in kismet. Feel free to have it default ping every few minutes if you feel its necessary as a failsafe to not have missed alerts.

-Begin by clicking the sub alerts node to have node red wait for kismet alerts to get forwarded to TAK. Click the sub mac/ssid node to have node red populate a CoT to TAK once targeted SSIDs or MACs are detected by kismet.

-creating a list of targeted SSIDs and MACs can be done in the tgt devices node.

-to stop data flow from websocket/eventbus, use the unsub node.


------------------------------------

HOW THE KISMETTAKFLOW WORKS:

-the top flow sends an http GET request to kismets web api that pulls all alerts that kismet has stored from the current active scan but repackages it up into CoT to push to TAKServer or multicast for TAK devices to view.

-the mid flow sends Kismets GPS Location to TAK in configured intervals to be used for device tracking.

-the bottom flow is subscribing to certain topics of kismets websocket eventbus (alerts and/or scan messages to detect if targeted devices are present) so when something is detected it will send the data and be repackaged as a CoT in realtime to be pushed to TAKServer or multicast.

-all features in the kismetTakFlow can be subscribed and toggled in the Node-Red UI (http://node-red-ip-address:1880/ui), the ping interval frequency still needs to be configured in the node-red dashboard.

--------------------------------------

TROUBLESHOOTING / KNOWN ISSUES:

-in the event the Node-Red service explodes and you lose connection to the Node-Red UI and cant reconnect, you may need to restart your Node-Red service

$ sudo service node-red restart
