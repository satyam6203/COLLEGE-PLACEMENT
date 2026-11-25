# 🌐 CN Interview Questions (1–100)

## 1. What is a computer network?

**Answer:**
A computer network is two or more devices connected together, either through cables or wirelessly, so they can exchange data and share resources. The devices talk to each other using agreed rules called protocols, like TCP/IP. Simply put, a network is what lets one machine talk to another machine.

**Example:**
Your laptop connects to a Wi-Fi router, the router connects to your ISP, and through that ISP you reach Google. That's a network of millions of devices working together.

**Interview Follow-up:**
- What is the difference between a LAN and a WAN?
- Why do we need protocols like TCP/IP?
- What is a client-server model?

**Short Interview Answer:**
A computer network is a set of interconnected devices that exchange data using common rules called protocols. A home Wi-Fi setup or the internet itself is a network.

---

## 2. What are the types of networks (LAN, MAN, WAN, PAN)?

**Answer:**
These are networks classified by their geographic size. LAN covers a small area like an office, MAN covers a city, WAN covers a country or the whole world, and PAN covers a very small personal area. The bigger the area, the more powerful and expensive the network hardware needed.

**Example:**
Your home Wi-Fi is a LAN. A company's private network connecting all its branches across India is a WAN. Your Bluetooth headset connected to your phone is a PAN.

| Type | Full Form | Area | Example |
|------|-----------|------|---------|
| LAN | Local Area Network | Room / building | Office, home Wi-Fi |
| MAN | Metropolitan Area Network | City | City-wide cable TV network |
| WAN | Wide Area Network | Country / Globe | Internet, private MPLS network |
| PAN | Personal Area Network | ~10 meters | Bluetooth devices |

**Interview Follow-up:**
- Is the internet a LAN or a WAN?
- What is a VPN and how is it different from a private WAN?
- Which is more secure, LAN or WAN, and why?

**Short Interview Answer:**
Networks are classified by area — LAN in a building, MAN in a city, WAN across a country or globally, and PAN for personal devices like Bluetooth. The bigger the area, the more expensive the setup.

---

## 3. What are network topologies (bus, star, ring, mesh, hybrid)?

**Answer:**
Topology is the physical or logical arrangement of how devices are connected in a network. In bus, all devices share one cable; in star, all connect to a central switch; in ring, each device connects to two neighbors forming a loop; in mesh, devices connect to many others for high reliability. In real offices you usually see a hybrid — a star network of switches connected in a mesh.

**Example:**
Old coax-cable office networks used bus topology. Today's offices use star topology with a switch per floor, and multiple switches linked together, which is a hybrid.

| Topology | Shape              | Pros                                         | Cons                                             |
| -------- | ------------------ | -------------------------------------------- | ------------------------------------------------ |
| Bus      | Single shared line | Cheap, simple                                | Single failure breaks everything, collisions     |
| Star     | Central switch     | Easy to add devices, one failure is isolated | Needs more cable, hub is single point of failure |
| Ring     | Closed loop        | Data flows in one direction, no collisions   | One node/link failure breaks the ring            |
| Mesh     | Many-to-many       | Very reliable, high fault tolerance          | Expensive, complex cabling                       |
| Hybrid   | Mix of two or more | Scalable for large networks                  | Complex to design and manage                     |

**Interview Follow-up:**
- Which topology is used in modern LANs and why?
- What is a logical vs physical topology?
- How does topology affect network performance?

**Short Interview Answer:**
Topology is the layout of how devices are connected. Star is most common in offices because it's easy to manage and one cable failure doesn't affect others, while mesh gives the highest reliability at a higher cost.

---

## 4. What is the difference between circuit switching and packet switching?

**Answer:**
In circuit switching a dedicated physical path is reserved for the whole conversation, so you get guaranteed bandwidth but the path is blocked even if idle. In packet switching, data is split into small packets that each find their own way and share the link, which is far more efficient but with no guarantee. The internet and all modern networks use packet switching; the old telephone system used circuit switching.

**Example:**
A traditional phone call is circuit switching — the path stays reserved for the entire call. Streaming a YouTube video is packet switching — many small packets share the network with everyone else's traffic.

| Feature | Circuit Switching | Packet Switching |
|---------|-------------------|------------------|
| Path | Dedicated, reserved | Shared, no reservation |
| Setup | Needed before data | No setup needed |
| Bandwidth | Fixed for the call | Variable, dynamic |
| Efficiency | Low if link is idle | High |
| Failure | Connection breaks completely | Packets reroute automatically |
| Example | Old phone network | Internet, WhatsApp |

**Interview Follow-up:**
- Why is packet switching better for the internet?
- What is statistical multiplexing?
- What happens if a packet is lost in a circuit-switched network?

**Short Interview Answer:**
Circuit switching reserves a dedicated path for the whole call, while packet switching breaks data into packets that share the link dynamically. Packet switching is more efficient, which is why the internet uses it.

---

## 5. What is an IP address, MAC address, and port number?

**Answer:**
An IP address is the logical address of a device on a network, used to route data across networks. A MAC address is the physical hardware address burned into the network card, used to identify a device inside a single local network. A port number identifies a specific process or service on that machine, so the data reaches the right application.

**Example:**
When you call a website, the IP address gets the packet to the server's machine, the MAC address gets it to the right network card on your local LAN, and the port number (like 443 for HTTPS) gets it to the right process running on the server.

| Address | Type | Layer | Example | Scope |
|---------|------|-------|---------|-------|
| MAC | Physical / Hardware | Layer 2 | `A4:5E:60:C1:22:3F` | Local LAN only |
| IP | Logical | Layer 3 | `192.168.1.10` | Across networks |
| Port | Transport | Layer 4 | `8080` | Process on the machine |

**Common Misconception:**
A MAC address is not permanent internet identity — it only works within your local network and doesn't cross routers.

**Interview Follow-up:**
- Which address changes when you move to a different network?
- What is the default port for HTTP and HTTPS?
- How does the router find the MAC address of a device?

**Short Interview Answer:**
MAC is the physical hardware address for a local network, IP is the logical address to route data across networks, and the port number points to the specific service on the machine. All three work together to deliver a packet to the right process.

---

## 6. What is the difference between IPv4 and IPv6?

**Answer:**
IPv4 uses 32 bits, which gives about 4.3 billion addresses, and that pool is now exhausted. IPv6 uses 128 bits, giving an almost unlimited number of addresses, and it also brings built-in auto-configuration and better security support. In short, IPv6 exists mainly because we ran out of IPv4 addresses.

**Example:**
Your home router may give your phone an IPv6 address like `240e:3b1:...` while still using a private IPv4 address internally. Most networks today run both together, called dual stack.

| Feature | IPv4 | IPv6 |
|---------|------|------|
| Address size | 32 bits | 128 bits |
| Format | `192.168.1.1` | `240e:3b1:5f::1` |
| Total addresses | ~4.3 billion | ~3.4 × 10³⁸ |
| Header | Variable, complex | Fixed, simplified |
| Configuration | Manual or DHCP | SLAAC auto-config |
| Fragmentation | Routers allowed | Only sender allowed |
| Security | Optional (IPsec) | Built-in IPsec |

**Interview Follow-up:**
- What is NAT and why is it needed with IPv4?
- What is dual stack?
- Is IPv6 fully replacing IPv4 today?

**Short Interview Answer:**
IPv4 has 32-bit addresses and is exhausted, so IPv6 was introduced with 128-bit addresses giving practically unlimited space, plus simpler headers and built-in security. Most systems today run both, called dual stack.

---

## 7. What is subnetting and why is it used?

**Answer:**
Subnetting means splitting one big network into smaller sub-networks by borrowing bits from the network portion of the IP address. It is mainly used for better organization, reduced broadcast traffic, and improved security. In simple words, it lets one organization split its address block into separate departments and control traffic between them.

**Example:**
A company with the block `192.168.0.0/16` can give `192.168.1.0/24` to the HR department and `192.168.2.0/24` to the accounts department, so HR traffic stays local and doesn't flood the whole network.

**Interview Follow-up:**
- What is CIDR notation?
- What is the subnet mask and default gateway?
- How do you calculate the number of usable hosts in a subnet?

**Short Interview Answer:**
Subnetting divides a big network into smaller subnets using subnet masks. It reduces broadcast traffic, improves organization, and adds a layer of security between departments.

---

## 8. What is the difference between hub, switch, and router?

**Answer:**
A hub is a Layer 1 device that just repeats incoming data to every other port, so everyone shares bandwidth and gets collisions. A switch is a Layer 2 device that learns MAC addresses and sends data only to the exact port needed, giving each port its own bandwidth. A router is a Layer 3 device that connects different networks and decides the best path for data using IP addresses.

**Example:**
An old office might use a hub so every PC hears all traffic. Replacing it with a switch immediately makes the network faster. A router then connects that office LAN to the internet.

| Device | Layer | Works On | Main Job |
|--------|-------|----------|----------|
| Hub | Layer 1 | Electrical signals | Repeat to all ports |
| Switch | Layer 2 | MAC addresses | Forward frames to the right port |
| Router | Layer 3 | IP addresses | Route packets between different networks |

**Common Misconception:**
A switch does not replace a router — you still need a router to connect two different networks.

**Interview Follow-up:**
- What is the difference between a switch and a managed switch?
- Does a switch work on IP or MAC addresses?
- Can a Layer 2 switch route between networks?

**Short Interview Answer:**
A hub blindly repeats signals to all ports, a switch forwards frames to the correct port using MAC addresses, and a router moves packets between different networks using IP addresses. Switch = local efficiency, router = inter-network communication.

---

## 9. What is DNS and how does it work?

**Answer:**
DNS is the Domain Name System, which translates human-friendly names like `google.com` into IP addresses like `142.250.185.78`. The browser first checks its own cache, then asks the resolver, which may check the browser cache, OS cache, then DNS servers, root server, TLD server, and finally the authoritative server. Once the IP is found, normal HTTP communication starts.

**Example:**
When you type `github.com`, your OS resolver asks the local ISP DNS server, which forwards to a root server, then to the `.com` TLD server, then to GitHub's nameserver, and the final answer is GitHub's IP address.

**Interview Follow-up:**
- What is a DNS cache and why does it exist?
- What is DNS load balancing?
- What causes "DNS_PROBE_FINISHED_NXDOMAIN"?

**Short Interview Answer:**
DNS converts domain names into IP addresses. The lookup goes browser cache → OS cache → resolver → root → TLD → authoritative server, and the result is usually cached to make future lookups instant.

---

## 10. What is DHCP?

**Answer:**
DHCP is Dynamic Host Configuration Protocol, and it automatically assigns IP address, subnet mask, default gateway, and DNS server to devices when they join a network. The device first sends a DISCOVER broadcast, a DHCP server replies with an OFFER, the client requests it, and the server confirms it. This is why you never manually type an IP address when connecting to a new Wi-Fi.

**Example:**
When your phone joins a café Wi-Fi, the router's DHCP server gives it something like `192.168.1.42/24` along with the gateway and DNS automatically.

| DHCP Message | Meaning |
|--------------|---------|
| DORA — Discover | "Is there any DHCP server out there?" |
| DORA — Offer | Server offers an IP address |
| DORA — Request | Client asks for that specific IP |
| DORA — Ack | Server confirms the lease |

**Interview Follow-up:**
- What is a DHCP lease and how long does it last?
- What happens if no DHCP server responds — what is APIPA?
- What is DHCP relay and why is it needed?

**Short Interview Answer:**
DHCP automatically assigns IP addresses and network settings to devices using the DORA process — Discover, Offer, Request, Ack. That's why connecting to Wi-Fi requires no manual configuration.

---

## 11. Explain the OSI model and its 7 layers.

**Answer:**
The OSI model is a 7-layer reference framework that explains how data travels from one device to another. Each layer has a specific job, and data moves down the layers on the sender, across the wire, and back up on the receiver. It is mainly used to understand and troubleshoot networking problems, not as a literal protocol stack.

**Example:**
When you load a webpage, your data goes from Application → Transport → Network → Data Link → Physical, crosses the network, and comes back up in reverse on the server.

| Layer | Name | Job | Example Protocols | Data Unit |
|-------|------|-----|-------------------|----------|
| 7 | Application | Services to the user | HTTP, FTP, SMTP, DNS | Data |
| 6 | Presentation | Encoding, encryption, compression | SSL/TLS, JPEG | Data |
| 5 | Session | Establishes, manages, terminates sessions | NetBIOS, RPC | Data |
| 4 | Transport | End-to-end delivery, reliability | TCP, UDP | Segment |
| 3 | Network | Logical addressing and routing | IP, ICMP, OSPF | Packet |
| 2 | Data Link | MAC addressing, error detection | Ethernet, PPP | Frame |
| 1 | Physical | Raw bit transmission | Cable, fiber, radio | Bits |

**Mnemonic (top to bottom):**
**A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing.

**Interview Follow-up:**
- Which layer does a router work on?
- Why was the OSI model created if TCP/IP is actually used?
- Which layers are involved when a browser loads a page?

**Short Interview Answer:**
OSI is a 7-layer model — Application, Presentation, Session, Transport, Network, Data Link, Physical — where each layer handles one specific job. Data flows down on the sender side and up on the receiver side, and it helps us isolate where a network problem occurs.

---

## 12. Explain the TCP/IP model.

**Answer:**
The TCP/IP model is the 4 or 5 layer model that the internet actually uses, and OSI was derived from it. It has Application, Transport, Internet, and Network Access layers, with the Application layer merging the OSI Application, Presentation, and Session layers. It is more practical and implementation-focused than OSI.

**Example:**
When your Spring Boot app calls an external API over HTTPS, the data is built in the Application layer, handed to TCP, then IP, then Ethernet/Wi-Fi at the last layer.

| TCP/IP Layer | Maps To OSI | Protocols |
|--------------|--------------|-----------|
| Application | 7, 6, 5 | HTTP, DNS, SMTP, FTP |
| Transport | 4 | TCP, UDP |
| Internet | 3 | IP, ICMP, ARP |
| Network Access | 2, 1 | Ethernet, Wi-Fi, PPP |

**Interview Follow-up:**
- Why is the TCP/IP model called the practical model?
- How many layers does TCP/IP have — 4 or 5?
- What does the Network Access layer combine?

**Short Interview Answer:**
TCP/IP is the practical 4-layer model the internet actually follows, and it merges OSI's Application, Presentation, and Session into one Application layer. It is more realistic than OSI, which is mainly a teaching reference.

---

## 13. What is the difference between OSI and TCP/IP models?

**Answer:**
OSI is a theoretical 7-layer model used for learning, while TCP/IP is a practical 4-layer model used by real protocols. OSI separates presentation and session layers, but TCP/IP merges them into the Application layer. TCP/IP is the model the internet is built on, so it's what you actually configure and debug with.

**Example:**
When a packet is small, the IP header at Layer 3 takes 20 bytes, which is 22% overhead — something you see in real networks, not just in textbooks.

| Feature | OSI | TCP/IP |
|---------|-----|--------|
| Layers | 7 | 4 or 5 |
| Nature | Theoretical | Practical |
| Used by | Textbooks, theory | Real internet protocols |
| Session/Presentation | Separate layers | Merged into Application |
| Transport | Separate | TCP and UDP |
| Protocol independence | Model is fixed | Model came from protocols |

**Interview Follow-up:**
- Which model is more relevant for a developer and why?
- How does OSI's Layer 4 map to TCP/IP?
- Why do companies still teach OSI?

**Short Interview Answer:**
OSI is a 7-layer theoretical model, TCP/IP is the 4-layer model actually implemented by real protocols. TCP/IP merges OSI's session, presentation, and application layers into one, which is why it maps more closely to how the internet works.

---

## 14. What are the functions of each OSI layer?

**Answer:**
Each OSI layer has a specific responsibility, from serving the user down to sending raw bits on the wire. Layers 1–3 handle physical transmission, addressing, and routing, while layers 4–7 handle reliable delivery, sessions, and data meaning. Knowing which layer does what is the fastest way to pinpoint where a problem is.

