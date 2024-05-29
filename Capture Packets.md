# How to Capture Packets using CableLabs tool and iperf3
This is a guide on how to capture packets using the CableLabs capture packet tool, specifically on iperf3 tests between devices. This is curated specifically for the Dell Optiflex in the CTech lab on the 3rd floor at the Low Latency Test Station. If you have any questions/comments, please contact zack.galpern@cox.com or Zack Galpern on Teams.


# Initial Steps
Either SSH into or be at the physical Dell station in the lab. I recommend being at the physical lab for this, as SSH is difficult to set up and requires extra equipment. However, if SSH is already set up on your local device, then follow instructions as written.

 - Open two command line interfaces and SSH into the two devices in separate tabs. (Intel NUCs or Mac Minis).
	 - IPs (subject to change):
		 - NUC1 (client): 192.168.0.236
		 - NUC2 (server): 192.168.0.86
		 - Mac Mini 1: 192.168.0.25
		 - Mac Mini 2: 192.168.0.136
- ```ssh zack@ip```
	- Contact Zack for password.
- Once both devices have been successfully connected, follow instructions for the devices you are testing (either NUCs or Macs).

## Intel NUC iperf3 Testing
- In both SSH windows, enable TCP prague (may be redundant) (this is all on one line):
	- `sudo sysctl net.ipv4.tcp_ecn=3 net.ipv4.tcp_congestion_control=prague`
- From the SSH window of the server side (zack@l4s-NUC2), open an iperf3 server:
	- ```iperf3 -s```
	-  You should see ```Server listening on 5201``` if successfully started
- On the client side (zack@l4sNUC1), you have the option of testing with L4S or without L4S (we'll call this Classic testing).
### Classic and L4S Testing
- Before running the iperf3 test, navigate to localhost in a browser and open the Packet Capture User Interface.
- In the Test name box, name the test with specific terms that will help you remember which test the results apply to.
- The test duration should be 15 seconds (or 5 seconds longer than the iperf3 test)
- Do NOT click Submit until the iperf3 client is set up and ready to run (but do not run that yet either)
- In the client window, set up an iperf3 client for 10 seconds but do NOT click enter yet:
	- Classic Upstream: ```iperf3 -c 10.0.252.5 -t 10 -C cubic```
	- Classic Downstream: ```iperf3 -c 10.0.252.5 -t 10 -C cubic -R```
	- L4S Upstream: ```iperf3 -c 10.0.252.5 -t 10 -C prague```
	- L4S Downstream: ```iperf3 -c 10.0.252.5 -t 10 -C prague -R```
- Go back to the Packet Capture UI and click submit
- IMMEDIATELY press enter on the client and let the test run.
- In the UI, go to the Results File of the test you just ran (it usually takes an extra 10-15 seconds for all of the files to populate)
- The correct results file will be 2_upstream or 2_downstream, depending on if upstream or downstream was tested.

## Mac Mini iperf3-darwin Testing
Testing on the Mac Minis is similar to the NUCs with a few differences. Mac uses iperf3-darwin instead of iperf3, and there is an extra argument needed on the client side command line.

- From the SSH window of the server side (zack@l4s-NUC2), open an iperf3 server:
	- ```iperf3-darwin -s```
	-  You should see ```Server listening on 5201``` if successfully started
- On the client side (zack@ip184-176-49-135), you have the option of testing with L4S or without L4S (we'll call this QUIC testing, based on Apple QUIC).

### QUIC and L4S Testing
- Before running the iperf3 test, navigate to localhost in a browser and open the Packet Capture User Interface.
- In the Test name box, name the test with specific terms that will help you remember which test the results apply to.
- The test duration should be 15 seconds (or 5 seconds longer than the iperf3-darwin test)
- Do NOT click Submit until the iperf3-darwin client is set up and ready to run (but do not run that yet either)
- In the client window, set up an iperf3-darwin client for 10 seconds but do NOT click enter yet:
	- Classic Upstream: ```iperf3-darwin --apple-quic -c 10.0.252.4 -t 10```
	- Classic Downstream: ```iperf3-darwin --apple-quic -c 10.0.252.4 -t 10 -R```
	-  L4S Upstream: ```iperf3-darwin --apple-quic --apple-l4s -c 10.0.252.4 -t 10```
	- L4S Downstream: ```iperf3-darwin --apple-quic --apple-l4s -c 10.0.252.4 -t 10 -R```
- Go back to the Packet Capture UI and click submit
- IMMEDIATELY press enter on the client and let the test run.
- In the UI, go to the Results File of the test you just ran (it usually takes an extra 10-15 seconds for all of the files to populate)
- The correct results file will be 2_upstream or 2_downstream.
	- NOTE: Sometimes 1_upstream will be the correct file. I'm not sure why, but it should be easy to tell which one is correct (e.g. which one contains ECT1 bits).


