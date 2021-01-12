# Course--WebRTC-Crash-Course
Course tutorial referenced by [WebRTC Crash Course: 
Hussein Nasser](https://www.youtube.com/watch?v=FExZvpVvYxA)

WebRTC (Web Real-Time Communication) is a free, open-source project that provides web browsers and mobile applications with real-time communication (RTC) via simple application programming interfaces (APIs).

Goal: let's design a protocol that connects peer to peer that shortest possible lowest latency path and let's provide a nice api that a simple for every one to use.

## Why build **webRTC**?

We build it because we need to transmit media: (audio and video) in a standardized way in low latency way. standardized mean i need an API simple to use.

low latency: ?

## WebRTC main parts
 - NAT
 - STUN, TURN
 - ICE
 - SDP
 - Signaling the SDP

 ### Network Address Translation (NAT)
Devices that can be connected to the Internet have an IP address but in NAT we can use a private IP address in our private network. router configure to NAT i.e. router 10.0.0.2 private IP address to convert public IP address and store in NAT table with port. deep exprenation on: [geeksforgeeks](https://www.geeksforgeeks.org/network-address-translation-nat/)

![asdf](./images/NAT_Concept-en.png)

## NAT Transmissions Method

1. One to One NAT (Full-cone NAT)

    Full-cone NAT check only external ip address and port present or not in the NAT Table.

    The router configure that  Full-cone NAT mean 
    ![one-to-one-nat](./images/one-to-one-nat.png)

2. Address Restricted NAT

    Address Restricted NAT check only Destination IP


    ![one-to-one-nat](./images/address-restricted-nat.png)

3. Port Restricted NAT

    ![port-restricted-nat](./images/port-restricted-nat.png)

4. Symmetric NAT

    ![port-restricted-nat](./images/symmetric-nat.png)

## Session Traversal Utilities for NAT (STUN)

- We traverse the session. this is a bunch of utilities. one of the utilities is just get my public IP address and port through NAT and the layer of information put in the packet and then send it back to me as data.

- Work for Full-cone, Port/ Address restricted NAT.

- Doesn't work for symmetric NAT.

- these server port cheap to maintain because a docker container has a stun server and it's very lightweight. response back content. google give you public STUN server because they don't care they search so cheap to maintain.
    - STUN server port **3478**: server usually run on this port.
    - TLS server port **5349** : secure STUN

### STUN Request
Goal: We can find out our public presents so we can communicate public presents to someone and they communicate with us.

#### Explanation how work STUN

- We create a packet and i want to request my STUN server. first user create a package, that  destination is 9.9.9.9, port is 3478 and may ip and port is 10.0.0.2:8992.

    ![stun-request](./images/stun-request.png)


- Sended to the router. Router just remapping the packet and store data in NAT table.

    ![remapping-packet](./images/remapping-packet.png)

- That request send to the server and server check it this is a STUN request then take your public presence is 5.5.5.5:3333 pushing into a packet and send back to a server.

    ![stun-server-remapping](./images/stun-server-remapping.png)

- Server check the NAT table.

    ![stun-response-remapping-top](./images/stun-response-remapping-top.png)

- The router is change the packet information through the NAT Table.⬇⬇⬇⬇⬇⬇⬇⬇⬇⬇⬇

    ![stun-response-remapping-bottom](./images/stun-response-remapping-bottom.png)

- And lost this information 5.5.5.5:3333. the packet is encrypted through the TLS and the packet information is send to the user.

    ![stun-request-response-back-to-user](./images/stun-request-response-back-to-user.png)


#### STUN when it work

- Both devices have a separate router and request to the server through full-cone NAT. This is use full-cone NAT mean allow every one when then is at least established the communication with the server. 
I am communicate to the server with send a packet and your presents is save here. when i talk to any person check the my presents available or not in the destination user router. mean my presents stored that user router and that user presents stored in my router. They will have matching addresses so their addresses will matched then can send information before that i can't communicate.
That user connected each other shortest path with lowest latency.

    ![stun-when-it-work](./images/stun-when-it-work.png)


#### STUN when it doesn't work

- When presents is not matched the not connected to each other.

    ![stun-when-it-does-not-work](./images/stun-when-it-does-not-work.png)
___

### Traversal Using Relays around (TURN)
- In case of Symmetric NAT use TURN
- It's little bit lighter weight just looks a layer for information most of the time do deep packet inspection like 7 proxy
- TURN default server port 3778, 5349 for TLS
- Expensive to maintain and run

#### How to work turn

Both users use behind symmetric NAT. first user communicate another user through server. send message with packet, server check destination ip and port and send response to another user because this is established a trusted NAT safe communication between them and response back to server and server send back to user.

![turn](./images/turn.png)

___

### Interactive Connectivity Establishment (ICE)
- ICE collects all available candidates (local IP addresses, reflexive addresses - STUN ones and relayed addresses)
- Called ice candidates
- All the collected addresses are then send to the remote peer via SDP

bunch of turn bunch or stun so many options and how i pick appropriate options or just collect all this options. this is not a fancy way. ICE is collect all the information and throw into SDP

___

### Session Description Protocol (SDP)

- SDP is describe the ICE candidates, describe the networking options, describe the media options just have a security options and more...
- SDP is a file.
- main work: take the sdp generated by the user and send the other third party.

#### SDP Example
```
v=0 
o=- 487255629242026503 2 IN IP4 127.0.0.1 
s=- 
t=0 0 

a=group:BUNDLE audio video 
a=msid-semantic: WMS 6x9ZxQZqpo19FRr3Q0xsWC2JJ1lVsk2JE0sG 
m=audio 9 RTP/SAVPF 111 103 104 9 0 8 106 105 13 126 
c=IN IP4 0.0.0.0

a=rtcp:9 IN IP4 0.0.0.0 
a=ice-ufrag:8a1/LJqQMzBmYtes 
a=ice-pwd:sbfskHYHACygyHW1wVi8GZM+ 
a=ice-options:google-ice 
a=fingerprint:sha-256 28:4C:19:10:97:56:FB:22:57:9E:5A:88:28:F3:04:
   DF:37:D0:7D:55:C3:D1:59:B0:B2:81 :FB:9D:DF:CB:15:A8 
a=setup:actpass 
a=mid:audio 
a=extmap:1 urn:ietf:params:rtp-hdrext:ssrc-audio-level 
a=extmap:3 http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time
...
```
___

### Signaling

- SDP signaling
- Send the SDP generated string data file to the other party.
- which type of network to send data to third party, don't matter.