**Example:**
If you can ping the IP but the website doesn't load, the issue is likely at the Application layer (HTTP/DNS), not at the Network layer.

| Layer | Function |
|-------|----------|
| Application | Provides network services directly to apps (HTTP, DNS, SMTP) |
| Presentation | Translates data format, handles encoding, compression, encryption |
| Session | Creates, manages, and closes sessions between applications |
| Transport | Reliable end-to-end delivery, segmentation, flow/congestion control |
| Network | Logical addressing, routing, packet forwarding (IP) |
| Data Link | Physical addressing (MAC), framing, error detection, media access |
| Physical | Transmits raw bits as electrical, light, or radio signals |

**Interview Follow-up:**
- Which layer is responsible for error detection?
- Where does routing happen and where does MAC addressing happen?
- How do the layers interact when sending a file?

**Short Interview Answer:**
Layers 4 to 7 manage the meaning of data and reliable delivery, while layers 1 to 3 handle physical transmission, MAC addressing, and IP routing. Knowing the layer helps you isolate whether a problem is physical, network-level, or application-level.

---

## 15. At which layer does encryption and decryption occur?

**Answer:**
In the OSI model, encryption and decryption are handled at the Presentation layer (Layer 6). In the TCP/IP model, this same job is done inside the Application layer, which is why protocols like SSL/TLS and HTTPS are considered application-layer protocols. HTTPS is actually HTTP data encrypted at the presentation level before it is sent over TCP.

**Example:**
When you visit `https://flipkart.com`, TLS encrypts the HTTP request and response, then the encrypted bytes travel over TCP, IP, and Ethernet as normal data.

| Where | Layer | Examples |
|-------|-------|----------|
| OSI | Layer 6 — Presentation | SSL/TLS, DES, AES, JPEG |
| TCP/IP | Application layer | HTTPS, SSH, PGP |
| IPsec (VPN) | Layer 3 — Network | IPsec tunnel/transport mode |

**Common Misconception:**
Encryption does not only happen at Layer 6 — IPsec encrypts at Layer 3, and application-layer encryption like E2EE happens in Layer 7.

**Interview Follow-up:**
- What is the difference between SSL/TLS encryption and IPsec?
- Does TLS encrypt at the transport layer or application layer?
- What is end-to-end encryption?

**Short Interview Answer:**
In OSI, encryption happens at the Presentation layer (Layer 6). In the TCP/IP model it falls under the Application layer, which is why HTTPS is treated as an application-layer protocol.

---

## 16. At which layer do switches, routers, and gateways operate?

**Answer:**
A switch works at Layer 2 and uses MAC addresses, while a router works at Layer 3 and uses IP addresses. A gateway operates at the top layer, usually Layer 7, and translates between two completely different protocol systems. So the device's layer tells you what kind of decision it is making.

**Example:**
An L2 switch in an office only forwards Ethernet frames inside the LAN. To reach another network you need a router, which is where the default gateway address on your machine points.

| Device | Layer | Address Type | Role |
|--------|-------|--------------|------|
| Repeater / Hub | Layer 1 | None | Regenerate signals |
| Switch | Layer 2 | MAC | Forward frames inside LAN |
| Router | Layer 3 | IP | Route packets between networks |
| Modem | Layer 1 / 2 | — | Convert digital to analog signal |
| Gateway | Layer 7 (usually) | Application-level | Translate between different protocols |

**Interview Follow-up:**
- Can a router act as a Layer 2 switch?
- What is a Layer 3 switch and where is it used?
- What does "default gateway" mean exactly?

**Short Interview Answer:**
Switches operate at Layer 2 using MAC addresses, routers at Layer 3 using IP addresses, and gateways at Layer 7 to translate between different protocols. The layer tells you whether the device handles local delivery or routing between networks.

---

## 17. What is the difference between session layer and transport layer?

**Answer:**
The session layer manages the conversation between two applications — establishing, maintaining, and terminating sessions, and handling checkpoints. The transport layer is responsible for delivering data reliably from one end to the other, with segmentation, sequencing, error control, and flow control. In short, session layer manages the *dialogue*, transport layer manages the *data delivery*.

**Example:**
When you use a video call, the session layer sets up and manages the call session, while the transport layer (TCP) ensures the video frames arrive in order without loss.

| Feature | Session Layer (5) | Transport Layer (4) |
|---------|--------------------|---------------------|
| Purpose | Manage sessions | Reliable data delivery |
| Functions | Establish, sync, terminate | Segmentation, sequencing, ACK, flow control |
| Focus | Applications talking to each other | End-to-end process communication |
| Protocols | NetBIOS, RPC, SQL sessions | TCP, UDP |
| Example | Login session in a web app | TCP ensuring the request reaches the server |

**Interview Follow-up:**
- Which layer performs end-to-end communication?
- Is the session layer implemented separately in modern systems?
- What is the role of port numbers in the transport layer?

**Short Interview Answer:**
The session layer manages the conversation between applications, while the transport layer ensures reliable end-to-end delivery of data. Session layer is about the dialogue; transport layer is about how the data actually reaches the other side safely.

---

## 18. What is encapsulation and decapsulation?

**Answer:**
Encapsulation is when data moves down the sender's OSI layers, and each layer adds its own header to the data before passing it down. Decapsulation is the reverse — as data reaches the receiver, each layer removes its header and passes the data up. This is the core mechanism of how networking works.

**Example:**
Your HTTP data becomes a TCP segment, then an IP packet, then an Ethernet frame, and finally raw bits on the cable. At the server the headers are peeled off in reverse order.

**Encapsulation (sender):**
```
Application Data
  → Transport (adds TCP/UDP header)  = Segment
  → Network (adds IP header)         = Packet
  → Data Link (adds MAC header)      = Frame
  → Physical (transmits bits)
```

**Decapsulation (receiver):**
```
Physical bits → Frame (strip MAC) → Packet (strip IP) → Segment (strip TCP) → Data
```

**Interview Follow-up:**
- Which layer performs encapsulation first?
- How does header size affect network performance?
- What is MTU and how does it relate to encapsulation?

**Short Interview Answer:**
Encapsulation is adding a header at each layer as data goes down on the sender side, and decapsulation is removing those headers as data goes up on the receiver side. So an HTTP request becomes a segment, a packet, a frame, and then bits.

---

## 19. What is the difference between connection-oriented and connectionless communication?

**Answer:**
In connection-oriented communication, both sides first set up a connection, exchange data over it, and then close it — like a phone call. In connectionless communication, data is sent without any setup, and each packet is independent — like sending a letter. Connection-oriented gives reliability and ordering, connectionless is faster but no guarantee.

**Example:**
A bank transaction uses TCP (connection-oriented) because losing or reordering data is unacceptable. A live video call or online game uses UDP (connectionless) because a slightly dropped frame is better than waiting.

| Feature | Connection-Oriented | Connectionless |
|---------|---------------------|-----------------|
| Setup | Required before data | No setup |
| Example | Phone call | Letter / postcard |
| Reliability | Guaranteed | Not guaranteed |
| Order | In order | May arrive out of order |
| Overhead | Higher (handshake) | Lower |
| Protocol | TCP | UDP |

**Interview Follow-up:**
- Why does TCP need a 3-way handshake?
- Can UDP be made reliable? How?
- Which one is better for a video streaming app and why?

**Short Interview Answer:**
Connection-oriented communication sets up a connection first and is reliable, like a phone call, while connectionless just sends packets independently, like posting a letter. TCP is connection-oriented, UDP is connectionless.

---

## 20. What is a Protocol Data Unit (PDU)?

**Answer:**
A PDU is the specific unit of data exchanged at a particular layer of the network — the data has a different name and format at each layer. A file at the Application layer becomes a segment at Transport, a packet at Network, and a frame at Data Link. Knowing the correct term for each layer helps you talk precisely in interviews and read documentation correctly.

**Example:**
When Wireshark captures traffic, you see different PDU names at each level — Frame, IP Packet, TCP Segment, and HTTP Data. That's the same data described at different layers.

| Layer | PDU Name | Description |
|-------|----------|-------------|
| Application / Presentation / Session | Data / Message | Actual user data |
| Transport | Segment (TCP) / Datagram (UDP) | Transport header added |
| Network | Packet | IP header added |
| Data Link | Frame | MAC header + trailer added |
| Physical | Bits | Raw bit transmission |

**Interview Follow-up:**
- What is an MTU and how does it affect PDU size?
- Why is a frame slightly larger than a packet?
- What is the maximum size of a TCP segment typically?

**Short Interview Answer:**
A PDU is the unit of data at a given layer — data, segment, packet, frame, bits. As we move down the layers, headers are added, and the PDU name changes accordingly.

## 21. What is the difference between TCP and UDP?

**Answer:**
TCP is connection-oriented, so it does a handshake, guarantees delivery, ordering, and error recovery. UDP is connectionless, so it sends data without setup and with no guarantee of delivery or order, but it is much faster and lighter. Choose TCP when you cannot lose data (HTTP, email, banking), and UDP when speed matters more than perfect delivery (live video, gaming, DNS).

**Example:**
Loading a webpage uses TCP because you need every byte of HTML and CSS in the right order. A Zoom call uses UDP because a dropped frame is better than a stalled stream.

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Required (handshake) | Not required |
| Reliability | Guaranteed | Not guaranteed |
| Order | In sequence | May arrive out of order |
| Speed | Slower, more overhead | Fast, minimal overhead |
| Header size | 20 bytes | 8 bytes |
| Error control | Yes (retransmission) | Optional checksum only |
| Flow/congestion control | Yes | No |
| Use cases | HTTP, HTTPS, FTP, SMTP, SSH | Video call, DNS, DHCP, online games |

**Common Misconception:**
UDP is not "bad" — it just makes different trade-offs. Also, TCP is not always slower in real networks; it can be faster than UDP over lossy links because it recovers quickly.

**Interview Follow-up:**
- What is head-of-line blocking and why does it affect TCP more?
- Can you make UDP reliable? How?
- When would you choose UDP in a backend service?

**Short Interview Answer:**
TCP is reliable, ordered, and connection-oriented with a 20-byte header, while UDP is fast, connectionless, and has only an 8-byte header with no delivery guarantee. I use TCP for HTTP and email, and UDP for real-time audio/video and DNS.

---

## 22. What is a socket?

**Answer:**
A socket is basically one end of a network connection — it is the combination of an IP address, a port number, and a protocol that an application uses to send or receive data. When a client and server talk, each side has its own socket. In Java you create sockets with `ServerSocket` and `Socket`, which is the low-level API behind higher-level frameworks.

**Example:**
When your browser opens `https://site.com`, the browser opens a socket to port 443 on the site's IP. The web server accepts a socket on port 8080 and responds through the same connection.

| Socket Type | Behaviour | Java Class |
|-------------|-----------|------------|
| Stream (TCP) | Reliable, ordered byte stream | `Socket` / `ServerSocket` |
| Datagram (UDP) | Connectionless, message based | `DatagramSocket` |

**Interview Follow-up:**
- What is a 4-tuple and 5-tuple connection?
- What is a connection pool and why is it needed?
- How does the OS keep track of millions of open sockets?

**Short Interview Answer:**
A socket is one end of a network connection, identified by IP address, port, and protocol. In Java, `Socket` and `ServerSocket` handle TCP, and `DatagramSocket` handles UDP.

---

## 23. Explain the TCP 3-way handshake.

**Answer:**
TCP establishes a connection using three steps — SYN, SYN-ACK, ACK — and until all three are exchanged, no data is transferred. The client sends SYN with a random sequence number, the server replies SYN-ACK with its own sequence number, and the client confirms with ACK. This handshake also synchronizes sequence numbers so both sides know where to start and where to retransmit from.

**Example:**
When you open any website, your browser sends SYN to the server, the server replies SYN-ACK, the browser sends ACK, and then the HTTP request goes out. That's the handshake.

| Step | Sender | Message | Contains |
|------|--------|---------|----------|
| 1 | Client → Server | SYN | Client's initial sequence number |
| 2 | Server → Client | SYN + ACK | Server's sequence number + ACK for client |
| 3 | Client → Server | ACK | Acknowledges server's sequence number |

**Common Misconception:**
The handshake is not for security — it only synchronizes state. Encryption is a separate step, done later with TLS.

**Interview Follow-up:**
- Why is it called 3-way and not 2-way?
- What is the SYN flood attack?
- What are SYN cookies and how do they help?

**Short Interview Answer:**
TCP uses a 3-way handshake — SYN, SYN-ACK, ACK — to establish a connection and synchronize sequence numbers before any data is sent. It is not a security mechanism, just a way to confirm both sides are ready.

---

## 24. What is TCP 4-way termination?

**Answer:**
Closing a TCP connection needs four steps because each side closes its own direction independently. The client sends FIN, the server ACKs it, then the server sends its own FIN, and the client sends a final ACK. The extra step compared to the handshake is because a FIN and its ACK cannot be combined into one message in the same way SYN and SYN-ACK can.

**Example:**
When you close a browser tab, your machine sends FIN to the server, the server ACKs, and the server separately sends FIN back before the connection is fully closed.

| Step | Sender | Message |
|------|--------|---------|
| 1 | Client → Server | FIN (client done sending) |
| 2 | Server → Client | ACK |
| 3 | Server → Client | FIN (server done sending) |
| 4 | Client → Server | ACK (connection fully closed) |

**Interview Follow-up:**
- What is TIME_WAIT and why is it needed?
- What is a half-open connection?
- How does a server handle thousands of closing connections?

**Short Interview Answer:**
TCP teardown takes four steps — FIN, ACK, FIN, ACK — because each side closes its own send direction separately. The last ACK leaves the client in TIME_WAIT state for a while to handle any retransmissions.

---

## 25. What is flow control in TCP?

**Answer:**
Flow control is about keeping the sender from overwhelming the receiver. TCP uses a sliding window with ACKs, and each ACK tells the sender how many bytes the receiver can accept next, called the receive window or `rwnd`. If the buffer is full, the receiver advertises a window of zero and the sender pauses until space is available.

**Example:**
If your laptop's receive buffer is full, it advertises a zero window, and the server stops sending data until your application reads some data and the window opens again.

**Interview Follow-up:**
- How is flow control different from congestion control?
- What is the zero-window probe?
- What causes the window to shrink?

**Short Interview Answer:**
Flow control protects the receiver by using the TCP receive window to tell the sender how much data it can accept. If the receiver's buffer is full, it advertises a zero window and the sender stops.

---

## 26. What is congestion control in TCP?

**Answer:**
Congestion control protects the network rather than the receiver, and it is about not sending more data than the links in the path can carry. TCP detects congestion through packet loss or increased delay, and it reacts by reducing the sending window. The main algorithms are slow start, congestion avoidance, and fast retransmit/fast recovery, plus mechanisms like AIMD.

**Example:**
If you download a file and packets start getting dropped because the link is full, TCP halves its sending rate instead of continuing to blast data.

| Technique | Behaviour |
|-----------|-----------|
| Slow start | Window grows exponentially at the start |
| Congestion avoidance | Window grows linearly once steady |
| Fast retransmit | Resend after 3 duplicate ACKs |
| Fast recovery | Avoid full restart after minor loss |
| AIMD | Additive increase, multiplicative decrease |

**Common Misconception:**
Flow control protects the *receiver*, congestion control protects the *network*. Mixing these two up is a very common interview mistake.

**Interview Follow-up:**
- What is the congestion window (`cwnd`) and how does it differ from `rwnd`?
- What is BDP and how do you calculate the ideal window?
- How does QUIC handle congestion control differently from TCP?

**Short Interview Answer:**
Congestion control stops the sender from overloading the network, and TCP detects this through loss or delay and reduces its window. Slow start, congestion avoidance, and AIMD are the main techniques used.

---

## 27. What are sliding window protocols?

**Answer:**
A sliding window protocol allows the sender to keep multiple unacknowledged packets in flight instead of waiting after every single packet. The window is a range of sequence numbers the sender is allowed to send, and it slides forward as ACKs are received. This keeps the pipeline full and makes TCP actually fast over high-latency links.

