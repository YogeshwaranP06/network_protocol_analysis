Network Protocol Analysis: TLS 1.3 vs QUIC
By Yogeshwaran P | ECE, SRM TRP Engineering College

A hands-on Wireshark study comparing how modern web traffic establishes secure, encrypted connections — analyzing real captured traffic from GitHub (TCP + TLS 1.3), WhatsApp Web (TLS session resumption), and YouTube (QUIC over UDP).

🎯 Objective
Understand and compare the handshake mechanics of the traditional TCP+TLS stack against the newer QUIC protocol by capturing and dissecting live traffic with Wireshark and Python/Scapy.

🛠️ Tools Used
Wireshark — packet capture and protocol dissection
Python + Scapy — offline .pcapng parsing and TLS/QUIC layer extraction
Capture interface: Wi-Fi (dual-stack IPv4/IPv6, NAT64 network)
1. GitHub — TCP + TLS 1.3 Handshake
A fresh connection to github.com was captured from the first DNS query through to encrypted application data.

Sequence observed:

DNS query resolved github.com → NAT64 IPv6 address mapping real IPv4 20.207.73.82
TCP 3-way handshake: SYN → SYN-ACK → ACK on port 443
TLS Client Hello sent immediately after TCP handshake
TLS Server Hello + Certificate + Change Cipher Spec returned
Client Change Cipher Spec → first encrypted Application Data (page load begins)
Key Client Hello details:

Field	Value
SNI (Server Name)	github.com
TLS Versions Offered	TLS 1.3, TLS 1.2
Cipher Suites	TLS_AES_128_GCM_SHA256, TLS_AES_256_GCM_SHA384, TLS_CHACHA20_POLY1305_SHA256 + 13 legacy suites
Key Exchange Groups	X25519MLKEM768 (post-quantum hybrid), x25519, secp256r1, secp384r1
ALPN	h2 (HTTP/2), http/1.1
Server's response: Chose TLS_AES_128_GCM_SHA256, confirmed TLS 1.3, selected x25519 key share.

🔐 Notable finding: The browser proactively offered X25519MLKEM768 — a post-quantum hybrid key-exchange algorithm — even though GitHub's server chose the classical x25519 curve. This shows quantum-resistant cryptography is already being rolled out in everyday browsing, ahead of any real quantum threat.

2. WhatsApp Web — TLS Session Behaviour
The capture for web.whatsapp.com showed a different pattern from GitHub:

Repeated DNS queries for web.whatsapp.com in a short window — consistent with frequent reconnection/keep-alive checks
TLS Application Data packets appeared without a fresh Client Hello in this capture window
Indicates an already-established TLS session was reused (session resumption) rather than a full handshake
This is a real-world example of TLS session resumption — a performance optimization that skips the full handshake for returning connections.

3. YouTube — QUIC Handshake
A fresh connection to www.youtube.com was captured through to an active QUIC session with a video CDN endpoint (resolved via rr1---sn-q4flrnsd.googlevideo.com).

Sequence observed:

DNS query for www.youtube.com, followed by a separate lookup for the CDN hostname — YouTube separates the main site from its video-serving CDN
Immediately after CDN resolution, a 1292-byte UDP packet was sent directly to port 443 — no TCP SYN at any point
Decoded as a QUIC Initial packet, confirmed via header byte 0xC0 (long header, Initial type)
26 total UDP packets exchanged with this single CDN endpoint, most near the 1292-byte QUIC max datagram size — continuous encrypted video delivery
Extracted QUIC Initial packet details:

Field	Value
Header Form	0xC0 — Long Header, Initial packet type
QUIC Version	0x00000001 (QUIC v1, RFC 9000)
Destination Connection ID	ce0eabe38063c2f9 (8 bytes)
Transport	UDP, source port 60485 → destination port 443
Payload	Encrypted CRYPTO frame (carries TLS 1.3 handshake inside QUIC)
Once the handshake completed, all further packets appeared as encrypted QUIC short-header packets ("Protected Payload") — continuous encrypted video streaming with no further visible handshake structure.

4. TCP+TLS vs QUIC — Comparison
Aspect	TCP + TLS 1.3 (GitHub)	QUIC (YouTube)
Transport Layer	TCP	UDP
Handshake Steps	TCP SYN/SYN-ACK/ACK, then separate TLS Client Hello/Server Hello	Single QUIC Initial packet combines transport + TLS 1.3 handshake
Round Trips to Secure Data	2 round trips (1 TCP + 1 TLS)	1 round trip (0-RTT possible on resumption)
Head-of-Line Blocking	Present — one lost TCP segment blocks all streams	Avoided — each QUIC stream is independent
Best Suited For	General web browsing, APIs, code hosting	Video streaming, real-time or lossy-network use cases
5. Conclusion
This capture-based analysis confirmed the practical differences between legacy TCP+TLS and modern QUIC in real traffic:

GitHub followed the classical multi-step TCP+TLS 1.3 handshake, including an early post-quantum key-exchange offer
WhatsApp Web demonstrated TLS session resumption in action
YouTube's use of QUIC showed how Google collapses the handshake into a single UDP round trip, reducing latency — a major reason QUIC is preferred for video streaming, where every millisecond of start-up delay affects user experience
This exercise strengthened practical understanding of network security fundamentals — directly relevant to embedded systems and IoT work, where secure, low-latency communication design is increasingly important.

🔧 Skills Demonstrated
Wireshark packet capture and display filtering
Python/Scapy for offline .pcapng analysis and TLS layer parsing
TLS 1.2/1.3 handshake structure (Client Hello, Server Hello, Cipher Suites, Key Exchange)
QUIC protocol fundamentals and Connection ID handling
DNS resolution and NAT64 address translation in dual-stack networks
📄 Full report (Word format): Network_Protocol_Analysis_TLS_vs_QUIC (1).docx


