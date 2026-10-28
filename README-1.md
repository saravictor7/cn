# Computer Networks Lab PBL — 40 Marks
**Sanjivani University | AIML | 5th Semester | Computer Networks Lab (24UAMPC301)**

This README provides step-by-step setup, source code, expected outputs, and Cisco Packet Tracer instructions based on the supplied **PBL_40_Marks_Lab.pdf**.

> **Important:** The supplied PDF includes complete Java code for Experiments 3–8 and 11–13, plus Packet Tracer tasks and diagnostic commands. It does not provide full code for every experiment in the semester plan. Experiments 1, 2, 9, and 10 are listed in the plan but do not have complete worked solutions in this PDF. The Packet Tracer screenshots are examples; your interface may look different.

## Table of Contents
1. [Requirements and basic setup](#1-requirements-and-basic-setup)
2. [Experiment 3 — Basic TCP socket programming in Java](#2-experiment-3--basic-tcp-socket-programming-in-java)
3. [Experiment 4 — UDP Echo Server](#3-experiment-4--udp-echo-server)
4. [Experiment 5 — UDP Daytime Server](#4-experiment-5--udp-daytime-server)
5. [Experiment 6 — TCP Echo Server](#5-experiment-6--tcp-echo-server)
6. [Experiment 7 — TCP Daytime Server](#6-experiment-7--tcp-daytime-server)
7. [Experiment 8 — CRC in Java](#7-experiment-8--crc-in-java)
8. [Experiment 11 — IPC Message Queue](#8-experiment-11--ipc-message-queue)
9. [Experiment 12 — Stop-and-Wait ARQ](#9-experiment-12--stop-and-wait-arq)
10. [Experiment 13 — Sliding Window](#10-experiment-13--sliding-window)
11. [Cisco Packet Tracer Q1 — Star topology using a hub](#11-cisco-packet-tracer-q1--star-topology-using-a-hub)
12. [Cisco Packet Tracer Q2 — Star topology using a switch](#12-cisco-packet-tracer-q2--star-topology-using-a-switch)
13. [Cisco Packet Tracer Q3 — Connect two LANs using a router](#13-cisco-packet-tracer-q3--connect-two-lans-using-a-router)
14. [Network diagnostic commands](#14-network-diagnostic-commands)
15. [Quick troubleshooting](#15-quick-troubleshooting)
16. [Experiments listed in the plan but not worked out in the PDF](#16-experiments-listed-in-the-plan-but-not-worked-out-in-the-pdf)

---

## 1. Requirements and basic setup

### Software
- Java JDK (for Java programs)
- A terminal / Command Prompt
- Cisco Packet Tracer (for topology questions)

### Check Java
Open Command Prompt / Terminal and run:

```bash
java -version
javac -version
```

If either command is not recognized, install a JDK and ensure its `bin` folder is on your PATH, then reopen the terminal.

### Make a folder for each program
Keep each Java program in its own folder or use the exact file names below. In Java, a `public class` name must match its filename.

### General run steps
1. Open a text editor or IDE.
2. Create a file using the filename specified for that experiment.
3. Paste the full code.
4. Save the file.
5. Open a terminal in that file's folder.
6. Compile with `javac FileName.java`.
7. Run with `java ClassName`.
8. For client/server experiments, open **two terminals**: start the server first, then run the client in the second terminal.

For TCP programs using port numbers, keep the server running while the client connects. `127.0.0.1` and `localhost` refer to your own computer.

---

## 2. Experiment 3 — Basic TCP socket programming in Java

**Aim:** Create a TCP server and client. The client sends a message and the server displays it.

### Step 1 — Create `BasicTCPServer.java`

```java
import java.net.*;
import java.io.*;

public class BasicTCPServer {
    public static void main(String[] args) throws IOException {
        ServerSocket server = new ServerSocket(5000);
        System.out.println("Server is waiting for client...");

        Socket socket = server.accept();
        DataInputStream in =
                new DataInputStream(socket.getInputStream());

        System.out.println("Client says: " + in.readUTF());

        in.close();
        socket.close();
        server.close();
    }
}
```

### Step 2 — Create `BasicTCPClient.java`

```java
import java.net.*;
import java.io.*;

public class BasicTCPClient {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1", 5000);
        DataOutputStream out =
                new DataOutputStream(socket.getOutputStream());

        out.writeUTF("Hello Server");
        out.flush();

        out.close();
        socket.close();
    }
}
```

### Step 3 — Compile
In the folder containing both files:

```bash
javac BasicTCPServer.java
javac BasicTCPClient.java
```

### Step 4 — Run
**Terminal 1:**
```bash
java BasicTCPServer
```

**Terminal 2:**
```bash
java BasicTCPClient
```

### Expected output
Server terminal:
```text
Server is waiting for client...
Client says: Hello Server
```

Client terminal has no output in this program.

---

## 3. Experiment 4 — UDP Echo Server

**Aim:** Send a UDP message from a client; the server prints it and echoes the same message back.

### Step 1 — Create `UDPEchoServer.java`

```java
import java.net.*;

public class UDPEchoServer {
    public static void main(String[] args) throws Exception {
        DatagramSocket server = new DatagramSocket(6000);
        byte[] buffer = new byte[1024];

        System.out.println("UDP Server waiting for client...");

        while (true) {
            DatagramPacket request =
                    new DatagramPacket(buffer, buffer.length);
            server.receive(request);

            String msg = new String(
                    request.getData(), 0, request.getLength());
            System.out.println("Received: " + msg);

            DatagramPacket response = new DatagramPacket(
                    request.getData(),
                    request.getLength(),
                    request.getAddress(),
                    request.getPort());

            server.send(response); // Echo back to sender
        }
    }
}
```

### Step 2 — Create `UDPEchoClient.java`

```java
import java.net.*;

public class UDPEchoClient {
    public static void main(String[] args) throws Exception {
        DatagramSocket client = new DatagramSocket();
        InetAddress serverIP = InetAddress.getByName("localhost");

        String message = "Hello UDP Server";
        byte[] buffer = message.getBytes();

        DatagramPacket packet = new DatagramPacket(
                buffer, buffer.length, serverIP, 6000);
        client.send(packet);

        byte[] recvBuf = new byte[1024];
        DatagramPacket response =
                new DatagramPacket(recvBuf, recvBuf.length);
        client.receive(response);

        System.out.println("Server echoed: " +
                new String(response.getData(), 0, response.getLength()));

        client.close();
    }
}
```

### Step 3 — Compile
```bash
javac UDPEchoServer.java
javac UDPEchoClient.java
```

### Step 4 — Run
**Terminal 1:**
```bash
java UDPEchoServer
```

**Terminal 2:**
```bash
java UDPEchoClient
```

### Expected output
Server:
```text
UDP Server waiting for client...
Received: Hello UDP Server
```

Client:
```text
Server echoed: Hello UDP Server
```

**Note:** The server intentionally keeps running in an infinite loop. Stop it with `Ctrl+C`.

---

## 4. Experiment 5 — UDP Daytime Server

**Aim:** The client requests the current date and time; the server sends it over UDP.

### Step 1 — Create `UDPDayTimeServer.java`

```java
import java.net.*;
import java.util.Date;

public class UDPDayTimeServer {
    public static void main(String[] args) throws Exception {
        DatagramSocket server = new DatagramSocket(9999);
        byte[] buffer = new byte[1024];

        while (true) {
            DatagramPacket request =
                    new DatagramPacket(buffer, buffer.length);
            server.receive(request);

            String date = new Date().toString();
            byte[] responseData = date.getBytes();

            DatagramPacket response = new DatagramPacket(
                    responseData, responseData.length,
                    request.getAddress(), request.getPort());

            server.send(response);
        }
    }
}
```

### Step 2 — Create `UDPDayTimeClient.java`

```java
import java.net.*;

public class UDPDayTimeClient {
    public static void main(String[] args) throws Exception {
        DatagramSocket client = new DatagramSocket();
        InetAddress serverIP = InetAddress.getByName("localhost");

        byte[] request = "Send Date".getBytes();
        DatagramPacket packet = new DatagramPacket(
                request, request.length, serverIP, 9999);
        client.send(packet);

        byte[] responseBuf = new byte[1024];
        DatagramPacket response =
                new DatagramPacket(responseBuf, responseBuf.length);
        client.receive(response);

        System.out.println("Current Date & Time: " +
                new String(response.getData(), 0, response.getLength()));

        client.close();
    }
}
```

### Step 3 — Compile
```bash
javac UDPDayTimeServer.java
javac UDPDayTimeClient.java
```

### Step 4 — Run
**Terminal 1:**
```bash
java UDPDayTimeServer
```

**Terminal 2:**
```bash
java UDPDayTimeClient
```

### Expected output
Client (example; date/time will vary):
```text
Current Date & Time: Wed Oct 28 00:00:00 IST 2026
```

Stop the server with `Ctrl+C` when finished.

---

## 5. Experiment 6 — TCP Echo Server

**Aim:** The client sends a message to the server; the server returns the message with an `Echo:` prefix.

### Step 1 — Create `TCPEchoServer.java`

```java
import java.net.*;
import java.io.*;

public class TCPEchoServer {
    public static void main(String[] args) throws IOException {
        ServerSocket server = new ServerSocket(7000);
        System.out.println("TCP Echo Server waiting...");

        Socket socket = server.accept();
        DataInputStream in =
                new DataInputStream(socket.getInputStream());
        DataOutputStream out =
                new DataOutputStream(socket.getOutputStream());

        String msg = in.readUTF();
        System.out.println("Received from client: " + msg);
        out.writeUTF("Echo: " + msg);

        in.close();
        out.close();
        socket.close();
        server.close();
    }
}
```

### Step 2 — Create `TCPEchoClient.java`

```java
import java.net.*;
import java.io.*;

public class TCPEchoClient {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1", 7000);
        DataOutputStream out =
                new DataOutputStream(socket.getOutputStream());
        DataInputStream in =
                new DataInputStream(socket.getInputStream());

        out.writeUTF("Hello TCP Server");
        out.flush();

        System.out.println("Server response: " + in.readUTF());

        out.close();
        in.close();
        socket.close();
    }
}
```

### Step 3 — Compile
```bash
javac TCPEchoServer.java
javac TCPEchoClient.java
```

### Step 4 — Run
**Terminal 1:**
```bash
java TCPEchoServer
```

**Terminal 2:**
```bash
java TCPEchoClient
```

### Expected output
Server:
```text
TCP Echo Server waiting...
Received from client: Hello TCP Server
```

Client:
```text
Server response: Echo: Hello TCP Server
```

---

## 6. Experiment 7 — TCP Daytime Server

**Aim:** The server sends the current date and time to a TCP client.

### Step 1 — Create `TCPDayTimeServer.java`

```java
import java.net.*;
import java.io.*;
import java.util.Date;

public class TCPDayTimeServer {
    public static void main(String[] args) throws IOException {
        ServerSocket server = new ServerSocket(8000);
        System.out.println("TCP Daytime Server waiting...");

        Socket socket = server.accept();
        PrintWriter out =
                new PrintWriter(socket.getOutputStream(), true);

        out.println("Server Date & Time: " + new Date().toString());

        out.close();
        socket.close();
        server.close();
    }
}
```

### Step 2 — Create `TCPDayTimeClient.java`

```java
import java.net.*;
import java.io.*;

public class TCPDayTimeClient {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1", 8000);

        BufferedReader in = new BufferedReader(
                new InputStreamReader(socket.getInputStream()));

        System.out.println(in.readLine());

        in.close();
        socket.close();
    }
}
```

### Step 3 — Compile
```bash
javac TCPDayTimeServer.java
javac TCPDayTimeClient.java
```

### Step 4 — Run
**Terminal 1:**
```bash
java TCPDayTimeServer
```

**Terminal 2:**
```bash
java TCPDayTimeClient
```

### Expected output
Server:
```text
TCP Daytime Server waiting...
```

Client (the actual date/time will vary):
```text
Server Date & Time: Wed Oct 28 00:00:00 IST 2026
```

---

## 7. Experiment 8 — CRC in Java

**Aim:** Generate a Cyclic Redundancy Check (CRC) remainder using binary division with XOR.

### Step 1 — Create `CRC.java`

```java
public class CRC {
    public static void main(String[] args) {
        String data = "11010011101100";
        String key = "1011";

        String crc = generateCRC(data, key);
        System.out.println("CRC Code: " + crc);
    }

    public static String generateCRC(String data, String key) {
        int keyLen = key.length();
        String appendedData = data + "0".repeat(keyLen - 1);

        char[] dataArr = appendedData.toCharArray();
        char[] keyArr = key.toCharArray();

        for (int i = 0; i < data.length(); i++) {
            if (dataArr[i] == '1') {
                for (int j = 0; j < keyLen; j++) {
                    dataArr[i + j] =
                            (dataArr[i + j] == keyArr[j]) ? '0' : '1';
                }
            }
        }

        return new String(dataArr).substring(data.length());
    }
}
```

### Step 2 — Compile and run
```bash
javac CRC.java
java CRC
```

### Expected output
```text
CRC Code: 100
```

**Concept:** CRC uses modulo-2 binary division. XOR is used instead of ordinary subtraction. The generator key `1011` has length 4, so three zero bits are appended to the data before division, and the remainder has 3 bits.

---

## 8. Experiment 11 — IPC Message Queue

**Aim:** Demonstrate inter-process communication (IPC) style message exchange between a producer thread and a consumer thread using a bounded blocking queue.

### Step 1 — Create `ProducerConsumer.java`

```java
import java.util.concurrent.ArrayBlockingQueue;

class ProducerConsumer {
    public static void main(String[] args) {
        ArrayBlockingQueue<Integer> queue = new ArrayBlockingQueue<>(5);

        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    queue.put(i);
                    System.out.println("Produced: " + i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        Thread consumer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    int val = queue.take();
                    System.out.println("Consumed: " + val);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        producer.start();
        consumer.start();
    }
}
```

### Step 2 — Compile and run
```bash
javac ProducerConsumer.java
java ProducerConsumer
```

### Expected output
The exact interleaving can vary because the producer and consumer run concurrently. One possible output is:

```text
Produced: 1
Produced: 2
Produced: 3
Produced: 4
Produced: 5
Consumed: 1
Consumed: 2
Consumed: 3
Consumed: 4
Consumed: 5
```

**Concept:** `ArrayBlockingQueue` is a bounded, thread-safe queue. `put()` adds an item and may wait if the queue is full; `take()` removes an item and may wait if the queue is empty.

---

## 9. Experiment 12 — Stop-and-Wait ARQ

**Aim:** Demonstrate Stop-and-Wait Automatic Repeat reQuest (ARQ). The sender sends one frame and waits for its acknowledgement before proceeding.

### Step 1 — Create `StopWaitARQ.java`

```java
public class StopWaitARQ {
    public static void main(String[] args) {
        String[] frames = {"Frame1", "Frame2", "Frame3"};
        boolean ack;

        for (String frame : frames) {
            System.out.println("Sending: " + frame);

            ack = true; // Simulate ACK received
            if (ack) {
                System.out.println("ACK received for " + frame);
            } else {
                System.out.println("Resending " + frame);
            }
        }
    }
}
```

### Step 2 — Compile and run
```bash
javac StopWaitARQ.java
java StopWaitARQ
```

### Expected output
```text
Sending: Frame1
ACK received for Frame1
Sending: Frame2
ACK received for Frame2
Sending: Frame3
ACK received for Frame3
```

**Concept:** Only one unacknowledged frame is in transit at a time. This supplied example simulates successful acknowledgements (`ack = true`); it does not implement actual network transmission, timeout, or packet loss.

---

## 10. Experiment 13 — Sliding Window

**Aim:** Demonstrate sending frames in groups according to a window size.

### Step 1 — Create `SlidingWindow.java`

```java
public class SlidingWindow {
    public static void main(String[] args) {
        String[] frames = {"F1", "F2", "F3", "F4", "F5"};
        int windowSize = 3;

        for (int i = 0; i < frames.length; i++) {
            System.out.println("Sending: " + frames[i]);

            if ((i + 1) % windowSize == 0 || i == frames.length - 1) {
                System.out.println("Waiting for ACKs...");
                System.out.println(
                    "ACKs received for window ending at: " + frames[i]);
            }
        }
    }
}
```

### Step 2 — Compile and run
```bash
javac SlidingWindow.java
java SlidingWindow
```

### Expected output
```text
Sending: F1
Sending: F2
Sending: F3
Waiting for ACKs...
ACKs received for window ending at: F3
Sending: F4
Sending: F5
Waiting for ACKs...
ACKs received for window ending at: F5
```

**Concept:** Sliding Window allows multiple frames to be sent before acknowledgements arrive. This supplied code is a simplified simulation; it does not fully implement Go-Back-N or Selective Repeat retransmission logic.

---

## 11. Cisco Packet Tracer Q1 — Star topology using a hub

**Question:** Design a star topology using a hub and demonstrate real-time simulation.

### Step 1 — Place devices
1. Open Cisco Packet Tracer.
2. From **End Devices**, add four PCs (PC0, PC1, PC2, PC3).
3. From **Network Devices → Hubs**, add one hub.
4. Arrange the hub in the middle and the four PCs around it.

### Step 2 — Connect the devices
1. Click **Connections** (lightning icon).
2. Choose **Copper Straight-Through** (or use Automatic connection if allowed by your lab).
3. Connect each PC's FastEthernet interface to a different hub port.

### Step 3 — Assign IP addresses
Use one network for all PCs, for example:

| Device | IPv4 address | Subnet mask |
|---|---|---|
| PC0 | 192.168.1.1 | 255.255.255.0 |
| PC1 | 192.168.1.2 | 255.255.255.0 |
| PC2 | 192.168.1.3 | 255.255.255.0 |
| PC3 | 192.168.1.4 | 255.255.255.0 |

For each PC:
1. Click the PC.
2. Open **Desktop → IP Configuration**.
3. Enter the IPv4 address and subnet mask.
4. No default gateway is required for communication within this single LAN.

### Step 4 — Test connectivity
1. Click PC0 → **Desktop → Command Prompt**.
2. Run:

```text
ping 192.168.1.2
ping 192.168.1.3
ping 192.168.1.4
```

3. Successful replies demonstrate connectivity.
4. Click **Simulation** to inspect packet movement, or use **Realtime** for normal operation.
5. Save the project as `Q1_Star_Hub.pkt`.

**Expected result:** All PCs are connected to the central hub and can ping each other. A hub repeats incoming signals to its ports; it does not learn MAC addresses like a switch.

---

## 12. Cisco Packet Tracer Q2 — Star topology using a switch

**Question:** Design a star topology using a switch and demonstrate real-time simulation.

### Step 1 — Place devices
1. Add four PCs.
2. Add one **2960 switch** from **Network Devices → Switches**.
3. Place the switch in the middle.

### Step 2 — Connect devices
1. Select **Connections → Copper Straight-Through**.
2. Connect each PC's FastEthernet0 port to a separate switch FastEthernet port.

### Step 3 — Configure IP addresses

| Device | IPv4 address | Subnet mask |
|---|---|---|
| PC0 | 192.168.2.1 | 255.255.255.0 |
| PC1 | 192.168.2.2 | 255.255.255.0 |
| PC2 | 192.168.2.3 | 255.255.255.0 |
| PC3 | 192.168.2.4 | 255.255.255.0 |

For each PC: **PC → Desktop → IP Configuration**. Enter the IP and subnet mask.

### Step 4 — Verify
From PC0's Command Prompt:

```text
ping 192.168.2.2
ping 192.168.2.3
ping 192.168.2.4
```

The first ping may time out while ARP resolves the destination; repeat it if necessary.

### Step 5 — Demonstrate simulation and save
1. Click **Simulation**.
2. Use **Add Simple PDU** (envelope icon) and click the source PC, then destination PC.
3. Use **Capture/Forward** or **Auto Capture/Play** to observe the packet.
4. Return to **Realtime** if requested.
5. Save as `Q2_Star_Switch.pkt`.

**Expected result:** All four PCs communicate through the central switch. A switch learns source MAC addresses and forwards frames to the appropriate port when it knows the destination.

---

## 13. Cisco Packet Tracer Q3 — Connect two LANs using a router

**Question:** Connect two LANs using one router and two switches, configure IP addresses, and verify connectivity.

### Suggested addressing plan
Use two different /24 networks.

| Device | Interface / setting | IPv4 address | Subnet mask | Default gateway |
|---|---|---|---|---|
| Router | G0/0 (LAN 1) | 192.168.10.1 | 255.255.255.0 | — |
| Router | G0/1 (LAN 2) | 192.168.20.1 | 255.255.255.0 | — |
| PC0 (LAN 1) | NIC | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC1 (LAN 1) | NIC | 192.168.10.11 | 255.255.255.0 | 192.168.10.1 |
| PC2 (LAN 2) | NIC | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |
| PC3 (LAN 2) | NIC | 192.168.20.11 | 255.255.255.0 | 192.168.20.1 |

> Router interface names vary by model. If your router uses `GigabitEthernet0/0/0` and `GigabitEthernet0/0/1`, use those actual names instead.

### Step 1 — Place devices
1. Add one router (for example, a model with two Ethernet interfaces).
2. Add two switches.
3. Add at least two PCs for each LAN.
4. Place one switch and its PCs on each side of the router.

### Step 2 — Cable the topology
1. Connect LAN 1 PCs to Switch 1 using copper straight-through cables.
2. Connect Switch 1 to the router's first Ethernet interface.
3. Connect LAN 2 PCs to Switch 2.
4. Connect Switch 2 to the router's second Ethernet interface.

### Step 3 — Configure the router
Click the router → **CLI**. If prompted for the initial configuration dialog, answer `no`, then press Enter. Enter the following commands (adjust interface names to match your router):

```text
enable
configure terminal
interface gigabitEthernet 0/0
ip address 192.168.10.1 255.255.255.0
no shutdown
exit
interface gigabitEthernet 0/1
ip address 192.168.20.1 255.255.255.0
no shutdown
exit
end
show ip interface brief
```

If your router uses different interface names, substitute them. Wait for the links to turn green.

### Step 4 — Configure PCs
For each PC, open **Desktop → IP Configuration** and enter the corresponding IP address, subnet mask, and default gateway from the table.

### Step 5 — Verify communication
From PC0 (LAN 1), open **Desktop → Command Prompt** and run:

```text
ping 192.168.10.11
ping 192.168.20.10
ping 192.168.20.11
```

The first ping to a destination can time out while ARP is resolved. Repeat the command if needed.

### Step 6 — Save and demonstrate
1. Use **Simulation** and **Add Simple PDU** to demonstrate traffic between a PC in LAN 1 and a PC in LAN 2.
2. Verify successful ping replies.
3. Save as `Q3_Two_LANs_Router.pkt`.

**Expected result:** PCs within each LAN communicate through their switch, and the router routes packets between the two different IP networks.

---

## 14. Network diagnostic commands

The supplied PDF lists the following commands.

### Windows Command Prompt

```bat
ping www.google.com
ipconfig
ipconfig /all
tracert www.google.com
netstat -ano
nslookup www.google.com
```

### Linux Terminal

```bash
ping -c 4 www.google.com
ifconfig -a
traceroute www.google.com
netstat -tunlp
nslookup www.google.com
```

Some Linux systems may not have `ifconfig`, `traceroute`, or `netstat` installed by default. Modern alternatives include `ip addr`, `tracepath`/`traceroute`, and `ss`.

| Command | Purpose |
|---|---|
| `ping` | Tests reachability using ICMP echo messages. |
| `ipconfig` | Shows basic IP configuration on Windows. |
| `ipconfig /all` | Shows detailed Windows network configuration, including MAC, DHCP, and DNS details. |
| `ifconfig -a` | Shows/configures Linux network interfaces on systems where the tool is installed. |
| `tracert` / `traceroute` | Shows the route (hops) toward a destination. |
| `netstat -ano` | Shows Windows connections/listening ports and process IDs. |
| `netstat -tunlp` | Shows Linux TCP/UDP listening sockets and associated process information (may require elevated privileges). |
| `nslookup` | Queries DNS to resolve domain names and IP addresses. |

### Quick procedure
1. Open Command Prompt on Windows or Terminal on Linux.
2. Type one command and press Enter.
3. Read the displayed IP configuration, reachability, route, connections, or DNS response.
4. If a command is not recognized, check whether it is available on your OS.

---

## 15. Quick troubleshooting

### `javac` is not recognized
- Install a Java Development Kit (JDK).
- Ensure the JDK `bin` folder is included in PATH.
- Close and reopen Command Prompt / Terminal.

### `Could not find or load main class`
- Ensure you are in the directory containing the `.class` file.
- Check spelling and capitalization of the class name.
- Compile first with `javac FileName.java`, then run `java ClassName` without `.java`.

### `Address already in use`
- Another program may be using that port, or an earlier server is still running.
- Stop the previous server with `Ctrl+C`, or change the port in both client and server consistently.

### Client cannot connect
- Start the server first.
- Ensure the client uses the same port as the server.
- Keep both programs on the same computer when using `localhost` or `127.0.0.1`.

### Client hangs waiting for a UDP response
- Confirm the UDP server is running on the correct port.
- Ensure the client and server port numbers match.
- Stop the client with `Ctrl+C` if necessary.

### Packet Tracer pings fail
- Check the cable connections and wait for links to turn green.
- Confirm every PC's IP and subnet mask.
- For two LANs, confirm each PC's default gateway and both router interfaces are configured and enabled with `no shutdown`.
- Repeat the first ping to allow ARP resolution.

---

## 16. Experiments listed in the plan but not worked out in the PDF

The semester plan on page 1 lists these additional experiments, but the supplied PDF does not include complete instructions/code for them:

1. **Experiment 1:** NIC installation/configuration, IP addressing, and subnetting.
2. **Experiment 2:** Basic socket programming using C.
3. **Experiment 9:** DNS server setup/configuration.
4. **Experiment 10:** TELNET server setup/configuration.
5. **Experiment 14:** Familiarization with CAT 5/UTP cables, RJ45/T-connectors, hubs, and switches.

These are included in the semester plan, but full answers are not present in this particular PBL answer PDF. Ask your instructor for the corresponding lab sheet if these are part of your assessed practical.

---

## Viva quick revision

- **TCP:** Connection-oriented transport protocol; reliable, ordered byte stream.
- **UDP:** Connectionless transport protocol; datagrams with no built-in delivery guarantee.
- **Socket:** Endpoint used by a program to communicate over a network.
- **ServerSocket:** Java class used to listen for and accept TCP connections.
- **Socket:** Java endpoint used by a TCP client or accepted TCP connection.
- **DatagramSocket:** Java class used to send/receive UDP datagrams.
- **Port:** Number identifying an application/service endpoint on a host.
- **Echo server:** Sends received data back to the sender.
- **Daytime server:** Returns the current date and time.
- **CRC:** Cyclic Redundancy Check; detects transmission errors using polynomial/modulo-2 division.
- **IPC:** Inter-Process Communication; mechanisms that allow concurrent processes/threads to exchange data.
- **ARQ:** Automatic Repeat reQuest; uses acknowledgements and retransmission to improve reliability.
- **Stop-and-Wait ARQ:** Sends one frame and waits for its ACK before sending the next.
- **Sliding Window:** Allows multiple outstanding frames before acknowledgements arrive.
- **Hub:** Repeats incoming signals to its ports.
- **Switch:** Forwards Ethernet frames based on learned MAC addresses.
- **Router:** Forwards packets between IP networks.
- **DNS:** Domain Name System; resolves domain names to IP addresses.
- **TELNET:** Remote terminal protocol; unencrypted by default.

---

## Checklist before lab submission

- [ ] Every Java file name matches its public class name.
- [ ] Each program compiles successfully.
- [ ] For client/server programs, the server is started before the client.
- [ ] Expected output is visible in the terminal.
- [ ] Packet Tracer topology has correct cabling and addressing.
- [ ] Pings between required PCs succeed.
- [ ] `.pkt` files are saved with clear names.
- [ ] README and screenshots are uploaded to your GitHub repository.