**Example:**
If the window size is 4, the sender can send 4 packets back-to-back and then send one more for every ACK received, without waiting for each individually.

| Term | Meaning |
|------|---------|
| Window size | Max unacknowledged bytes the sender can have |
| Sliding | Window moves forward as ACKs arrive |
| Sender window | Sequence numbers the sender may transmit |
| Receiver window | Sequence numbers the receiver can accept |

**Interview Follow-up:**
- What is the window scaling option and why is it needed?
- How is the window size decided?
- What is a zero window and when does it occur?

**Short Interview Answer:**
A sliding window lets the sender keep several packets unacknowledged and push more forward as ACKs come back, instead of waiting for each one. That's what makes TCP efficient over long-distance links.

---

## 28. What is piggybacking in TCP?

**Answer:**
Piggybacking is sending an ACK along with the next data packet instead of sending the ACK separately. Since every data packet already carries a header, adding the ACK number to it costs almost nothing and saves an entire packet. It's one of the small efficiency tricks that makes TCP perform well.

**Example:**
When you are downloading a file, the client often sends its next data request along with the ACK for previously received data, instead of a standalone ACK packet.

**Interview Follow-up:**
- When is a delayed ACK used?
- What is Nagle's algorithm and how does it relate?
- Why is piggybacking not always possible?

**Short Interview Answer:**
Piggybacking means attaching an ACK to the outgoing data packet instead of sending the ACK separately. It saves bandwidth and reduces the number of packets on the network.

---

## 29. What is the difference between HTTP and HTTPS?

**Answer:**
HTTPS is simply HTTP running over TLS, which encrypts the data and authenticates the server using a certificate. The requests, methods, and status codes are identical — only the transport is secure. The difference is visible in the URL and the default port: HTTP uses port 80 and HTTPS uses port 443.

**Example:**
When you log in to a banking site, HTTPS encrypts your username, password, and card details so an attacker on the same Wi-Fi cannot read them in transit.

| Feature | HTTP | HTTPS |
|---------|-------|--------|
| Port | 80 | 443 |
| Encryption | None | TLS-based |
| Certificate | Not needed | Required |
| Data in transit | Plain text | Encrypted |
| Speed | Slightly faster | Minimal overhead with HTTP/2 and HTTP/3 |
| Browser indicator | "Not secure" | Padlock icon |

**Common Misconception:**
HTTPS does not make the server itself secure — it only protects data in transit. A server can still be hacked.

**Interview Follow-up:**
- What happens during a TLS handshake?
- What is a certificate pinning?
- Can HTTP/3 run on HTTP or HTTPS?

**Short Interview Answer:**
HTTPS is HTTP encrypted with TLS, using port 443 and a certificate, while plain HTTP uses port 80 with no encryption. Everything else stays the same, so the difference is purely the security layer.

---

## 30. What are FTP, TFTP, and SFTP?

**Answer:**
FTP is the standard file transfer protocol but it sends everything in plain text, including passwords, so it is insecure. TFTP is a very lightweight version with no login and no directory listing, used mainly for booting network devices like routers. SFTP is not really a separate protocol — it is just FTP's file transfer capability running over SSH, so it is encrypted and secure.

**Example:**
An admin uploading a router's startup config to a remote server would use SFTP, not FTP, because FTP would send the password in clear text.

| Feature | FTP | TFTP | SFTP |
|---------|-----|------|------|
| Port | 21 | 69 | 22 (via SSH) |
| Encryption | None | None | Yes (SSH) |
| Authentication | User/password | None | Key/password |
| Reliability | Reliable | Not reliable | Reliable |
| Use case | Legacy transfers | Network device boot | Secure file transfer |
| Direction | Two-way | Read-only | Two-way |

**Interview Follow-up:**
- What is the difference between SFTP and FTPS?
- Why is TFTP not used for general file transfer?
- Which port does SFTP use and why is it 22?

**Short Interview Answer:**
FTP transfers files without encryption, TFTP is a minimal unauthenticated version used for booting devices, and SFTP is secure because it runs over SSH. In practice I always use SFTP, or HTTPS APIs, for file transfer.

---

## 31. What are SMTP, POP3, and IMAP?

**Answer:**
SMTP is the protocol used to *send* and relay emails between mail servers. POP3 downloads the mail to your device and usually deletes it from the server, while IMAP keeps the mail on the server and syncs only the read state. That is why IMAP is better if you read mail on multiple devices.

**Example:**
When you send an email from Gmail, SMTP handles it. Your phone syncing mail with the server uses IMAP, so the mail stays on Google's servers.

| Feature | SMTP | POP3 | IMAP |
|---------|------|------|------|
| Purpose | Sending email | Retrieving email | Retrieving + syncing |
| Port | 587 (submission) | 995 (secure) | 993 (secure) |
| Mail stored on server | Yes (temporarily) | Usually deleted | Yes, permanently |
| Multi-device support | Not applicable | Poor | Good |
| Folders on server | No | Limited | Yes |

**Interview Follow-up:**
- What is the difference between POP3 and POP3S?
- Why does email bounce use a DSN / SMTP status code?
- What is an MX record and where is it used?

**Short Interview Answer:**
SMTP sends email, POP3 downloads it to your device, and IMAP keeps it synced on the server. IMAP is the modern choice because it supports multiple devices and folders.

---

## 32. What is DNS resolution?

**Answer:**
DNS resolution is the process of converting a domain name into an IP address by walking through a hierarchy of servers. The resolver checks caches first, then queries the root server, the TLD server, and finally the authoritative nameserver for that domain. The answer is then cached for a period given by the record's TTL.

**Example:**
Asking for `www.amazon.in` goes: your OS → ISP resolver → root server → `.in` TLD server → Amazon's nameserver → returns an IP, which is then cached.

**Common Misconception:**
The root server does not know the final IP — it only points to the correct TLD server. Each step narrows the search.

**Interview Follow-up:**
- What are the different DNS record types (A, CNAME, MX, TXT, NS)?
- What is DNS recursion vs iterative resolution?
- Why does changing DNS take time to propagate?

**Short Interview Answer:**
DNS resolution turns a domain name into an IP by querying the root, TLD, and authoritative servers, with caching at each step for speed. The hierarchy exists so no single server handles all lookups.

---

## 33. What is SSL/TLS and what is its role?

**Answer:**
SSL is the older and now broken protocol, and TLS is its secure replacement — we usually say SSL/TLS but mean TLS. Its job is to provide encryption, authentication of the server through certificates, and integrity checking of the data. TLS 1.3 is the current recommended version, and the handshake uses asymmetric cryptography to agree on keys, then switches to fast symmetric encryption.

**Example:**
When you open an HTTPS site, the TLS handshake verifies the server's certificate, negotiates an encryption key, and after that everything you send is encrypted.

| Version | Status |
|---------|--------|
| SSL 1.0 / 2.0 / 3.0 | Broken, deprecated |
| TLS 1.0 / 1.1 | Deprecated |
| TLS 1.2 | Still used, should be upgraded |
| TLS 1.3 | Current and recommended |

**Interview Follow-up:**
- What is the SSL/TLS handshake?
- Why does TLS use both asymmetric and symmetric encryption?
- What is a man-in-the-middle attack and how does a certificate prevent it?

**Short Interview Answer:**
SSL/TLS provides encryption, server authentication, and data integrity for data in transit. SSL is deprecated, so always use TLS 1.2 or 1.3, which is what HTTPS runs on.

---

## 34. What is QUIC protocol?

**Answer:**
QUIC is a transport protocol built on top of UDP, and it is what HTTP/3 runs on. It gives you TCP's reliability with TLS's security built directly into the protocol, so a separate TLS handshake over TCP is not needed, and this makes connections much faster. It also avoids head-of-line blocking by running multiple independent streams in one connection.

**Example:**
Google Chrome loading a page over HTTP/3 uses QUIC — the connection is established in roughly one round trip instead of the two or three TCP plus TLS needs.

| Feature | TCP + TLS | QUIC |
|---------|----------|------|
| Transport | TCP | UDP |
| Encryption | Separate TLS handshake | Built into handshake |
| Setup | 2–3 round trips | ~1 round trip |
| Head-of-line blocking | Yes, at TCP and TLS | No, per-stream |
| Multiplexing | Needs many connections | Many streams in one connection |
| Used by | HTTP/1.1, HTTP/2 | HTTP/3 |

**Interview Follow-up:**
- How does QUIC handle congestion control?
- Why is QUIC built on UDP instead of TCP?
- What is connection migration in QUIC?

**Short Interview Answer:**
QUIC is a transport protocol over UDP that integrates encryption and multiplexes streams, and it's the basis of HTTP/3. It gives faster connection setup and avoids head-of-line blocking compared to TCP + TLS.

---

## 35. What are WebSockets?

**Answer:**
WebSockets is a protocol that provides full-duplex persistent communication between client and server over one TCP connection. It starts with a normal HTTP request that gets upgraded using the `Upgrade: websocket` header, and after that the connection stays open for pushing data both ways. It solves the problem of HTTP polling, where the client has to repeatedly ask "is there anything new?"

**Example:**
A live cricket score app or a trading dashboard uses WebSockets to push score updates to the browser instantly instead of the client polling every few seconds.

| Feature | HTTP Polling | WebSocket |
|---------|---------------|------------|
| Connection | New request each time | Single persistent connection |
| Direction | Client → Server (mostly) | Full duplex |
| Overhead | High, repeated headers | Low after upgrade |
| Latency | Depends on poll interval | Immediate push |
| Use case | Traditional APIs | Chat, live scores, stock tickers |

**Interview Follow-up:**
- How is a WebSocket different from Server-Sent Events (SSE)?
- What is the difference between WebSocket and Socket.IO?
- What libraries are used for WebSockets in Java Spring Boot?

**Short Interview Answer:**
WebSockets gives a persistent full-duplex connection between client and server, established by upgrading an HTTP request. It's used for real-time updates like chat, live scores, and notifications, instead of polling.

---

## 36. What is ARP?

**Answer:**
ARP stands for Address Resolution Protocol, and it is used to find the MAC address of a device when we only know its IP address. The device broadcasts "who has this IP?" on the local network, and the device with that IP replies with its MAC address, which is then cached in the ARP table. ARP only works inside a local network because broadcasts don't cross routers.

**Example:**
Your laptop wants to reach `192.168.1.1`, so it sends an ARP request broadcast; your router replies with MAC `00:1A:2B:3C:4D:5E`, and your laptop caches that mapping for a few minutes.

**Interview Follow-up:**
- What is an ARP cache and how long does it live?
- What is gratuitous ARP?
- What is ARP spoofing and how is it prevented?

**Short Interview Answer:**
ARP maps an IP address to a MAC address on the local network using a broadcast request and reply. The result is cached in the ARP table, and it never crosses a router.

---

## 37. What is ICMP?

**Answer:**
ICMP is the Internet Control Message Protocol, used for diagnostics and error reporting in IP networks. It is what `ping` and `traceroute` are built on, and the most commonly known message is the Echo Request and Echo Reply. It does not carry application data and is not part of TCP or UDP.

**Example:**
Running `ping google.com` sends an ICMP Echo Request and gets back an Echo Reply with the round-trip time, which is how we check if a host is reachable.

| ICMP Message | Purpose |
|--------------|---------|
| Echo Request / Reply | `ping` reachability check |
| Destination Unreachable | Host or port not reachable |
| Time Exceeded (TTL) | Used by `traceroute` |
| Redirect | Router suggests a better path |

**Interview Follow-up:**
- Why do firewalls often block ICMP?
- What is the difference between ICMP and ICMPv6?
- Can you ping an IP but not a hostname, and why?

**Short Interview Answer:**
ICMP is used for error reporting and diagnostics in IP networks, and it's the protocol behind ping and traceroute. It doesn't carry user data — it only carries control messages.

---

## 38. What is the difference between ARP and RARP?

**Answer:**
ARP maps a known IP address to an unknown MAC address, while RARP does the opposite — it maps a known MAC address to an IP address. ARP is still used in every network today, but RARP has been replaced by BOOTP and then DHCP. RARP was mainly used when a machine had no IP stored and needed to find one at boot time.

**Example:**
When a device sends an ARP request it is asking "who has 192.168.1.1?" With RARP it would instead ask "192.168.1.1, what is your IP?"

| Feature | ARP | RARP |
|---------|-----|------|
| Direction | IP → MAC | MAC → IP |
| Asked by | Any host | Host with no IP |
| Modern status | Still used everywhere | Replaced by DHCP |
| Layer | Layer 2 | Layer 2 / 3 |

**Interview Follow-up:**
- Why was RARP replaced by DHCP?
- What is BOOTP and how does it differ from RARP?
- What is gratuitous ARP and what is it used for?

**Short Interview Answer:**
ARP finds the MAC address for a known IP, while RARP found the IP for a known MAC. RARP is obsolete today because DHCP does that job far better.

---

## 39. What are error detection techniques like CRC, checksum, and parity bit?

**Answer:**
These are methods to detect errors that occur while data travels over a noisy network. Parity bit is the weakest — it can only detect an odd number of bit errors. Checksum is better and is used in IPv4, UDP, and TCP headers. CRC is the strongest and most commonly used, because Ethernet frames, TCP segments, and IP packets all use it.

**Example:**
If a bit flips while your data crosses the network, the receiver recalculates the CRC and compares it with the received CRC. If they don't match, the frame is dropped and TCP retransmits it.

| Technique | Strength | Where Used |
|-----------|----------|------------|
| Parity bit | Very weak, odd errors only | Simple memory systems |
| Checksum | Moderate | IPv4 header, UDP, TCP |
| CRC | Strong, polynomial based | Ethernet frames, disks, PNG files |

**Common Misconception:**
These techniques only *detect* errors — they don't correct them. Recovery is done by retransmission (in TCP) or by forward error correction.

**Interview Follow-up:**
- What is the difference between error detection and error correction?
- How does CRC detect a burst error better than a parity bit?
- What is forward error correction (FEC)?

**Short Interview Answer:**
Error detection methods like parity, checksum, and CRC catch corrupted data in transit. Parity is the weakest, checksum is moderate, and CRC is the strongest and most widely used in Ethernet and TCP.

---

## 40. What is the difference between unicast, multicast, and broadcast?

**Answer:**
Unicast sends data from one sender to one receiver. Multicast sends to a specific group of interested receivers, where each receiver joins a group and traffic is delivered only to them. Broadcast sends to every device on the network, which is efficient for discovery but creates a lot of unnecessary traffic.

**Example:**
Your YouTube video request is unicast. A live cricket stream to millions of users is multicast through a CDN. And a DHCP DISCOVER message is broadcast so every device on the LAN receives it.

| Type | Recipients | Example |
|------|-----------|---------|
| Unicast | One specific host | HTTP request to a server |
| Multicast | A subscribed group | IPTV, stock tickers, video streaming |
| Broadcast | All hosts on the segment | ARP request, DHCP discover |

**Interview Follow-up:**
- Why is broadcast expensive in large networks?
- What is broadcast storm and how do you prevent it?
- How does multicast routing work?

**Short Interview Answer:**
Unicast is one-to-one, multicast is one-to-a-group of subscribers, and broadcast is one-to-everyone on the network. Broadcast is fine in a small LAN but causes problems on large networks, which is why multicast is preferred for streaming.

## 41. What is the difference between static routing and dynamic routing?

**Answer:**
Static routes are manually configured by an administrator with a fixed next hop, and they don't change unless someone edits them. Dynamic routing protocols like OSPF and BGP automatically learn routes from other routers and update them when the network changes. Static is simple and predictable for small networks, dynamic scales and adapts for larger ones.

**Example:**
A small office with one router uses a static default route (`0.0.0.0/0 → 192.168.1.1`). An enterprise with 50 branch offices uses OSPF so routes update automatically when a link fails.

| Feature | Static Routing | Dynamic Routing |
|---------|----------------|-----------------|
| Configuration | Manual | Automatic |
| CPU/memory usage | Very low | Higher |
| Adaptability | None | Automatic failover |
| Security | No routing updates received | Can authenticate peers |
| Best for | Small, simple, hub networks | Large, changing networks |

