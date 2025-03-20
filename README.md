# wiresharktracing
Network tracing using Wireshark, Python and KML.

<img width="1063" alt="Image" src="https://github.com/user-attachments/assets/ae3d4d58-e55c-4b68-801f-a61c65667834" />

## Prerequisites

Other than a Python IDE, you will need the following resources:

**GeoLiteCity**

This database will be utilised to translate an IP address into a geolocation (longitude & latitude). 

The database can be downloaded at: https://github.com/mbcc2006/GeoLiteCity-data

**Wireshark**

Wireshark captures network traffic on your device. 

The captured traffic will act as an input to our Python script and will be the data displayed at the end within Google Maps. 

Wireshark can be downloaded at: https://www.wireshark.org/

## Step 1: Capture network traffic

With Wireshark installed, we can create our input data which will consist of a captured __pcap__ file. 

The file will consist of all network traffic going to & from our device in the period we have the capture function activated.

To initialise a capture:

Open Wireshark
Select a interface which has traffic going through it (e.g. Ethernet, WiFi)
<img width="864" alt="Image" src="https://github.com/user-attachments/assets/370d8ef9-aa67-4f46-bbd1-8dfe6e55f9f7" />

Once an interface is selected, Wireshark automatically starts a new capture.

<img width="1424" alt="Image" src="https://github.com/user-attachments/assets/e335c767-2c84-4912-b919-eb8e7b6fcd64" />

After about 100 lines or so, stop the capture by pressing the red square in the top left-hand corner (next to the blue sharkfin).

Save your captured network traffic data in the **.pcap** file format.
For ease of access, you may save the file as main.pcap

<img width="800" alt="Image" src="https://github.com/user-attachments/assets/01030f34-d280-4325-b19f-c05f8577e706" />
