OSI Model (Networking Basics)

The OSI (Open Systems Interconnection) Model explains how network communication happens by dividing it into 7 layers.

Each layer has a specific responsibility in sending data from one computer to another.

7  Application
6  Presentation
5  Session
4  Transport
3  Network
2  Data Link
1  Physical
Easy Way to Remember
All People Seem To Need Data Processing
OSI Layers Overview
Layer	Name	What It Does	Example
7	Application	User-level network communication	HTTP, DNS
6	Presentation	Data encryption & formatting	TLS/SSL
5	Session	Maintains connection sessions	Login session
4	Transport	Application communication via ports	TCP, UDP
3	Network	Packet routing using IP	IP address
2	Data Link	Communication inside local network	MAC address
1	Physical	Transmits bits through hardware	cables, wifi
Layer-by-Layer Explanation
7️⃣ Application Layer

The Application Layer is where applications communicate with the network.

It defines protocols used by applications.

Examples:

HTTP
HTTPS
DNS
FTP
SMTP

Example request:

GET /products
POST /login

Example:

Opening a website in your browser.

6️⃣ Presentation Layer

The Presentation Layer formats and encrypts data before transmission.

Responsibilities:

encryption

compression

data formatting

Example:

HTTPS encryption (TLS/SSL)

Example:

Browser encrypts data before sending it to a website.
5️⃣ Session Layer

The Session Layer manages sessions between two communicating systems.

Responsibilities:

establish session

maintain session

terminate session

Example:

User login session
Database connection session
Video call session

Example:

User logs into a website and remains authenticated.
4️⃣ Transport Layer

The Transport Layer controls communication between applications using ports.

Protocols:

TCP
UDP

Example:

192.168.1.50:80
192.168.1.50:3306

Meaning:

IP → machine
Port → application

Example services:

Service	Port
HTTP	80
HTTPS	443
SSH	22
MySQL	3306

Example:

Browser → Server:443 (HTTPS)
3️⃣ Network Layer

The Network Layer is responsible for routing packets using IP addresses.

Responsibilities:

IP addressing

packet routing

path determination

Example:

192.168.1.10 → 192.168.1.50

Example command:

ping 8.8.8.8

Example in Kubernetes:

Pod A → Pod B
2️⃣ Data Link Layer

The Data Link Layer handles communication within the same local network.

It uses MAC addresses.

Example MAC address:

00:1A:2B:3C:4D:5E

Example devices:

Laptop → Switch
Server → Router

Protocols:

Ethernet
ARP

Example:

Your laptop sends a packet to your home router.
1️⃣ Physical Layer

The Physical Layer transmits raw bits through hardware.

Examples:

Ethernet cable
Fiber optic cable
WiFi signals
Network cards

Data transmitted:

0s and 1s

Example:

Electrical signals traveling through a network cable.
How All Layers Work Together (Example)

When you open a website:

https://google.com

The layers work like this:

Layer	What Happens
Application	Browser sends HTTP request
Presentation	Data encrypted using TLS
Session	Session established
Transport	TCP connection created
Network	Packet routed using IP
Data Link	MAC address used inside LAN
Physical	Data transmitted through cable/WiFi
Important Layers for DevOps / Kubernetes

In cloud and Kubernetes networking, we mainly deal with:

Layer	Example
L3	Pod IP communication
L4	Service port routing
L7	Ingress HTTP routing

Example flow:

Client
 ↓
Ingress (L7)
 ↓
Service (L4)
 ↓
Pod IP (L3)
Quick Interview Summary
Layer 3 → IP communication between hosts
Layer 4 → Port communication between applications
Layer 7 → Application-level communication (HTTP, APIs)

Example:

ping → Layer 3
curl → Layer 4
HTTP API → Layer 7