**Common Misconception:**
Static routing is not "better" — it's a trade-off. It's lower overhead but if the link dies, someone must fix it manually.

**Interview Follow-up:**
- What is a default route?
- How do you add a static route on a Linux machine and Windows?
- What happens if a dynamic route points to a black hole?

**Short Interview Answer:**
Static routes are configured manually and never change on their own, while dynamic routing protocols learn and update routes automatically. Static suits small simple networks; dynamic suits large ones where links fail and change often.

---

## 42. What is the difference between distance vector and link state routing?

**Answer:**
Distance vector protocols like RIP share the whole routing table with neighbours, and each router picks the path with the lowest hop count. Link state protocols like OSPF share only their own link information, and every router independently builds a complete map of the network to find the shortest path. Link state converges faster and scales better, but it needs more CPU and memory.

**Example:**
RIP is like each person telling their neighbours the full list of routes they know, while OSPF is like everyone having the full map and calculating the best path themselves.

| Feature | Distance Vector | Link State |
|---------|------------------|------------|
| Examples | RIP, EIGRP (hybrid) | OSPF, IS-IS |
| Knowledge | Full routing table of neighbours | Only own links |
| Algorithm | Bellman-Ford, hop count | Dijkstra's shortest path |
| Convergence | Slow | Fast |
| CPU & memory | Low | High |
| Network view | Partial, neighbour-based | Full topology map |

**Interview Follow-up:**
- Why does distance vector have the count-to-infinity problem?
- What is the split horizon and poison reverse technique?
- Which is better for a large enterprise and why?

**Short Interview Answer:**
Distance vector routers share their whole routing table and pick the lowest hop count, while link state routers share only their own links and build a full map to compute the shortest path. Link state converges faster and scales better, so OSPF replaced RIP in most networks.

---

## 43. What are RIP, OSPF, and BGP?

**Answer:**
RIP is an old distance vector protocol that uses hop count and is limited to 15 hops, so it's fine for small networks but not for large ones. OSPF is a link state protocol that uses shortest path calculation, converges fast, and works inside a single organization. BGP is the protocol that connects different organizations — it's the actual routing system of the internet, and it exchanges millions of routes between ISPs and networks.

**Example:**
Inside your campus network, routers run OSPF to find internal paths. Between your ISP and the rest of the world, BGP decides which paths to the internet your packets will take.

| Feature | RIP | OSPF | BGP |
|---------|-----|------|-----|
| Type | Distance vector | Link state | Path vector (EGP) |
| Metric | Hop count (max 15) | Cost based on bandwidth | AS path length + policies |
| Scope | Small networks | Medium to large LAN/WAN | Between autonomous systems |
| Convergence | Slow | Fast | Slow but stable |
| Algorithm | Bellman-Ford | Dijkstra (SPF) | Best path selection with policies |
| Admin distance | 120 | 110 | 20 (for eBGP) |

**Interview Follow-up:**
- What is an autonomous system (AS)?
- What is BGP path-vector and why does it prevent loops?
- What is the BGP best-path selection process?

**Short Interview Answer:**
RIP is a simple hop-count distance vector protocol for tiny networks, OSPF is a link state protocol used inside organizations for fast convergence, and BGP is the path vector protocol that routes traffic between autonomous systems — basically the internet's routing backbone.

---

## 44. What is the difference between IGP and EGP?

**Answer:**
IGP stands for Interior Gateway Protocol and is used for routing *within* one organization or autonomous system. EGP stands for Exterior Gateway Protocol and is used for routing *between* different autonomous systems. OSPF, RIP, and IS-IS are IGP protocols, while BGP is the EGP that actually runs the internet.

**Example:**
Your company's internal routers use OSPF, which is an IGP. Your ISP exchanging routes with Google's network uses BGP, which is an EGP.

| Feature | IGP | EGP |
|---------|-----|-----|
| Scope | Inside one AS | Between different ASes |
| Examples | OSPF, RIP, IS-IS, EIGRP | BGP (the only real EGP) |
| Admin control | Single organization | Multiple independent organizations |
| Metric | Technical, like cost/hop count | Policy and AS path length |
| Trust model | Internal trust | No inherent trust |

**Common Misconception:**
BGP is not "less secure" by design — it is deliberately open, and security is added with authentication, prefix filtering, and RPKI.

**Interview Follow-up:**
- What is an AS number and how is it assigned?
- What is iBGP vs eBGP?
- Why is BGP a path vector rather than a link state protocol?

**Short Interview Answer:**
IGP is for routing inside one organization's network, like OSPF or RIP, while EGP is for routing between different organizations. BGP is the EGP that connects all the autonomous systems on the internet.

---

## 45. What is a routing table?

**Answer:**
A routing table is a list stored in a router or host that tells it which next hop to use for a given destination network. When a packet arrives, the router looks up the destination IP in the table and forwards it via the best matching entry. The table is built from directly connected networks, static routes, and routes learned via dynamic protocols.

**Example:**
On your laptop, the routing table shows `0.0.0.0/0 → 192.168.1.1` which means "for anything not in my local network, send it to the router."

```
Destination        Gateway         Interface
0.0.0.0/0          192.168.1.1     192.168.1.10
192.168.1.0/24     on-link         192.168.1.10
127.0.0.1/8        on-link         lo
```

**Interview Follow-up:**
- What is the longest prefix match rule?
- How do you view the routing table on Linux, Windows, and a router?
- What is a routing loop and how is it detected?

**Short Interview Answer:**
A routing table is the set of best paths a router or host knows, and the longest matching prefix decides where a packet goes. It is filled by connected networks, static routes, and dynamic routing protocols.

---

## 46. What is the difference between Layer 2 and Layer 3 switches?

**Answer:**
A Layer 2 switch forwards Ethernet frames using MAC addresses and works only inside one broadcast domain. A Layer 3 switch also routes between VLANs using IP addresses, so it can do the job of both a switch and a router at high speed in hardware. Layer 3 switches are common in campus and data center networks where you have many VLANs.

**Example:**
A Layer 2 switch connects all devices in the HR VLAN. A Layer 3 switch routes traffic from the HR VLAN to the Accounts VLAN internally, without sending it to a separate router.

| Feature | Layer 2 Switch | Layer 3 Switch |
|---------|-----------------|-----------------|
| Operates at | Data Link layer | Network + Data Link |
| Uses | MAC addresses | IP addresses |
| Scope | Single broadcast domain | Multiple VLANs / subnets |
| Can route? | No | Yes |
| Speed | Very high | High (routing in hardware) |

**Interview Follow-up:**
- What is inter-VLAN routing and why does it need a Layer 3 device?
- What is a router-on-a-stick?
- How does a Layer 3 switch differ from a router for routing protocols?

**Short Interview Answer:**
A Layer 2 switch forwards frames using MAC addresses within one broadcast domain, while a Layer 3 switch also routes between VLANs using IP addresses. Layer 3 switches are faster for internal routing and widely used in campus networks.

---

## 47. What is VLAN and why is it used?

**Answer:**
A VLAN (Virtual LAN) logically segments one physical switch into multiple separate broadcast domains, each with its own VLAN ID. Devices in different VLANs cannot talk directly — they need a router or Layer 3 switch to communicate. VLANs are used to improve security, separate departments, and reduce broadcast traffic.

**Example:**
In an office, HR and Accounts plug into the same physical switch, but HR is in VLAN 10 and Accounts in VLAN 20. A user in Accounts cannot directly reach an HR machine unless routing is allowed by a firewall.

**Example in Cisco IOS:**
```
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 10
```

**Interview Follow-up:**
- What is a trunk port and why is it needed?
- What is 802.1Q and what does it do?
- What is a VLAN hopping attack?

**Short Interview Answer:**
A VLAN splits one physical switch into multiple logical LANs, each being a separate broadcast domain. It's used for security, department separation, and reducing broadcast traffic, and moving traffic between VLANs requires routing.

---

## 48. What is Spanning Tree Protocol (STP)?

**Answer:**
STP prevents loops in a switched network by calculating one active path between switches and blocking the redundant links. It works by exchanging Bridge Protocol Data Units (BPDUs) and electing a root bridge, then choosing the best path from there. Without STP, a broadcast storm would occur because a switch would forward the same broadcast frame endlessly around the loop.

**Example:**
If you connect two switches with two cables for redundancy, STP keeps one cable active and blocks the other. If the active cable fails, STP unblocks the backup in a few seconds.

| Variant | Speed | Use Case |
|---------|-------|----------|
| STP (802.1D) | 30–50 seconds | Legacy, rarely used now |
| RSTP (802.1w) | ~6 seconds | Fast convergence, common |
| MSTP (802.1s) | Fast, multiple instances | Runs several RSTPs together |

**Interview Follow-up:**
- What is a root bridge election and Bridge ID?
- What is BPDU and what does it contain?
- What is PortFast and BPDU Guard?

**Short Interview Answer:**
STP blocks redundant links in a switched network so broadcasts don't loop forever. It elects a root bridge and calculates the best path, with RSTP being the faster modern version used today.

---

## 49. What is the difference between collision domain and broadcast domain?

**Answer:**
A collision domain is the area where devices compete for the same shared medium and can cause collisions. A broadcast domain is the area where a broadcast sent by one device reaches every device. Switches break collision domains per port, while routers and VLANs break broadcast domains.

**Example:**
Every port on a switch is a separate collision domain, so a collision on one port doesn't affect others. But by default all ports in the same VLAN share one broadcast domain, which is why a broadcast storm can affect the whole VLAN.

| Feature | Collision Domain | Broadcast Domain |
|---------|-------------------|-------------------|
| Definition | Where collisions can happen | Where broadcasts reach everyone |
| Broken by | Switch (per port) | Router or VLAN boundary |
| Hub | One shared collision domain | One shared broadcast domain |
| Switch | One per port | Still one per VLAN |
| Router | One per interface | One per interface |

**Common Misconception:**
A switch reduces collision domains but does **not** break broadcast domains — that needs a router or a VLAN split. This is a very common interview question.

**Interview Follow-up:**
- What is a broadcast storm and how do you stop it?
- How does a router break a broadcast domain?
- Why is a hub bad for a modern network?

**Short Interview Answer:**
A collision domain is where devices compete and can collide, and a broadcast domain is where broadcasts reach all devices. Switches create separate collision domains per port, but routers and VLANs are needed to separate broadcast domains.

---

## 50. What is ARP spoofing?

**Answer:**
ARP spoofing (also called ARP poisoning) is an attack where an attacker sends fake ARP replies to a network, claiming that their MAC address belongs to the IP address of the gateway or another host. This lets the attacker intercept and modify traffic between victims, enabling man-in-the-middle attacks. It's a Layer 2 attack, so it only works inside the same local network.

**Example:**
An attacker on the office Wi-Fi sends ARP replies saying "192.168.1.1 is at attacker-MAC", so all users' traffic passes through the attacker's machine where passwords can be captured.

**Prevention:**
- Dynamic ARP Inspection (DAI) on switches
- Static ARP entries for critical hosts like gateways
- Port security and 802.1X authentication
- Encryption like HTTPS and VPN

**Interview Follow-up:**
- What is a man-in-the-middle attack?
- How does DHCP snooping prevent ARP spoofing?
- What is DNS spoofing and how is it different?

**Short Interview Answer:**
ARP spoofing is when an attacker sends fake ARP responses to redirect traffic through their machine, allowing man-in-the-middle attacks. It's prevented with dynamic ARP inspection, static ARP entries, and encryption.

---

## 51. What is a firewall?

**Answer:**
A firewall is a security device or software that monitors and controls network traffic based on a set of rules. It can allow or block traffic based on IP addresses, ports, protocols, or application content. It is the basic control that separates a trusted internal network from the untrusted internet.

**Example:**
A company firewall allows port 443 outbound for HTTPS but blocks port 23 (Telnet), and denies all direct inbound connections to internal servers.

**Types:**
| Type | Level | Example |
|------|-------|---------|
| Packet filter | Layer 3–4 | iptables rules |
| Stateful inspection | Layer 4 with connection tracking | Modern enterprise firewalls |
| Application firewall (WAF) | Layer 7 | Blocking SQL injection in HTTP |
| Next-gen firewall (NGFW) | Layer 7 + app awareness | Inspects app traffic |

**Common Misconception:**
A firewall does not protect against malware downloaded from a legitimate website, or against attacks inside your own network. It also doesn't replace patching and secure coding.

**Interview Follow-up:**
- What is the difference between a stateless and stateful firewall?
- What is a DMZ and how is it placed with a firewall?
- What is a reverse proxy and how is it different from a firewall?

**Short Interview Answer:**
A firewall filters incoming and outgoing traffic based on IP, port, protocol, or application rules, forming the boundary between a trusted and untrusted network. It protects the perimeter but doesn't replace secure coding or patching.

---

## 52. What is the difference between IDS and IPS?

**Answer:**
An IDS (Intrusion Detection System) only monitors traffic and raises an alert when it sees something suspicious, but it doesn't block it. An IPS (Intrusion Prevention System) sits inline in the network and can actively block malicious traffic in real time. IDS is like a security camera, IPS is like a security guard who can stop the person.

**Example:**
An IDS may alert you that someone is scanning ports. An IPS will detect the same scan and immediately drop those packets.

| Feature | IDS | IPS |
|---------|-----|-----|
| Action | Alert only | Alert + block |
| Placement | Out-of-band / mirror port | Inline in the path |
| Response time | Delayed | Real time |
| Latency impact | None | Slight, due to inspection |
| False positives | Just noise | Can block legitimate traffic |

**Interview Follow-up:**
- What is a false positive and false negative in security?
- How is a WAF different from an IPS?
- What is signature-based vs anomaly-based detection?

**Short Interview Answer:**
An IDS detects and alerts about suspicious traffic but doesn't block it, while an IPS sits inline and blocks it in real time. IDS is monitoring, IPS is prevention — that's the main difference.

---

## 53. What is NAT and its types?

**Answer:**
NAT (Network Address Translation) translates private IP addresses into a public IP address so multiple devices can share one public IP with the internet. It saves IPv4 addresses and also adds a layer of security by hiding internal addresses. The most common type today is PAT/NAT-overload, where many private IPs map to a single public IP using different port numbers.

**Example:**
In your home, your phone gets `192.168.1.42` and laptop gets `192.168.1.43`, but the internet sees only your router's public IP. The router keeps a table of which port belongs to which device.

| Type | Meaning | Example |
|------|---------|---------|
| Static NAT | Fixed private ↔ fixed public mapping | Web server with a fixed public IP |
| Dynamic NAT | Pool of public IPs assigned on demand | Many users sharing a pool |
| PAT (NAT overload) | Many private IPs → one public IP using ports | Home routers (most common) |

**Common Misconception:**
NAT breaks end-to-end connectivity — that's exactly why peer-to-peer apps need STUN/TURN servers, and why you can't easily host a server behind a normal home NAT.

**Interview Follow-up:**
- What is NAT traversal and how does STUN help?
- What is hairpin NAT and why does it matter?
- How does NAT interact with firewalls?

**Short Interview Answer:**
NAT translates private IPs into a public IP so many devices can share one internet connection, and the most common type is PAT where port numbers distinguish devices. It saves IPv4 addresses but breaks direct peer-to-peer connections.

---

## 54. What is a VPN and how does it work?

**Answer:**
A VPN creates a secure, encrypted tunnel over a public network so that remote users or branch offices can access internal resources as if they were on the local network. The user's traffic is encrypted, sent through the tunnel to a VPN server, decrypted there, and then forwarded to the internal network. It uses IPsec or SSL/TLS depending on the type.

**Example:**
An employee working from home connects to the company VPN, then accesses an internal Git repository that is not exposed to the internet.

| Type | Protocol | Use Case |
|------|----------|----------|
| Site-to-site VPN | IPsec | Connecting two branch offices |
| Remote access (SSL) | TLS | Employees connecting from home |
| MPLS VPN | MPLS | Managed enterprise WAN |
| GRE tunnel | GRE | Simple site-to-site over internet |

