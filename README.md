# Network tracing using Wireshark, Python and KML.

<img width="1063" alt="Image" src="https://github.com/user-attachments/assets/ae3d4d58-e55c-4b68-801f-a61c65667834" />

## Prerequisites

Other than a Python IDE, you will need the following resources:

**GeoLiteCity**

This database will be utilised to translate an IP address into a geolocation (longitude & latitude). 

The database can be downloaded at in the files section of this repo.

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

## Step 2: Python Implementation

Firstly, import the following libraries:

    import dpkt
    import socket
    import pygeoip

    gi = pygeoip.GeoIP('GeoLiteCity.dat')

As you will notice, a global variable is declared with the GeoLiteCity database that you downloaded earlier. 
Be sure to place .dat file in the root folder of your Python project (if you’re not to change the path of the variable.)

<img width="774" alt="Image" src="https://github.com/user-attachments/assets/630d6ffd-1f16-4109-85ba-b98aa5a6063e" />

Next, you will be implementing the main method.
This will open your captured data and create the header & footer of the KML file. The .KML file is the output file that we will upload to Google Maps. 
Again, be sure to place you captured .pcap file in the root of the project and change the name of the code below if you did not save it under the name **main.pcap**

    def main():
        f = open('main.pcap', 'rb')
        pcap = dpkt.pcap.Reader(f)
        kmlheader = '<?xml version="1.0" encoding="UTF-8"?> \n<kml     xmlns="http://www.opengis.net/kml/2.2">\n<Document>\n'\
      '<Style id="transBluePoly">' \
                    '<LineStyle>' \
                    '<width>1.5</width>' \
                    '<color>501400E6</color>' \
                    '</LineStyle>' \
                    '</Style>'
        kmlfooter = '</Document>\n</kml>\n'
        kmldoc=kmlheader+plotIPs(pcap)+kmlfooter
        print(kmldoc)

If you wish to change the styling of the lines that will be drawn in Google Maps, you may do so under the tags <style></style>. I have just adjusted the line width and colour but other style elements can be added like icons and whatnot.

Next, we add the method that loops over out captured network traffic data & extracts IP addresses.

    def plotIPs(pcap):
        kmlPts = ''
        for (ts, buf) in pcap:
            try:
                eth = dpkt.ethernet.Ethernet(buf)
                ip = eth.data
                src = socket.inet_ntoa(ip.src)
                dst = socket.inet_ntoa(ip.dst)
                KML = retKML(dst, src)
                kmlPts = kmlPts + KML
            except:
                pass
        return kmlPts

In the above method **plotIPs(pcap)**, the application will loop over our pcap data and extract source and destination IP addresses of each captured network packet.

However, IP addresses alone cannot be used as an input into Google Maps.

To get around this limitation, we must first attach a geolocation to the IP address.

    def retKML(dstip, srcip):
        dst = gi.record_by_name(dstip)
        src = gi.record_by_name('x.xxx.xxx.xxx')
        try:
            dstlongitude = dst['longitude']
            dstlatitude = dst['latitude']
            srclongitude = src['longitude']
            srclatitude = src['latitude']
            kml = (
                '<Placemark>\n'
                '<name>%s</name>\n'
                '<extrude>1</extrude>\n'
                '<tessellate>1</tessellate>\n'
                '<styleUrl>#transBluePoly</styleUrl>\n'
                '<LineString>\n'
                '<coordinates>%6f,%6f\n%6f,%6f</coordinates>\n'
                '</LineString>\n'
                '</Placemark>\n'
            )%(dstip, dstlongitude, dstlatitude, srclongitude, srclatitude)
            return kml
        except:
            return ''

In the above method, geolocations are located for each IP address - followed by a convertion into KML format. 

Be aware that in the above example I have hidden my own IP which is why it says x.xxx.xxx.xxx, replace the x's with your own IP address.. 

The reason why you need to input this is because our captured pcap data file has saved our internal IP which is not known outside our own home network. 

Execute the code using

    if __name__ == '__main__':
        main()

You will get an output which is a complete KML file, within the terminal.

<img width="772" alt="Image" src="https://github.com/user-attachments/assets/6072ecd8-0efc-4415-94be-855aa79a1282" />

Simply copy the outputted data into a file with a .kml extension.


## Step 3: Transposing data to Google Maps

Take your .kml file to https://www.google.com/mymaps.

Create a new map.

Click import a new layer.

<img width="293" alt="Image" src="https://github.com/user-attachments/assets/e6d3e1b7-67f4-46b9-8d44-0090482a2c9f" />


After it loads, your map might look something like this:

<img width="1063" alt="Image" src="https://github.com/user-attachments/assets/ae3d4d58-e55c-4b68-801f-a61c65667834" />

Clicking on a line tells you what the IP address associated is.

<img width="1265" alt="Image" src="https://github.com/user-attachments/assets/2658fa6a-d92f-4317-b00c-3c6b37fab5df" />



Tutorial complete :)



## Future implementation

Here are a few ideas off the top of my head to further extend the depth of this project:

**Automatic External IP Retrieval**

Currently, the script requires manual input of the source/local IP address. A valuable improvement would be to automatically retrieve the external/public IP address. This can be achieved using various online services or libraries (requests library)

**Enhanced KML Output**

The KML output could be enriched with additional information such as:

Chronological timestamp visualisation of network traffic from PCAP data.

Including information about the size and protocol used of the data packets

Custom icons for the placemarks, representing different types of network events.

Interactive elements in the KML; perhaps pop-up windows with detailed packet information.