**Interview Follow-up:**
- What is a split tunnel vs full tunnel VPN?
- What are the two modes of IPsec — transport and tunnel?
- Why is a VPN not a replacement for application-level security?

**Short Interview Answer:**
A VPN creates an encrypted tunnel across a public network so remote users or branches can securely access internal resources as if they were on the LAN. It uses IPsec or TLS to encrypt traffic end-to-end through a VPN gateway.

---

## 55. What is a DMZ in networking?

**Answer:**
A DMZ (Demilitarized Zone) is a separate network segment between the internal network and the internet, used to host public-facing servers like web or mail servers. It's deliberately exposed but isolated from the internal network, so if a DMZ server is compromised, the attacker still can't reach internal systems directly. The firewall controls traffic between internet ↔ DMZ and DMZ ↔ internal network.

**Example:**
A company's public website runs in the DMZ. If hackers break into the web server, the firewall still blocks them from reaching the internal database server.

**Typical rule set:**
- Internet → DMZ: allowed on 80/443 only
- Internet → Internal: blocked
- DMZ → Internal: only specific ports allowed
- Internal → DMZ: allowed for updates

**Interview Follow-up:**
- What is a three-legged firewall?
- Why shouldn't you place a database in the DMZ?
- How do you decide which ports to expose in the DMZ?

**Short Interview Answer:**
A DMZ is a semi-public network zone for internet-facing servers, placed between the internet and the internal network. If a DMZ server is hacked, the firewall still blocks access to internal systems.

---

## 56. What is port forwarding?

**Answer:**
Port forwarding tells a router to send incoming traffic on a specific external port to a specific internal device and port. It's used to expose an internal service, like a home web server or a gaming console, to the internet. The device must have a static or reserved IP so the forward stays valid.

**Example:**
To host a website at home, you forward external port 8080 to `192.168.1.50:80`, so anyone hitting your public IP on 8080 reaches your internal web server.

**Related terms:**
- **Port forwarding** — router-level mapping, inbound traffic
- **NAT / PAT** — translation of private to public IPs
- **Port redirection** — mapping to a different internal port

**Interview Follow-up:**
- What is the difference between port forwarding and a DMZ host setting?
- Why is port forwarding a security risk?
- What is UPnP and how does it automate port forwarding?

**Short Interview Answer:**
Port forwarding is a router setting that sends incoming traffic on an external port to a device inside your private network. It's used to expose internal services like a home server, but it also opens up a security risk.

---

## 57. What is a DDoS attack and how can it be prevented?

**Answer:**
A DDoS (Distributed Denial of Service) attack floods a server with massive traffic from many compromised devices, so legitimate users can't access the service. The traffic can be volumetric (huge volume) or protocol-based (like SYN floods). Prevention is done at the network edge using rate limiting, traffic filtering, and CDN or scrubbing services, since you cannot stop the traffic at the origin.

**Example:**
A botnet of 100,000 devices sends requests to a small e-commerce site, exhausting its bandwidth and taking it offline. Using a CDN with traffic scrubbing absorbs the flood before it reaches the origin server.

| Attack Type | Method | Example |
|-------------|--------|---------|
| Volumetric | Huge volume of traffic | UDP flood, DNS amplification |
| Protocol | Exhausts server resources | SYN flood, HTTP flood |
| Application | Mimics real user requests | Slowloris, Layer 7 attack |

**Prevention:**
- CDN and Anycast (spreads load geographically)
- Rate limiting and WAF rules
- Traffic scrubbing / DDoS mitigation service
- Anycast DNS and blocking known amplification sources

**Interview Follow-up:**
- What is DNS amplification and why is it so effective?
- Why can't you just block the attacking IPs at your server?
- What is a botnet?

**Short Interview Answer:**
A DDoS attack floods a service with traffic from many compromised devices so real users get denied access. You prevent it at the edge using a CDN, rate limiting, and traffic scrubbing, because blocking at the origin is too late.

---

## 58. What is IPsec and what are its modes?

**Answer:**
IPsec is a suite of protocols for securing IP traffic at the network layer, and it's the standard protocol for VPNs. It provides encryption, authentication, and integrity protection for IP packets. It works in two modes — transport mode protects only the payload, while tunnel mode encrypts the entire original packet inside a new IP header.

**Example:**
In transport mode, the original IP header stays visible and only the payload is encrypted. In tunnel mode, used by site-to-site VPNs, the whole original packet is encrypted and wrapped in a new IP header.

| Mode | What's Encrypted | IP Header | Use Case |
|------|------------------|-----------|----------|
| Transport | Payload only | Original kept | Encrypting app data, IPsec with GRE |
| Tunnel | Entire packet | New header added | Site-to-site VPNs, encrypting IP packets |

**Common Misconception:**
IPsec is not just a VPN protocol — it's a security framework that can be used for encryption, authentication, or both.

**Interview Follow-up:**
- What is IKE and what role does it play?
- What is AH vs ESP in IPsec?
- Why does tunnel mode add overhead?

**Short Interview Answer:**
IPsec secures IP traffic at Layer 3 using encryption and authentication. In transport mode only the payload is protected, while tunnel mode encrypts the whole packet and adds a new IP header, which is why it's used for site-to-site VPNs.

---

## 59. What is the difference between SSL/TLS and HTTPS?

**Answer:**
SSL/TLS is the encryption and authentication protocol, and HTTPS is simply HTTP running over that protocol. SSL/TLS is the transport-level security layer; HTTPS is the application-level result of using it. So every HTTPS connection uses TLS, but TLS can also be used for other protocols like SSH, SMTP, and FTP.

**Example:**
`https://api.example.com` is HTTP data wrapped in TLS encryption on port 443. `ssh` and `smtps` also use TLS but are not HTTPS.

| Feature | SSL/TLS | HTTPS |
|---------|----------|--------|
| Type | Security protocol | Application protocol over TLS |
| Purpose | Encrypt, authenticate, verify integrity | Serve web pages securely |
| Port | Not fixed | 443 |
| Can be used for | SSH, SMTP, IMAP, FTP too | Only HTTP traffic |
| Layer (TCP/IP) | Application layer security | Application layer protocol |

**Common Misconception:**
HTTPS and SSL/TLS are not the same thing — SSL/TLS is the mechanism, HTTPS is one specific use of it.

**Interview Follow-up:**
- Why did browsers remove support for SSL 3.0?
- What is HSTS and how does it improve security?
- What is a certificate chain and root CA?

**Short Interview Answer:**
SSL/TLS is the security protocol that provides encryption, while HTTPS is HTTP running over TLS. TLS can secure other protocols too, so HTTPS is just one application of it.

---

## 60. What is MAC filtering?

**Answer:**
MAC filtering is an access control technique where a switch or router allows or blocks traffic based on the device's MAC address. It's a Layer 2 control used to restrict which physical devices can connect to the network. However, MAC addresses are easy to spoof, so it's a weak security measure and should only be used as an extra layer, not as primary security.

**Example:**
A corporate switch has a whitelist of MAC addresses for company laptops, and any other MAC trying to connect on that port is blocked by port security.

**Limitations:**
- MAC addresses are easy to spoof
- Doesn't work across routed networks
- Doesn't encrypt traffic
- Not suitable for dynamic device environments like BYOD

**Interview Follow-up:**
- What is port security and how is it related to MAC filtering?
- How easy is MAC spoofing and how do you detect it?
- Why is IP-based filtering stronger than MAC filtering?

**Short Interview Answer:**
MAC filtering controls access based on a device's hardware MAC address at Layer 2. It's weak because MAC addresses can be spoofed easily, so it's used only as a supplementary security measure, not as the main defense.

---

## 61. What is MPLS?

**Answer:**
MPLS (Multiprotocol Label Switching) is a technology where packets are forwarded based on labels instead of doing a full IP routing lookup at every hop. Routers in an MPLS network only read a short label and swap it, which makes forwarding faster and allows easier traffic engineering. It is widely used by ISPs to build managed WANs and VPNs for enterprises.

**Example:**
An enterprise connects 20 branch offices using an MPLS VPN from an ISP. The company's internal addresses stay private across the ISP network, and the ISP guarantees QoS and a private WAN without the company owning any routers.

| Feature | IP Routing | MPLS |
|---------|-----------|-------|
| Forwarding basis | Longest prefix match on IP | Short label |
| Lookup speed | Slower, table-driven | Fast, label swap |
| Traffic engineering | Difficult | Easy with explicit paths |
| QoS / bandwidth guarantee | Difficult | Supported with TE |
| Used for | Internet | Enterprise WAN, service provider networks |

**Common Misconception:**
MPLS is not a replacement for IP routing — IP routing is still used at the edge, and MPLS is used in the core for fast, controlled forwarding.

**Interview Follow-up:**
- What is an MPLS label and what fields does it contain?
- What is L2TP and how does it compare to MPLS?
- What is traffic engineering in MPLS?

**Short Interview Answer:**
MPLS forwards packets using short labels instead of full IP lookups, which makes it faster and easier to manage for QoS and private WANs. It's commonly used by service providers to build enterprise MPLS VPNs.

---

## 62. What is QoS in networking?

**Answer:**
QoS (Quality of Service) is a set of techniques to give priority to certain types of network traffic over others. It uses classification, marking, queuing, and shaping to make sure that voice and video calls stay smooth even when the link is busy. QoS is a network-level guarantee you configure on routers and switches, and it's important for real-time applications like VoIP and video conferencing.

**Example:**
During a video call on a congested office link, QoS marks the call's packets as high priority so they get queued first, while a file download is marked low priority and waits.

| QoS Technique | What It Does |
|---------------|--------------|
| Classification | Identifies traffic type (voice, video, data) |
| Marking | Tags packets with priority (DSCP / 802.1p) |
| Queuing | Places packets in separate queues |
| Shaping | Smooths traffic to avoid bursts |
| Policing | Drops or re-marks excess traffic |

**Common Misconception:**
QoS does not create extra bandwidth — it only prioritises and manages the bandwidth you already have. If the link is fully saturated, QoS helps the important traffic survive.

**Interview Follow-up:**
- What is DSCP and how is it different from 802.1p?
- Why is QoS important for VoIP and gaming?
- What is traffic policing vs shaping?

**Short Interview Answer:**
QoS prioritises important traffic like voice and video over less critical traffic when bandwidth is limited, using classification, marking, and queuing. It doesn't add bandwidth — it decides who gets it first.

---

## 63. What is jitter in networking?

**Answer:**
Jitter is the variation in the delay between packets — when packets arrive with inconsistent gaps, audio or video breaks up into stuttery playback. Even if the average latency is low, high jitter makes real-time communication feel unstable. This is why VoIP and video conferencing use a jitter buffer to smooth out the variations.

**Example:**
In a video call, if one packet takes 30 ms and the next takes 200 ms, the video will freeze then jump, even though the average latency looks fine.

**Fixes:**
- Jitter buffer (adds a small delay to smooth output)
- Increasing bandwidth
- QoS prioritisation of real-time traffic
- Selecting a closer network path (CDN / edge server)

**Common Misconception:**
Bandwidth doesn't fix jitter — you can have plenty of bandwidth and still experience jitter if routing or queuing is unstable. Jitter is about delay *variation*, not delay *amount*.

**Interview Follow-up:**
- What is a jitter buffer and what trade-off does it introduce?
- How is jitter different from latency?
- How does jitter affect VoIP quality?

**Short Interview Answer:**
Jitter is the variation in packet delay, and it makes real-time audio and video stutter. You reduce its effect with a jitter buffer, QoS, and better routing, not just more bandwidth.

---

## 64. What is a CDN?

**Answer:**
A CDN (Content Delivery Network) is a network of servers placed across different geographic locations that caches content closer to the users. When a user requests content, the CDN serves it from the nearest server instead of the origin server, which reduces latency and origin load. CDNs also provide DDoS protection, SSL termination, and caching of static and dynamic content.

**Example:**
A user in Mumbai loading a YouTube video is served it from a nearby CDN server rather than a US data center, so the video starts much faster and YouTube's origin servers don't get overwhelmed.

**Example in Java backend context:**
```java
// Instead of every user hitting origin,
// static assets are cached at CDN edge nodes
public static final String CDN_BASE = "https://cdn.example.com/assets/";
```

**Interview Follow-up:**
- What is an edge node?
- How does a CDN decide which cache to serve from?
- What is a cache invalidation strategy (TTL, purge)?

**Short Interview Answer:**
A CDN caches content on servers located close to users, so pages and videos load faster and the origin server gets less load. It also helps with latency and DDoS protection, which is why almost every large site uses one.

---

## 65. What is load balancing?

**Answer:**
Load balancing distributes incoming requests across multiple servers so no single server gets overloaded, which improves both performance and availability. It works at two levels — Layer 4 balances TCP connections and Layer 7 balances based on the actual HTTP request, like URL or headers. It also handles health checks, removing unhealthy servers from rotation.

**Example:**
A Spring Boot app running as 4 instances behind Nginx or an AWS ALB gets requests distributed round-robin; if one instance goes down, the health check stops sending it traffic.

| Level | Works On | Example |
|-------|----------|---------|
| Layer 4 | TCP/UDP connections | IPVS, HAProxy TCP mode |
| Layer 7 | HTTP requests, URLs, headers | Nginx, Envoy, AWS ALB |
| Client-side | Requests from the browser | DNS round robin, client load balancers |

**Interview Follow-up:**
- What algorithms are used — round robin, least connections, IP hash?
- What is a sticky session and what problem does it solve?
- What is health check and why is it important?

**Short Interview Answer:**
Load balancing spreads requests across multiple servers to improve performance and availability, working at Layer 4 on connections or Layer 7 on HTTP requests. It also removes unhealthy servers automatically through health checks.

---

## 66. What is DNS caching?

**Answer:**
DNS caching stores the results of a DNS lookup for a period of time so the same query doesn't have to go out to the network again. The TTL (Time to Live) value in the DNS record decides how long the entry stays cached. Caching makes lookups almost instant, but it also means changes to DNS can take time to propagate.

**Example:**
When you visit `google.com`, the browser caches the IP for 300 seconds, so revisiting within that time needs no new DNS query.

| Cache Location | Managed By | Typical TTL Impact |
|----------------|------------|--------------------|
| Browser cache | The browser | Seconds to minutes |
| OS cache | The OS resolver | Minutes |
| Recursive resolver cache | ISP / public DNS (8.8.8.8) | Seconds to hours |
| Authoritative cache | DNS server itself | As per record TTL |

**Common Misconception:**
When you "change DNS and it doesn't work", it's usually because of caching at the browser, OS, or resolver — flushing the DNS cache (`ipconfig /flushdns`) usually fixes it.

**Interview Follow-up:**
- What is a TTL value and what happens if you set it too high or low?
- How does DNS caching improve performance?
- What is DNS propagation and why does it take time?

**Short Interview Answer:**
DNS caching stores lookup results for the TTL period so repeated queries are instant and the network isn't hit every time. The trade-off is that DNS changes take time to propagate everywhere.

---

## 67. What is Zero Window and Silly Window Syndrome?

**Answer:**
A Zero Window occurs when the receiver's buffer is full, so it advertises a window size of zero, telling the sender to stop sending until space frees up. Silly Window Syndrome happens when the receiver only has a tiny amount of space but still accepts a very small window, which results in tiny inefficient segments. It's avoided by having the receiver wait until it has enough space for a reasonably sized segment before accepting more.

**Example:**
A slow mobile client can't process data fast enough; its buffer fills, it sends a zero-window update, and the server pauses transmission until the client drains the buffer.

| Condition | Meaning | Problem |
|-----------|---------|---------|
| Zero window | Receiver buffer full, sender must stop | Can stall the connection |
| Silly window | Very small window advertised | Tiny, inefficient segments |
| Solution | Delayed ACK, window scaling, SWS avoidance | TCP algorithms handle this |

**Common Misconception:**
Zero window is a "hang" in TCP, but it is not a bug — it's the flow control mechanism doing its job. The connection recovers when the receiver drains its buffer.

**Interview Follow-up:**
- What is a zero-window probe and why is it sent?
- How does window scaling help with high bandwidth links?
- What causes a TCP connection to stall for a long time?

**Short Interview Answer:**
Zero window means the receiver's buffer is full so it asks the sender to pause, and Silly Window Syndrome is when the receiver advertises such a small window that segments become tiny and inefficient. Both are handled by TCP's own flow control algorithms.

---

## 68. What is header compression?

**Answer:**
Header compression reduces the size of protocol headers by removing fields that are predictable or can be inferred, so more of the bandwidth carries actual data. This matters most on mobile networks and lossy links where every byte is expensive. The most well known example is IP header compression in IPv6 and TCP header compression in mobile networks.

**Example:**
In a mobile 3G/4G network, TCP's 20-byte header can be bigger than a small acknowledgement packet, so compressing it saves significant bandwidth. In video calls, compressible headers reduce latency and jitter.

| Technique | What It Compresses | Where Used |
|-----------|-------------------|------------|
| TCP header compression | TCP header | Mobile / low bandwidth links |
| IP header compression (IPHC) | IPv6 header | Mobile networks |
| HPACK (HTTP/2) | HTTP headers | Web servers and browsers |
| Brotli / gzip | Response body | Web servers, CDNs |

**Common Misconception:**
Header compression doesn't compress the actual data payload — it compresses metadata. Body compression (gzip/Brotli) is a different thing.

**Interview Follow-up:**
- Why is header compression important in HTTP/2 and HTTP/3?
- What is HPACK and why was it introduced?
- When would header compression not be a good idea?

**Short Interview Answer:**
Header compression shrinks protocol headers that contain predictable or repeated fields, leaving more bandwidth for real data. It's especially useful on mobile links, and HTTP/2's HPACK is a common example.

---

## 69. What is the difference between soft state and hard state protocols?

**Answer:**
A hard state protocol keeps explicit information about every connection or session, so state is remembered until explicitly torn down. A soft state protocol keeps no permanent record and relies on periodic refresh messages or timeouts to keep state alive. Hard state is more reliable but uses more memory; soft state is simpler and self-healing but adds periodic overhead.

**Example:**
TCP is a hard state protocol because the connection state is maintained in the socket until it's closed. Routing protocols like OSPF are soft state because their entries expire if refresh hellos stop, so the network self-heals without manual cleanup.

| Feature | Hard State | Soft State |
|---------|-----------|------------|
| State storage | Explicit, kept in memory | No explicit storage |
| Cleanup | Manual, explicit | Automatic via timeout |
| Example | TCP, database sessions | OSPF, routing entries, DHCP leases |
| Reliability | Higher, but leaks if not closed | Self-healing, but needs refresh traffic |
| Resource usage | Higher | Lower |

**Common Misconception:**
Soft state doesn't mean "unreliable" — it means the state doesn't need explicit teardown, which is exactly why routing protocols self-heal.

**Interview Follow-up:**
- What is the state explosion problem in firewalls?
- Why do soft state protocols need periodic keepalives?
- Which approach is used by modern firewalls and why?

**Short Interview Answer:**
Hard state protocols explicitly maintain and store connection state until closed, like TCP, while soft state protocols rely on refresh messages and timeouts and don't keep permanent state, like routing tables in OSPF. Hard state is more reliable; soft state is simpler and self-healing.

---

## 70. What is network congestion and how can it be controlled?

**Answer:**
Network congestion happens when demand for bandwidth exceeds what a link can carry, causing packets to queue, latency to rise, and packets to be dropped. It is controlled at two levels — TCP does end-to-end congestion control using its congestion window, while routers use queuing, traffic shaping, and QoS to manage the link. The core rule is to detect congestion early and back off, not push more data into an already full pipe.

**Example:**
During a flash sale, a sudden spike in traffic saturates the link between your server and the internet; TCP slows down senders automatically, and QoS rules keep the critical checkout API responsive.

| Control Level | Technique | Scope |
|---------------|-----------|-------|
| Transport | TCP congestion control (slow start, AIMD) | End-to-end |
| Network | Queuing, QoS, traffic shaping | Per link / router |
| Application | Rate limiting, caching, CDN | Application layer |
| Capacity | Adding bandwidth, load balancing | Infrastructure |

**Common Misconception:**
Congestion is not the same as a slow server. Congestion is about the network link being full; a slow server is a resource problem, and the fix is very different.

**Interview Follow-up:**
- What is the difference between congestion and flow control?
- How does a router detect congestion?
- What is bufferbloat and how does it happen?

**Short Interview Answer:**
Network congestion occurs when traffic demand exceeds link capacity, causing queuing delay and packet loss. It's controlled with TCP's congestion window at the sender, and with QoS and queuing at the routers.

---

## 71. What is Wi-Fi?

**Answer:**
Wi-Fi is a wireless networking technology defined by the IEEE 802.11 standards family, and it lets devices connect to a local network using radio waves instead of cables. A Wi-Fi network has an access point that behaves like a wired switch, and devices authenticate and associate with it to get an IP address via DHCP. Wi-Fi works on radio bands, mainly 2.4 GHz and 5 GHz, and now also 6 GHz in Wi-Fi 6E.

**Example:**
Your home Wi-Fi router connects your phone, laptop, and smart TV to the internet over radio waves, and each device gets an IP address when it joins.

| Band | Range | Speed | Penetration |
|------|-------|-------|-------------|
| 2.4 GHz | Longer | Lower | Better, through walls |
| 5 GHz | Shorter | Higher | Weaker, blocked by walls |
| 6 GHz | Shortest | Highest | Poorest, needs line of sight |

**Common Misconception:**
Wi-Fi speed is shared — the total bandwidth is split among all connected devices, so more devices means slower speed per device.

**Interview Follow-up:**
- What is the difference between Wi-Fi 5, Wi-Fi 6, and Wi-Fi 7?
- What channels are used in 2.4 GHz and 5 GHz, and why?
- How does WPA3 improve on WPA2?

**Short Interview Answer:**
Wi-Fi is wireless networking based on the IEEE 802.11 standards, where an access point connects devices over radio bands like 2.4 GHz and 5 GHz. Each device gets an IP via DHCP when it joins, and total bandwidth is shared among all users.

---

## 72. What is the difference between CSMA/CD and CSMA/CA?

**Answer:**
CSMA/CD (Carrier Sense Multiple Access with Collision Detection) is used in wired Ethernet, where a device listens, transmits, and detects collisions, then retransmits. CSMA/CA (Collision Avoidance) is used in Wi-Fi, where a device listens for a fixed time before transmitting, because a wireless device cannot reliably detect collisions while transmitting. Wi-Fi avoids collisions instead of detecting them, using RTS/CTS and random backoff.

**Example:**
On a wired LAN, if two PCs send data at the same time, they detect the collision and retry. On Wi-Fi, a device waits for a random interval and listens for other transmissions first, because radios can't hear and transmit at the same time reliably.

| Feature | CSMA/CD (Ethernet) | CSMA/CA (Wi-Fi) |
|---------|---------------------|-----------------|
| Used in | Wired LAN | Wireless LAN |
| Strategy | Detect and handle collisions | Avoid collisions |
| Can detect collision? | Yes, while transmitting | No, half-duplex radio |
| Extra mechanism | Jam signal, backoff | Random backoff, RTS/CTS, ACK |
| Modern status | Legacy, in full-duplex not needed | Used in all Wi-Fi |

**Interview Follow-up:**
- Why is CSMA/CD not needed in modern full-duplex Ethernet?
- What are the hidden and exposed node problems?
- What is RTS/CTS and how does it help?

**Short Interview Answer:**
CSMA/CD is for wired Ethernet and detects collisions after transmitting, while CSMA/CA is for Wi-Fi and avoids collisions in the first place because radios can't detect them while sending. Wi-Fi uses random backoff and RTS/CTS instead of jam signals.

---

## 73. What is the hidden terminal and exposed terminal problem?

**Answer:**
The hidden terminal problem happens when two devices that can both reach the same access point cannot hear each other, so they transmit at the same time and collide at the AP. The exposed terminal problem is the opposite — a device stays silent unnecessarily because it hears another transmission that wouldn't actually interfere with it. Both are classic wireless problems, and CSMA/CA with RTS/CTS is used to reduce them.

**Example:**
Two devices behind different walls in a large office may both reach AP-A but not each other, so both send at once and their frames collide. With RTS/CTS, the AP's CTS silences them in advance.

| Problem | Scenario | Result | Solution |
|---------|----------|--------|----------|
| Hidden terminal | A and C can't hear each other, both reach B | Collision at B | RTS/CTS, increase carrier sense |
| Exposed terminal | A and C hear each other but can't both reach B | Unnecessary silence | Using directional antennas, adjusting CS thresholds |

**Common Misconception:**
The hidden node problem is a wireless-only issue; wired Ethernet doesn't have it because any device transmitting is visible to everything on the shared segment.

**Interview Follow-up:**
- How does RTS/CTS solve the hidden node problem?
- Why does 802.11e help with QoS issues related to this?
- Is the exposed terminal problem solved by CSMA/CA?

**Short Interview Answer:**
The hidden terminal problem is when two devices can't hear each other but both reach the same AP, so they collide. The exposed terminal is when a device stays silent even though its transmission wouldn't cause a collision. RTS/CTS and careful carrier sensing help reduce both.

---

## 74. What are Bluetooth and ZigBee?

**Answer:**
Bluetooth is a short-range wireless technology mainly used for personal devices like earphones, speakers, and smartwatches, and it offers higher data rates than ZigBee. ZigBee is a low-power, low-data-rate wireless standard designed for sensors, home automation, and IoT devices, with battery life measured in years. In short, Bluetooth is for data-heavy personal use, ZigBee is for many tiny low-power sensors.

**Example:**
Your wireless earphones use Bluetooth. A smart home with motion sensors, door sensors, and smart bulbs on one network uses ZigBee, because those devices need to run for years on a small battery.

| Feature | Bluetooth | ZigBee |
|---------|----------|---------|
| Range | ~10 m (BLE up to 100 m) | ~100 m |
| Data rate | 1–3 Mbps | 250 Kbps |
| Power | Medium | Very low |
| Frequency | 2.4 GHz | 2.4 GHz |
| Topology | Point-to-point, star | Mesh |
| Use case | Audio, wearables, file transfer | IoT, home automation, sensors |
| Battery life | Hours to days | Years |

**Common Misconception:**
Wi-Fi and ZigBee both work at 2.4 GHz, but they are different protocols and are not interoperable — a ZigBee device cannot connect to a Wi-Fi router directly without a bridge.

**Interview Follow-up:**
- What is Bluetooth Low Energy and how is it different from classic Bluetooth?
- What is a ZigBee mesh and why is mesh useful for sensors?
- What is the 2.4 GHz congestion problem and how does it affect IoT?

**Short Interview Answer:**
Bluetooth is a higher-speed short-range technology for personal devices like earphones, while ZigBee is low-power and low-speed, designed for many battery-powered IoT sensors. ZigBee's mesh topology is what makes it ideal for smart home deployments.

---

## 75. What is the difference between LTE and 5G?

**Answer:**
LTE (4G) is a mobile standard that uses OFDMA with an all-IP core, and it offers speeds typically in the tens to hundreds of Mbps. 5G builds on that with much higher bandwidth (using millimeter wave or higher sub-6 frequencies), ultra-low latency, and network slicing for dedicated virtual networks. 5G is designed not just for faster phones but for IoT devices and industrial use cases.

**Example:**
Streaming a 4K video on LTE is fine for a phone, but a remote surgical robot or a massive IoT deployment needs 5G's low latency and huge device capacity.

| Feature | LTE (4G) | 5G |
|---------|----------|-----|
| Speed | Up to ~100 Mbps–1 Gbps | Up to 10+ Gbps |
| Latency | ~50 ms | ~1 ms (URLLC) |
| Frequency | Below 6 GHz | Sub-6 GHz and mmWave |
| Device capacity | Limited | Millions per km² |
| Key features | OFDMA, all-IP core | Network slicing, beamforming, edge computing |
| Use cases | Mobile internet, streaming | IoT, AR/VR, autonomous vehicles, remote surgery |

**Common Misconception:**
5G speed heavily depends on the frequency band and signal strength — a 5G phone on a low band can be slower than a good LTE connection.

**Interview Follow-up:**
- What is network slicing and how is it useful?
- What is mmWave and what are its drawbacks?
- What is edge computing and how does it pair with 5G?

**Short Interview Answer:**
LTE is 4G, offering tens to hundreds of Mbps with a few milliseconds of latency, while 5G offers much higher speed, near-1 ms latency, and support for millions of IoT devices. 5G also enables network slicing, giving dedicated virtual networks for specific applications.

---

## 76. What is MIMO in wireless communication?

**Answer:**
MIMO stands for Multiple-Input Multiple-Output, and it uses multiple antennas on both the transmitter and receiver to send and receive multiple data streams at the same time. This increases throughput without needing more bandwidth, and it also improves signal reliability through beamforming and diversity. MIMO is a core part of Wi-Fi 4 (802.11n) and later, and 4G/5G cellular networks too.

**Example:**
A Wi-Fi 6 router with 4 transmit and 4 receive antennas can send 4 parallel streams to a compatible laptop, giving a much higher throughput than a single-antenna router.

| Term | Meaning |
|------|---------|
| SISO | Single antenna, single stream (baseline) |
| SIMO | 1 transmit, multiple receive (diversity) |
| MISO | Multiple transmit, 1 receive (beamforming) |
| MIMO | Multiple transmit, multiple receive (highest throughput) |

**Common Misconception:**
MIMO needs multiple independent paths between the antennas to actually work — in a small apartment with a single reflection path, MIMO gains are limited.

**Interview Follow-up:**
- What is spatial multiplexing and beamforming?
- What is SU-MIMO vs MU-MIMO?
- How many streams can 4×4 MIMO support?

**Short Interview Answer:**
MIMO uses multiple antennas on both the transmitter and receiver to send parallel data streams, which increases throughput and improves reliability. It's the key technology behind modern Wi-Fi and 4G/5G speeds.

---

## 77. What is Mobile IP?

**Answer:**
Mobile IP is a protocol that lets a mobile device keep its IP address unchanged even when it moves to a different network. The idea is that the device has a permanent home address, and a home agent on the home network forwards packets to its current care-of address. This avoids reconfiguring applications and TCP connections when the device roams.

**Example:**
If your laptop has IP `192.168.1.50` at home and connects to a coffee shop's network, Mobile IP keeps using `192.168.1.50` and the home router forwards traffic to the coffee shop address transparently.

**Modern Reality:**
Mobile IP as originally designed isn't widely used today. Instead, NAT traversal techniques like STUN/TURN, VPN, and IPv6 privacy extensions solve the roaming problem in practice.

**Interview Follow-up:**
- What is a home agent and a care-of address?
- Why isn't Mobile IP widely deployed?
- How do modern apps handle IP changes during roaming?

**Short Interview Answer:**
Mobile IP lets a device keep a fixed IP address even when it moves between networks, by having a home agent forward traffic to its current address. In practice, it's largely replaced by NAT traversal and VPN techniques, but the concept is still relevant.

---

## 78. What is roaming in mobile networks?

**Answer:**
Roaming is when a mobile device connected to one network operator uses another operator's network, usually when travelling outside its home country. The visited network checks with the home network through the HLR and VLR databases to confirm the user is legitimate, then it provides service. Roaming is why your phone works in another country without any manual setup.

**Example:**
When you're in Japan and your Indian SIM starts working, your phone registers with a Japanese network, which checks with your home operator's HLR, gets confirmation, and starts charging you international rates.

**Key Databases:**
| Abbreviation | Full Form | Role |
|--------------|-----------|------|
| HLR | Home Location Register | Home network's central user database |
| VLR | Visitor Location Register | Visited network's local copy |
| AUC | Authentication Centre | Verifies user credentials |
| EIR | Equipment Identity Register | Tracks blocked/blacklisted devices |

**Common Misconception:**
Roaming does not mean free internet. The visited operator still bills your home operator, and your data speeds may be limited even on 5G.

**Interview Follow-up:**
- What is the difference between HLR, VLR, and AUC?
- What is the difference between roaming and a local SIM?
- What is an international roaming bill and why is it expensive?

**Short Interview Answer:**
Roaming is when a device uses another operator's network while travelling, and the visited network validates the user through HLR and VLR databases before allowing service. It's automatic, but the data can be expensive since you're billed through your home operator.

---

## 79. What is handoff in cellular networks?

**Answer:**
Handoff is the process of transferring an ongoing call or data session from one cell tower to another as the user moves, without dropping the connection. The network decides when to hand off based on signal strength and load, and it must be fast and seamless. Modern networks use soft handoff in CDMA and make-before-break in LTE/5G.

**Example:**
You're on a phone call while driving from one area to another. As you move out of range of one tower, the network seamlessly switches your call to the next tower, so you hear no interruption.

| Handoff Type | Behaviour | Used In |
|--------------|-----------|---------|
| Hard handoff | Break old link first, then make new | GSM (2G) |
| Soft handoff | Connect to new before dropping old | CDMA |
| Make-before-break | Add new link, then release old | LTE, 5G |
| Horizontal handoff | Same technology, different cell | Common in all networks |
| Vertical handoff | Different technology (4G ↔ 5G) | Hetero networks |

**Common Misconception:**
Handoff is not the same as a new call being placed. The session (call) is preserved — only the radio link changes, which is why handoff must be very fast.

**Interview Follow-up:**
- What is the difference between hard and soft handoff?
- What causes handoff failure and call drop?
- What is vertical handoff and why is it harder?

**Short Interview Answer:**
Handoff is the seamless transfer of a live call or data session from one cell tower to another as the user moves, without breaking the connection. Modern networks use make-before-break handoff so the new link is established before the old one is released.

---

## 80. What is IEEE 802.11 standard?

**Answer:**
IEEE 802.11 is the family of standards that defines Wi-Fi — how devices associate, authenticate, and transmit data over wireless LANs. Each generation (802.11a/b/g/n/ac/ax) improved speed, security, or efficiency, and the Wi-Fi Alliance gives them friendly names like Wi-Fi 4, 5, 6, and 7. It operates in the 2.4 GHz, 5 GHz, and 6 GHz bands.

**Example:**
Your laptop's Wi-Fi adapter supports 802.11ax, also called Wi-Fi 6, which is why it can connect at higher speeds to a Wi-Fi 6 router.

| Standard | Wi-Fi Name | Year | Band | Max Speed |
|----------|-----------|------|------|-----------|
| 802.11b | — | 1999 | 2.4 GHz | 11 Mbps |
| 802.11n | Wi-Fi 4 | 2009 | 2.4 / 5 GHz | 600 Mbps |
| 802.11ac | Wi-Fi 5 | 2013 | 5 GHz | ~3.5 Gbps |
| 802.11ax | Wi-Fi 6 | 2019 | 2.4 / 5 / 6 GHz | ~9.6 Gbps |
| 802.11be | Wi-Fi 7 | 2024 | 2.4 / 5 / 6 GHz | ~46 Gbps |

**Common Misconception:**
802.11 defines Wi-Fi only — it doesn't define how the device gets an IP address on the network. That's DHCP's job at the next layer up.

**Interview Follow-up:**
- What is the difference between 2.4 GHz and 5 GHz Wi-Fi?
- What is WPA2 vs WPA3?
- How does 5 GHz differ from 6 GHz in terms of range and speed?

**Short Interview Answer:**
IEEE 802.11 is the standard family that defines Wi-Fi, covering how devices connect, authenticate, and transmit data. The generations are named Wi-Fi 4 through 7, with each new one adding higher speeds, better efficiency, and stronger security.

## 81. What is the difference between public IP and private IP?

**Answer:**
A private IP address is reserved for use inside a private network and is not routable on the internet, while a public IP address is unique and routable on the internet. Private addresses are reused by many networks, which saves IPv4 space, and NAT translates them to a public IP at the router. Public IPs are globally unique and assigned by an ISP or registry.

**Example:**
Your laptop has `192.168.1.15` (private) and your router has a public IP like `49.36.181.20`. When you browse the internet, the router swaps the private address for the public one using NAT.

| Feature | Private IP | Public IP |
|---------|-----------|-----------|
| Ranges | 10.x, 172.16–31.x, 192.168.x | Everything else |
| Routable on internet | No | Yes |
| Uniqueness | Can repeat across networks | Globally unique |
| Who assigns | Internally (you or DHCP) | ISP / IANA |
| Used for | Internal communication | Public-facing servers, internet access |
| Accessible from internet | No | Yes (if exposed/firewall allows) |

**Common Misconception:**
You can't host a public-facing server from a private IP without port forwarding, a public IP, or something like a tunnel/Cloudflare — this trips up a lot of backend developers.

**Interview Follow-up:**
- What are the reserved private IP ranges?
- How does NAT let multiple private devices share one public IP?
- Can a server with a private IP be reached from outside? How?

**Short Interview Answer:**
A private IP is used inside a network and isn't routable on the internet, while a public IP is globally unique and routable. Networks use private IPs internally and reach the internet through NAT, which translates them to a public IP.

---

## 82. What is APIPA?

**Answer:**
APIPA stands for Automatic Private IP Addressing, and it's a fallback mechanism where a machine assigns itself an IP address from `169.254.0.0/16` when it cannot get an IP from a DHCP server. Windows calls it "169.254.x.x" link-local addressing. Seeing this address usually means DHCP failed — the machine is on the network but not properly configured.

**Example:**
If your laptop shows `169.254.23.11`, it means the laptop couldn't reach the DHCP server, so it self-assigned an address. This is a very common troubleshooting clue.

| Feature | Value |
|---------|-------|
| Range | 169.254.0.0/16 |
| Assigned by | The device itself |
| When used | DHCP fails |
| Scope | Local link only |
| Meaning for troubleshooting | DHCP failure, or a cable/VLAN issue |

**Common Misconception:**
APIPA is not a real network connection — it won't reach the internet. If you see it, DHCP isn't working.

**Interview Follow-up:**
- Why is the APIPA range `169.254.x.x`?
- What is the equivalent on Linux?
- What should you check first if a machine gets an APIPA address?

**Short Interview Answer:**
APIPA is a fallback where a device self-assigns an IP from `169.254.0.0/16` when DHCP fails. If you see that address, it means DHCP isn't working — check the cable, the VLAN, and the DHCP server.

---

## 83. What is traceroute and how does it work?

**Answer:**
Traceroute shows the path packets take from your machine to a destination, hop by hop. It sends packets with an increasing Time to Live (TTL) value — with TTL 1 the first router drops it and returns an ICMP Time Exceeded, with TTL 2 the second router does, and so on. Once the TTL is high enough, the destination returns an echo reply and the trace is complete.

**Example:**
`traceroute google.com` shows each router's IP and the round-trip time at each hop, which helps you find where latency or packet loss is being introduced.

| Tool | Shows | Best For |
|------|-------|----------|
| `traceroute` (Linux/Mac) | Path and per-hop latency | Finding where latency occurs |
| `tracert` (Windows) | Same, Windows syntax | Same |
| `mtr` | Continuous traceroute + ping | Live diagnosis |

**Interview Follow-up:**
- Why does the path sometimes change between runs?
- What does a `*` (asterisk) in the output mean?
- Why is ICMP blocked in many networks, and how does that affect traceroute?

**Short Interview Answer:**
Traceroute maps the path to a destination by sending packets with increasing TTL values, so each router on the way returns a Time Exceeded message. The per-hop timings help you pinpoint where latency or packet loss is added.

---

## 84. What is the difference between ping and traceroute?

**Answer:**
`ping` tests whether a host is reachable and measures the round-trip time, using ICMP Echo Request and Reply. `traceroute` shows the full path the packet takes, listing every router in between. So ping answers "is it up and how slow", and traceroute answers "where is it slow".

**Example:**
If `ping 8.8.8.8` shows high latency but `traceroute` shows the jump at hop 7, you know the problem is on that ISP's link, not on your machine or the destination.

| Feature | ping | traceroute |
|---------|------|------------|
| Shows | Reachability + RTT | Every hop in the path |
| Protocol used | ICMP Echo | ICMP with varying TTL |
| Answers | "Is it up, and how slow?" | "Where is it slow?" |
| Output | Single RTT | List of hops with RTTs |

**Common Misconception:**
Ping succeeding doesn't mean the website works. ICMP can be blocked while HTTP still works fine, and the reverse is also possible.

**Interview Follow-up:**
- Why can ping fail while the website loads?
- What does 100% packet loss mean in ping?
- What is `mtr` and how is it different?

**Short Interview Answer:**
Ping checks reachability and round-trip time to a host, while traceroute shows each router in the path and its latency. Ping tells you if it's slow, traceroute tells you where.

---

## 85. What happens when you type a URL in a browser?

**Answer:**
The browser first parses the URL into protocol, host, port, and path, then checks its own cache to see if the page is already loaded. If not, it does a DNS lookup to get the IP address, then opens a TCP connection (3-way handshake), and if it's HTTPS it does a TLS handshake to get a secure connection. After that it sends the HTTP request, the server responds with status code, headers, and body, the browser renders the HTML, and then it makes more requests for CSS, JS, and images.

**Example:**
Typing `https://www.google.com` triggers DNS resolution, TCP connect, TLS handshake, then an HTTP GET, and the response HTML triggers dozens of follow-up requests for scripts and images.

**Steps:**
1. URL parsing (scheme, host, port, path)
2. Browser cache / service worker check
3. DNS resolution to get the IP address
4. TCP 3-way handshake
5. TLS handshake (for HTTPS)
6. HTTP request sent
7. Server responds (status, headers, body)
8. Browser parses HTML, then fetches CSS/JS/images
9. Rendering and paint on screen

**Common Misconception:**
DNS is not "converted" once per page — with HTTP/2 and HTTP/3, all sub-resources are multiplexed over one connection, which is a big performance win.

**Interview Follow-up:**
- What is the difference between this and the old HTTP/1.1 behaviour of opening multiple connections?
- What is the preconnect / preload hint?
- Where does the browser cache fit in this flow?

**Short Interview Answer:**
The browser parses the URL, checks its cache, resolves DNS, opens a TCP connection, does a TLS handshake for HTTPS, sends the HTTP request, and renders the response plus sub-resources. In HTTP/2 and 3 those follow-up requests are multiplexed over a single connection.

---

## 86. What is ARQ?

**Answer:**
ARQ stands for Automatic Repeat reQuest, and it's the fundamental error-recovery mechanism in data link layer protocols. The sender sends a frame with a sequence number, the receiver sends an acknowledgement, and if the sender doesn't get the ACK in time it retransmits. It converts an unreliable physical layer into a reliable link, and TCP is essentially ARQ implemented in software at the transport layer.

**Example:**
If a frame is corrupted in transit, the receiver's CRC check fails, it discards the frame and doesn't ACK it, so the sender times out and retransmits.

| ARQ Type | Behaviour | Efficiency |
|----------|-----------|-----------|
| Stop-and-Wait ARQ | Send one frame, wait for ACK | Low, one frame at a time |
| Go-Back-N ARQ | Sliding window, resend from error | Medium |
| Selective Repeat ARQ | Resend only the lost frames | High |
| TCP | Sliding window + fast retransmit | High |

**Common Misconception:**
ARQ is a link-layer concept, but TCP uses the same principle at the transport layer — which is why TCP is called a reliable ARQ protocol.

**Interview Follow-up:**
- Why is Stop-and-Wait ARQ inefficient and what fixes it?
- What is a sequence number and why do we need it?
- How does Go-Back-N differ from Selective Repeat?

**Short Interview Answer:**
ARQ is the automatic retransmission mechanism that makes an unreliable link reliable — the sender retransmits frames that aren't acknowledged. Stop-and-Wait is the simple version, while Go-Back-N and Selective Repeat use sliding windows for better efficiency, and TCP is ARQ at the transport layer.

---

## 87. What is fragmentation in networking?

**Answer:**
Fragmentation is splitting a large packet into smaller pieces so they can pass through a link with a smaller Maximum Transmission Unit (MTU). IPv4 routers can fragment packets, but IPv6 does not allow routers to fragment — only the sender can. Fragmentation increases overhead and reassembly complexity, and it's a common cause of performance issues.

**Example:**
A 4000-byte IP packet arriving at a link with a 1500-byte MTU must be split into three smaller fragments, each with its own IP header. Reassembly happens at the destination.

**Key Terms:**
| Term | Meaning |
|------|---------|
| MTU | Maximum bytes in a frame on a link |
| Fragment | A smaller piece of an IP packet |
| Fragment Offset | Position of the fragment in the original |
| DF (Don't Fragment) | Flag asking routers not to fragment |

**Common Misconception:**
MTU issues are a classic cause of "website works but VPN/ping to a host doesn't." Path MTU discovery fixes this, and the usual culprit is a tunnel adding header overhead.

**Interview Follow-up:**
- What is Path MTU Discovery and how does it work?
- What causes MTU black hole issues with VPNs?
- How does IPv6 handle fragmentation differently?

**Short Interview Answer:**
Fragmentation splits large packets to fit a link's MTU, with each fragment carrying its own header. IPv4 routers can fragment, IPv6 routers cannot, and fragmentation is often avoided using Path MTU Discovery or smaller packets.

---

## 88. What is tunneling in networking?

**Answer:**
Tunneling is wrapping one protocol's packet inside another protocol's packet so it can travel across a network that doesn't natively support the inner protocol. The original packet is encapsulated, sent through the tunnel, and decapsulated at the other end. It is the foundation of VPNs, SSH, and GRE.

**Example:**
A VPN wraps your original IP packet (with source 10.0.0.5) inside a new packet with a public IP, sends it across the internet, and the VPN gateway unwraps and forwards it to the internal network.

| Tunnel Type | Encapsulated In | Use Case |
|-------------|-----------------|----------|
| IP-in-IP | IP | Simple tunneling |
| GRE | IP (protocol 47) | Site-to-site VPN |
| IPsec Tunnel Mode | IP | Encrypted VPN |
| SSH Tunnel | TCP | Secure port forwarding |
| VXLAN | UDP | Data center / Kubernetes overlays |

**Common Misconception:**
A tunnel adds headers, which increases overhead and reduces the effective MTU — which is exactly why MTU black holes happen with VPNs.

**Interview Follow-up:**
- How is tunneling different from NAT?
- What is VXLAN and why is it used in Kubernetes?
- How does tunneling affect MTU and performance?

**Short Interview Answer:**
Tunneling wraps packets from one protocol inside another so they can cross networks that don't support the inner protocol, and it's the basis of VPNs, SSH, and GRE. The added headers increase overhead and reduce effective MTU.

---

## 89. What is Software Defined Networking (SDN)?

**Answer:**
SDN separates the network's control plane (where routing decisions are made) from the data plane (where packets are forwarded). Instead of each router running its own routing logic, a central controller programs the forwarding tables of all switches via APIs like OpenFlow. This makes network management programmable, so you can automate changes and apply policy across the whole network from one place.

**Example:**
In a data center with 500 switches, an SDN controller can instantly push a new VLAN policy to all of them, which would take weeks if done manually on each device.

| Aspect | Traditional Network | SDN |
|--------|---------------------|-----|
| Control plane | On every device | Centralized controller |
| Configuration | Per device, manual | Programmatic, via API |
| Change speed | Slow, manual | Automated, seconds |
| Consistency | Can differ per device | Central policy enforcement |
| Example | CLI per router | OpenFlow, Cisco ACI, VMware NSX |

**Common Misconception:**
SDN does not replace routing protocols — OSPF and BGP still run and provide reachability info. SDN typically sits above them and programs forwarding based on that information.

**Interview Follow-up:**
- What is the OpenFlow protocol?
- What are the three planes of SDN (application, control, data)?
- What is the CAP theorem and how does it apply to SDN controllers?

**Short Interview Answer:**
SDN moves the network's control logic from individual devices into a central programmable controller that programs the switches' forwarding tables. It makes changes automated and consistent across the network, which is huge for large data centers.

---

## 90. What is Network Function Virtualization (NFV)?

**Answer:**
NFV is about running network functions like firewalls, load balancers, and routers as software on generic servers instead of dedicated hardware appliances. It's a companion idea to SDN — SDN decides how traffic is forwarded, while NFV provides the network services themselves in software. NFV allows faster deployment, cheaper scaling, and running functions in containers or virtual machines.

**Example:**
A telecom operator running a virtual firewall instead of a physical appliance can add capacity in minutes, and a company can run a virtual load balancer on Kubernetes instead of buying a hardware appliance.

| Aspect | Hardware Appliance | NFV |
|--------|-------------------|-----|
| Deployment | Weeks (ship + install) | Minutes |
| Scaling | Buy more hardware | Add a VM / container |
| Flexibility | Fixed function | Any function on any host |
| Cost | High upfront | OpEx, pay as you grow |
| Examples | Cisco ASA box | Open vSwitch, OVN, VyOS, Firewalls in containers |

**Interview Follow-up:**
- How is NFV different from SDN?
- What is VNF and what is CNF?
- Where have you seen virtualized network functions used?

**Short Interview Answer:**
NFV means running network functions like firewalls and load balancers as software on general-purpose servers rather than dedicated hardware. It makes deployment and scaling much faster, and it's commonly used in telecom and container platforms.

---

## 91. What is edge computing in networking?

**Answer:**
Edge computing means processing data closer to where it's generated, instead of sending everything to a central cloud data center. In networking terms, it means putting compute and storage at the network edge — CDNs, IoT gateways, and regional servers. This reduces latency, saves bandwidth, and lets devices work even when the cloud connection is slow or down.

**Example:**
A smart CCTV camera with edge AI detects motion locally and only uploads a clip to the cloud when something happens, instead of streaming 24/7 video over the internet.

| Aspect | Centralized Cloud | Edge Computing |
|--------|-------------------|-----------------|
| Latency | Higher, round trip to distant region | Very low, local processing |
| Bandwidth use | High, all data sent out | Low, only what's needed |
| Offline support | Poor | Works offline |
| Cost model | Pay per request | Higher upfront, lower ongoing |
| Use cases | Batch processing, big data | IoT, video, gaming, AR/VR |

**Common Misconception:**
Edge computing doesn't replace the cloud — it complements it. The edge handles time-sensitive work, and the cloud handles heavy long-term analysis.

**Interview Follow-up:**
- How does edge computing relate to 5G and low latency?
- What is CDN edge vs compute edge?
- What are the challenges of managing distributed edge infrastructure?

**Short Interview Answer:**
Edge computing processes data closer to where it's generated, at the network edge, instead of sending everything to a central cloud. It reduces latency and bandwidth use, and it's key for IoT, video, and 5G applications.

---

## 92. What is the difference between cloud networking and traditional networking?

**Answer:**
Cloud networking refers to network connectivity that supports cloud resources — VPCs, subnets, security groups, load balancers, and internet gateways provided as a service. Traditional networking is managed with physical or virtual devices you configure yourself, with fixed hardware and manual cabling. Cloud networking is virtual, API-driven, and elastic, while traditional networking gives you more physical control but is harder to scale.

**Example:**
In AWS, a VPC with subnets, route tables, and a security group is defined in the console or via API in minutes. The equivalent on-premise setup would need physical switches, routers, and firewalls configured manually.

| Aspect | Traditional Network | Cloud Network |
|--------|----------------------|----------------|
| Provisioning | Days to weeks, hardware | Minutes, self-service API |
| Scaling | Buy new hardware | Elastic, virtual |
| Control | Physical access, full control | Virtual, config-level control |
| Failure domain | Physical hardware failure | Designed for redundancy |
| Cost | CapEx upfront | OpEx, pay-as-you-go |
| Example | Office LAN with switches | AWS VPC, Azure VNet |

**Common Misconception:**
Cloud networking is not automatically more secure or more insecure — it just changes where responsibility lies. The provider secures the infrastructure, and you secure your configuration (IAM, security groups, encryption).

**Interview Follow-up:**
- What is a VPC/VNet and how does it map to a traditional network?
- What is a transit gateway or virtual private link?
- What are the security responsibilities that shift to you in cloud?

**Short Interview Answer:**
Traditional networking uses physical devices you configure and scale manually, while cloud networking is virtual, API-driven, and elastic — like AWS VPCs. The main shift is that provisioning goes from weeks to minutes, but you become responsible for the security configuration.

---

## 93. What is IoT networking?

**Answer:**
IoT networking is about connecting small, low-power, often battery-operated devices like sensors, smart meters, and wearables to the internet or each other. These devices typically use short-range, low-power protocols like ZigBee, BLE, LoRa, or Wi-Fi, and they need to handle constraints like tiny memory and long battery life. The main challenges are power consumption, interference, and security at scale.

**Example:**
A smart agriculture field has hundreds of soil moisture sensors on LoRa, sending data to a gateway that forwards it to a cloud server via MQTT.

| Protocol | Range | Power | Typical Use |
|---------|-------|-------|-------------|
| BLE | ~10–100 m | Low | Wearables, beacons |
| ZigBee | ~100 m | Very low | Home automation mesh |
| Wi-Fi | ~50 m | High | Smart home devices, cameras |
| LoRa | ~2–10 km | Very low | Smart meters, agriculture |
| NB-IoT | km range, cellular | Very low | Large-scale city sensors |

**Common Misconception:**
IoT devices are directly reachable from the internet in most designs — usually they sit behind a gateway, and communication is outbound-initiated. Exposing them directly is a security risk.

**Interview Follow-up:**
- Why can't IoT devices just use normal Wi-Fi protocols?
- What is MQTT and why is it used in IoT?
- How do you secure millions of IoT devices?

**Short Interview Answer:**
IoT networking connects small, low-power devices like sensors and smart meters using short-range protocols like BLE, ZigBee, and LoRa. The key challenges are battery life, interference, and securing thousands of devices at once.

---

## 94. What is TTL in IP packets?

**Answer:**
TTL stands for Time to Live, and it's a field in the IP header that limits how many hops a packet can travel through. Every time a router forwards a packet, it decrements the TTL by 1, and when it reaches 0 the router drops the packet and sends back an ICMP Time Exceeded message. This prevents packets from circulating forever in a routing loop.

**Example:**
A packet with TTL 64 can cross at most 64 routers. `traceroute` exploits exactly this by sending packets with TTL 1, 2, 3, and so on to discover each router in the path.

**Common Misconception:**
TTL is not about time in seconds — it's a hop count. Even though the field is called "Time to Live", it's measured in routers passed, not in seconds.

**Interview Follow-up:**
- How does traceroute use TTL?
- Why was TTL named "Time to Live" if it's a hop count?
- What is the typical initial TTL value on Windows and Linux?

**Short Interview Answer:**
TTL is a field in the IP header that decrements at every router, and the packet is dropped when it reaches 0. This prevents infinite routing loops, and tools like traceroute use it deliberately to map the path.

---

## 95. What is checksum in networking?

**Answer:**
A checksum is a calculated value added to a packet so the receiver can verify the data arrived intact. The receiver recalculates it over the received data and compares — if they don't match, the packet is discarded. TCP and UDP use a checksum for reliability, and IP, ICMP, and Ethernet headers also carry checksums, though IPv6 header checksum was removed for performance.

**Example:**
If one bit of a TCP segment flips in transit, the receiver's checksum doesn't match and the segment is dropped, and TCP then retransmits it.

| Where | Checksum Present |
|-------|-------------------|
| TCP header + data | Yes |
| UDP (optional in IPv4) | Yes |
| IPv4 header | Yes |
| IPv6 header | No (removed) |
| Ethernet frame | Yes (CRC) |

**Common Misconception:**
A checksum only detects errors — it does not correct them. Recovery requires retransmission or forward error correction.

**Interview Follow-up:**
- What's the difference between a checksum and a CRC?
- Why is the UDP checksum optional in IPv4 but mandatory in IPv6?
- How does a checksum help against tampering?

**Short Interview Answer:**
A checksum is a value computed over a packet so the receiver can detect corruption in transit — if the recomputed value doesn't match, the packet is dropped. TCP uses it for reliability, and TCP retransmits when it fails.

---

## 96. What is hop count?

**Answer:**
A hop count is the number of routers (intermediate devices) a packet passes through between source and destination. It's used by routing protocols like RIP that use hop count as their metric, by traceroute to show path length, and by protocols like BGP in limited ways. Fewer hops usually means lower latency, but it's not always true — a path with more hops can sometimes be faster if those links are less congested.

**Example:**
If a packet goes PC → router → ISP router → Google's router → Google server, that's 3 hops. Traceroute will show 3 intermediate routers.

| Concept | Meaning |
|---------|---------|
| Hop | One router-to-router segment |
| Hop count | Total routers traversed |
| Used by | RIP (metric), traceroute, TTL |
| Limitation | Doesn't account for link quality or bandwidth |

**Common Misconception:**
Fewer hops doesn't always mean faster — a 3-hop path over congested links can be slower than a 5-hop path over clean fiber. OSPF uses cost (based on bandwidth) rather than hop count for this reason.

**Interview Follow-up:**
- Why is hop count a poor metric in modern networks?
- How does OSPF's cost differ from RIP's hop count?
- What is the maximum hop count in RIP and why?

**Short Interview Answer:**
Hop count is the number of routers a packet passes through. It's used by RIP as its metric and by traceroute to show the path, but it's a poor metric because it ignores link quality and bandwidth.

---

## 97. What is the difference between goodput, throughput, and bandwidth?

**Answer:**
Bandwidth is the theoretical maximum capacity of a link, like the width of a pipe. Throughput is the actual amount of data that successfully passes per unit of time after overhead. Goodput is the useful application-level data delivered, excluding protocol overhead like headers and retransmissions. So bandwidth is the maximum, throughput is what's actually moving, and goodput is what's actually useful.

**Example:**
On a 100 Mbps link, HTTP with TLS headers and some retransmissions might give 85 Mbps throughput, and the actual file content the user downloads — goodput — could be 80 Mbps.

| Term | Definition | Includes Overhead? |
|------|-----------|-------------------|
| Bandwidth | Theoretical max capacity | N/A (it's the capacity) |
| Throughput | Actual data rate achieved | Yes |
| Goodput | Useful data delivered to the app | No — overhead excluded |
| Latency | Delay for one packet | N/A |

**Common Misconception:**
Bandwidth ≠ speed in general. A 100 Mbps link with 300 ms latency can feel slower than a 20 Mbps link with 10 ms latency — especially for interactive work.

**Interview Follow-up:**
- How does TCP retransmission affect goodput?
- Why can high bandwidth still give a poor user experience?
- What is bandwidth-delay product and why does it matter?

**Short Interview Answer:**
Bandwidth is the link's maximum capacity, throughput is the actual data rate achieved, and goodput is the useful data the application receives. Throughput and goodput are lower than bandwidth because of protocol overhead and retransmissions.

---

## 98. What are network sockets and ports?

**Answer:**
A port is a 16-bit number (0–65535) that identifies a specific service or process on a machine, and a socket is the combination of IP address, port, and protocol that a connection uses. Well-known ports below 1024 are reserved for standard services like HTTP (80) and HTTPS (443), while ports above 1024 are for custom applications. In Java, you bind a server to a port and the OS hands each incoming connection a new socket.

**Example:**
A Spring Boot app on port 8080 means the OS listens on 8080, and each incoming request gets its own socket with a different client-side port. This is why a server can handle thousands of simultaneous connections despite having just one listening port.

**Common Ports:**
| Port | Protocol | Service |
|------|----------|---------|
| 20 / 21 | TCP | FTP |
| 22 | TCP | SSH |
| 25 | TCP | SMTP |
| 53 | UDP/TCP | DNS |
| 80 | TCP | HTTP |
| 443 | TCP/TCP+UDP | HTTPS / HTTP/3 (QUIC) |
| 3306 | TCP | MySQL |
| 5432 | TCP | PostgreSQL |
| 8080 | TCP | Common app port (Spring Boot, Tomcat) |

**Common Misconception:**
A port is not a security control. Anything bound to `0.0.0.0` is reachable from the network unless a firewall blocks it — use firewall rules, not port choice, for security.

**Interview Follow-up:**
- What is a 4-tuple and 5-tuple socket address?
- What is the ephemeral port range and why does it matter?
- Can two apps use the same port on the same machine?

**Short Interview Answer:**
A port is a 16-bit number identifying a service on a machine, and a socket is the full address — IP plus port plus protocol. Standard services use well-known ports like 80, 443, and 3306, while user applications get dynamic ports.

---

## 99. What is the Berkeley sockets API?

**Answer:**
The Berkeley sockets API, originally from BSD Unix, is the standard programming interface for network communication in almost every language. It provides functions like `socket()`, `bind()`, `listen()`, `accept()`, `connect()`, `send()`, and `recv()` to create and manage connections. In Java, `java.net.Socket` and `ServerSocket` are re-implementations of this same model, just wrapped in classes and exceptions.

**Example:**
A basic TCP echo server in C:
```c
int s = socket(AF_INET, SOCK_STREAM, 0);
bind(s, (struct sockaddr*)&addr, sizeof(addr));
listen(s, 5);
int c = accept(s, NULL, NULL);
send(c, "Hello\n", 6, 0);
recv(c, buf, sizeof(buf), 0);
```

**Java equivalent:**
```java
try (ServerSocket server = new ServerSocket(8080)) {
    Socket client = server.accept();
    client.getOutputStream().write("Hello\n".getBytes());
}
```

**Common Misconception:**
Sockets are not the same as WebSockets. A socket is an OS-level connection primitive; a WebSocket is a protocol on top of an HTTP connection for messaging.

**Interview Follow-up:**
- What is the difference between a blocking and non-blocking socket?
- What are `select`, `epoll`, and `kqueue`?
- Why is connection pooling important for database sockets?

**Short Interview Answer:**
The Berkeley sockets API is the standard low-level interface for TCP/UDP communication, and almost every language implements it. In Java, `Socket` and `ServerSocket` wrap the same concepts, and frameworks like Netty and Tomcat build on them.

---

## 100. What are common network troubleshooting tools like ping, traceroute, netstat, nslookup, and Wireshark?

**Answer:**
Each tool answers a different question — `ping` checks reachability, `traceroute` shows the path, `nslookup`/`dig` checks DNS, `netstat` shows active connections and listening ports, and `Wireshark` captures and analyzes actual packets. Using them in the right order quickly narrows down whether the problem is DNS, connectivity, path, port, or the application. For a backend developer, `curl -v` and Postman/Chrome DevTools are the everyday tools for HTTP-level issues.

**Example:**
If a Spring Boot service is not reachable: use `nslookup` to confirm DNS, `ping` for host reachability, `netstat` to confirm it's listening on the right port, `curl -v` to test HTTP, and `traceroute` if it's a network path issue.

| Tool | Question It Answers | Example Command |
|------|---------------------|------------------|
| ping | Is the host reachable? | `ping google.com` |
| traceroute / tracert | Where is the path breaking? | `traceroute google.com` |
| nslookup / dig | Is DNS resolving? | `nslookup myapp.local` |
| netstat / ss | What ports are open/used? | `netstat -ano \| findstr 8080` |
| Wireshark | What is actually on the wire? | Filter `tcp.port == 8080` |
| curl | Does the HTTP call succeed? | `curl -v https://api.example.com` |
| telnet / nc | Is the port open? | `nc -zv host 8080` |

**Common Misconception:**
`ping` failing does not always mean the host is down — ICMP is commonly blocked by firewalls while the actual service is working fine. Always test the actual port with `curl` or `nc`.

**Interview Follow-up:**
- A user says "the site is slow" — how do you troubleshoot it step by step?
- How do you check if a specific port is open on a remote host?
- How do you capture HTTPS traffic if it's encrypted?

**Short Interview Answer:**
`ping` checks reachability, `traceroute` shows the path, `nslookup` checks DNS, `netstat` shows connections and listening ports, and Wireshark shows the actual packets. Using them in order isolates whether the issue is DNS, network, port, or application.

