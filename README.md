# bandwagonhost speed test: How to Measure VPS Latency, Bandwidth, Disk I/O, and China Routing Before You Buy

A BandwagonHost speed test should answer more than “how many Mbps did the server reach?” A useful test checks four separate things:

- **Latency:** how quickly packets travel between your users and the VPS
- **Packet loss and jitter:** whether the connection stays stable
- **Network throughput:** how much data the VPS can actually transfer
- **CPU, memory, and disk performance:** whether the server can handle the workload behind the network connection

That distinction matters because a VPS can show an impressive port speed while still feeling slow for SSH, web applications, databases, or users in another country. A 2.5 Gbps uplink is not the same thing as a guaranteed 2.5 Gbps download from every location.

BandwagonHost’s AFF link currently redirects to an **E-Commerce VPS ordering page for the USCA_9 Los Angeles location**. BandwagonHost identifies USCA_9 as a location with China Telecom CN2 GIA, China Mobile CMIN2, and China Unicom Premium connectivity. The company also says VPS services use KVM virtualization and the KiwiVM management panel.

This guide explains how to run a BandwagonHost speed test, how to read the results, and which current USCA_9 plans make sense for different workloads.

## What Does a BandwagonHost Speed Test Actually Measure?

The phrase “speed test” is vague. For a VPS, it usually refers to several different measurements.

### Latency

Latency is the round-trip time between your device and the VPS, usually measured in milliseconds.

A lower number generally means faster interaction:

| Latency | Practical impression |
| --- | --- |
| Under 30 ms | Very responsive for nearby users |
| 30–80 ms | Good for most applications |
| 80–150 ms | Usable for websites, APIs, and remote administration |
| 150–250 ms | Noticeable delay for interactive work |
| Over 250 ms | Poor for real-time interaction, though still usable for some services |

Latency depends heavily on where the test originates. A server in Los Angeles may be quick for users on the US West Coast and much slower for users in Europe or Asia. The same VPS can look excellent from one ISP and mediocre from another.

### Packet Loss

Packet loss occurs when data packets fail to reach their destination or return.

Even a small amount of packet loss can make SSH sessions freeze, increase page-load time, interrupt database connections, or cause VPN and voice traffic to behave badly. A single failed ping does not prove that the server is unreliable, but repeated loss across multiple tests deserves attention.

### Jitter

Jitter is the variation in latency. For example, a connection that returns 40 ms, 42 ms, and 41 ms is more stable than one that returns 25 ms, 180 ms, and 60 ms, even if the average looks similar.

Jitter matters for:

- VoIP and video calls
- Game servers
- Remote desktop sessions
- VPN gateways
- Interactive APIs
- SSH administration over long-distance routes

### Throughput

Throughput measures how quickly data moves. It is normally shown in Mbps or Gbps.

A speed test can measure:

- Download speed to the VPS
- Upload speed from the VPS
- Transfer speed between the VPS and a selected test server
- Speed between the VPS and your own network

The result is affected by both ends of the connection. If the remote test server is busy or geographically distant, the number may not represent the VPS’s full capacity.

### Disk and CPU Performance

A website can have a fast network and still load slowly because the VPS has limited CPU or poor disk performance.

A complete BandwagonHost performance test should therefore include:

- Sequential disk read and write
- Random disk I/O
- CPU performance
- Memory bandwidth
- Network throughput
- Latency to the locations that matter to you

## How to Run a BandwagonHost Speed Test

Run the tests from the network where your real users are located. Testing from a US office does not tell you much about the experience of users in Europe, India, or mainland China.

### 1. Check Basic Connectivity

After activating the VPS, connect through SSH:

bash
ssh root@YOUR_SERVER_IP


Replace `YOUR_SERVER_IP` with the address shown in KiwiVM.

Start with a basic ping test from your local computer:

bash
ping -c 20 YOUR_SERVER_IP


On Windows, use:

powershell
ping YOUR_SERVER_IP -n 20


Record:

- Average latency
- Minimum and maximum latency
- Number of packets lost
- Whether the response time changes sharply between packets

A ping test is useful, but it is only the first step. Some networks deprioritize or block ICMP traffic, so a failed ping does not always mean that TCP traffic is unavailable.

### 2. Inspect the Route

Use `traceroute` or `tracert` to see how traffic reaches the server.

On Linux or macOS:

bash
traceroute YOUR_SERVER_IP


For a more useful view of latency and packet loss at each hop:

bash
mtr -rwzc 100 YOUR_SERVER_IP


On Windows:

powershell
tracert YOUR_SERVER_IP


If `mtr` is not installed, Debian and Ubuntu users can add it with:

bash
apt update && apt install mtr-tiny


A route test can reveal:

- Long detours
- A congested upstream hop
- High packet loss beginning at a particular network
- Large latency increases between regions
- Whether the return path is different from the outbound path

Do not judge a route from one test alone. Routers may limit ICMP responses without dropping normal application traffic. Look for loss that continues through later hops and reaches the final destination.

### 3. Run a General VPS Benchmark

A common all-in-one benchmark checks CPU, memory, disk, and network performance:

bash
curl -Lso- bench.sh | bash


Only run scripts from sources you trust and inspect the script before executing it on a production server. For a new test VPS, this kind of benchmark can provide a quick overview, but it should not be treated as a definitive measurement.

The output normally includes:

- Operating system
- CPU model and number of virtual cores
- Memory size
- Disk read and write speed
- Network results to several locations
- Basic system information

Run the benchmark at least twice. If the numbers vary widely, investigate whether the difference comes from network congestion, noisy neighbors, disk activity, or the selected test server.

### 4. Test Network Throughput With `iperf3`

`iperf3` is more useful than a browser-based speed test when you want to measure server-to-server throughput.

Install it on the VPS:

bash
apt update && apt install iperf3


You need an `iperf3` server that you control or a public endpoint that explicitly allows testing. Then run:

bash
iperf3 -c TEST_SERVER -P 4


The `-P 4` option opens four parallel streams. A single TCP stream may fail to fill a high-capacity connection because of distance, congestion, or TCP window behavior.

For the reverse direction:

bash
iperf3 -c TEST_SERVER -P 4 -R


For longer testing:

bash
iperf3 -c TEST_SERVER -P 4 -t 30


Do not run an aggressive bandwidth test on a production server during busy hours without checking your transfer allowance. High-volume tests consume network traffic and may affect other services.

### 5. Test Disk I/O With `fio`

For a new, empty VPS, `fio` can show whether storage performance is likely to become a bottleneck.

A sequential write test might look like this:

bash
fio --name=seq-write \
    --filename=/tmp/fio-test \
    --size=1G \
    --bs=1M \
    --rw=write \
    --iodepth=16 \
    --direct=1 \
    --runtime=60 \
    --time_based


A random read test:

bash
fio --name=random-read \
    --filename=/tmp/fio-test \
    --size=1G \
    --bs=4k \
    --rw=randread \
    --iodepth=32 \
    --direct=1 \
    --runtime=60 \
    --time_based


Delete the test file afterward:

bash
rm -f /tmp/fio-test


Do not compare sequential MB/s directly with random IOPS. They measure different workloads. A database may care more about random I/O latency than headline sequential read speed, while backups and large media files may benefit more from sequential throughput.

## How to Interpret the Results

A useful test report should include the test location and time. For example:

> Tested from Seattle over residential fiber at 8:00 PM Pacific Time. Average latency was 28 ms, packet loss was 0%, and throughput reached 1.4 Gbps using four TCP streams.

That report is much more meaningful than:

> The server is fast.

### Compare Latency by User Location

If your users are in California, test from California. If your application serves several regions, test from each major region separately.

For a global website, create a small table like this:

| Test location | Average latency | Packet loss | Download | Upload |
| --- | ---: | ---: | ---: | ---: |
| US West Coast |  |  |  |  |
| US East Coast |  |  |  |  |
| Europe |  |  |  |  |
| East Asia |  |  |  |  |
| Southeast Asia |  |  |  |  |

This prevents one favorable result from hiding a poor route elsewhere.

### Test During Peak and Off-Peak Hours

Network quality can change with congestion. Run one test during a quieter period and another when your users are most active.

For each run, record:

- Exact date and time
- Test origin
- ISP or carrier
- Protocol used
- Number of parallel streams
- Test server
- Average and maximum latency
- Packet loss
- Throughput

Do not turn one speed test into a permanent verdict. A VPS is a shared service, and Internet routes change.

### Separate Port Speed From Real Transfer Speed

BandwagonHost advertises uplink speeds ranging from 1 Gbps to 10 Gbps across its VPS offerings. That describes the network port or uplink capability, not a promise that one client will always receive that throughput to every destination.

A 2.5 Gbps port may still produce 300 Mbps to a distant test server because of:

- The remote server’s capacity
- TCP congestion control
- The number of streams
- Cross-border routing
- ISP peering
- Time of day
- Network distance

For that reason, use multiple test endpoints and repeat the test with both one and several TCP streams.

## BandwagonHost USCA_9 Plans and Current Prices

The AFF link supplied for this article resolves to BandwagonHost’s Los Angeles USCA_9 E-Commerce ordering flow. The table below covers the current CN2 GIA E-Commerce plans displayed in the official pricing data, including the larger high-bandwidth variants. Prices are shown in USD and can change when the billing period or product availability changes.

| Plan | Core configuration | Transfer | Link speed | Billing options | Purchase |
| --- | --- | ---: | ---: | --- | --- |
| Special 20G KVM Promo V5 | 20 GB RAID-10 SSD, 1 GB RAM, 2 Intel Xeon cores | 1 TB/mo | 2.5 Gbps | $49.99 quarterly; $89.99 semi-annually; $169.99 annually | [ Check 20G availability](https://bit.ly/BandwaGon) |
| Special 40G KVM Promo V5 | 40 GB RAID-10 SSD, 2 GB RAM, 3 Intel Xeon cores | 2 TB/mo | 2.5 Gbps | $89.99 quarterly; $169.99 semi-annually; $299.99 annually | [ Check 40G availability](https://bit.ly/BandwaGon) |
| Special 80G KVM Promo V5 | 80 GB RAID-10 SSD, 4 GB RAM, 4 Intel Xeon cores | 3 TB/mo | 2.5 Gbps | $56.99 monthly; $149.99 quarterly; $289.99 semi-annually; $549.99 annually | [ Check 80G availability](https://bit.ly/BandwaGon) |
| Special 160G KVM Promo V5 | 160 GB RAID-10 SSD, 8 GB RAM, 6 Intel Xeon cores | 5 TB/mo | 5 Gbps | $86.99 monthly; $239.99 quarterly; $459.99 semi-annually; $879.99 annually | [ Check 160G availability](https://bit.ly/BandwaGon) |
| Special 320G KVM Promo V5 | 320 GB RAID-10 SSD, 16 GB RAM, 8 Intel Xeon cores | 8 TB/mo | 5 Gbps | $159.99 monthly; $459.99 quarterly; $869.99 semi-annually; $1,599.99 annually | [ Check 320G availability](https://bit.ly/BandwaGon) |
| Special 640G KVM Promo V5 | 640 GB RAID-10 SSD, 32 GB RAM, 10 Intel Xeon cores | 10 TB/mo | 10 Gbps | $289.99 monthly; $799.99 quarterly; $1,499.99 semi-annually; $2,759.99 annually | [ Check 640G availability](https://bit.ly/BandwaGon) |
| Special 1280G KVM Promo V5 | 1,280 GB RAID-10 SSD, 64 GB RAM, 12 Intel Xeon cores | 12 TB/mo | 10 Gbps | $549.99 monthly; $1,559.99 quarterly; $2,979.99 semi-annually; $5,499.99 annually | [ Check 1280G availability](https://bit.ly/BandwaGon) |
| Special 1280G HIBW 15T | 1,280 GB local NVMe RAID-10, 64 GB ECC RAM, 12 AMD dedicated cores | 15 TB/mo | 10 Gbps | $879.99 monthly; $2,509.99 quarterly; $4,768.99 semi-annually; $8,799.99 annually | [ Check HIBW 15T availability](https://bit.ly/BandwaGon) |
| Special 1280G HIBW 20T | 1,280 GB local NVMe RAID-10, 64 GB ECC RAM, 12 AMD dedicated cores | 20 TB/mo | 10 Gbps | $1,159.99 monthly; $3,299.99 quarterly; $6,269.99 semi-annually; $11,598.99 annually | [ Check HIBW 20T availability](https://bit.ly/BandwaGon) |

The official cart also lists other BandwagonHost products, including Basic VPS, E-Commerce+SLA, Dubai plans, and additional locations. Those are separate product families and should not be treated as identical to the USCA_9 CN2 GIA plans above. The E-Commerce plans shown here include automatic migration options, backups, snapshots, KVM/KiwiVM management, root access, and a 99.95% uptime guarantee according to the product details.

## Which Plan Is Practical for a Speed Test?

You do not need a large VPS to measure network latency. A small plan is enough for:

- Ping and route tests
- `mtr`
- `iperf3`
- Basic HTTP benchmarks
- Lightweight monitoring
- A small personal website

The **20G or 40G plan** can work for a temporary test node if your application has low traffic and does not need much memory. The 1 GB RAM limit on the 20G plan leaves little room for a modern control panel, database, caching service, and web application running together.

The **80G plan** is a more comfortable starting point for a small website, reverse proxy, VPN gateway, or test environment. It provides 4 GB RAM, 4 CPU cores, 3 TB monthly transfer, and a 2.5 Gbps link.

The **160G plan** is easier to justify when you need more storage, a larger transfer allowance, or several services on one VPS. Its 8 GB RAM and 5 Gbps link leave more room for application caching and concurrent connections.

The **320G and 640G plans** are aimed at substantially heavier workloads. They make sense for media delivery, large backups, busy web applications, data processing, or multiple virtual services. Buying one solely to run a speed test would be excessive.

The HIBW plans are even more specialized. The 15 TB and 20 TB transfer options are relevant when the workload genuinely moves large volumes of data. They are not economical choices for a basic WordPress site or occasional development server.

## BandwagonHost Speed Test: Common Mistakes

### Mistake 1: Testing Only From the VPS

A download test from the VPS to a nearby server measures the VPS’s outbound path. It does not tell you how users reach the VPS.

Run tests from both directions:

text
Your users -> VPS
VPS -> test endpoint


The two results can differ because Internet routing is often asymmetric.

### Mistake 2: Treating One Speed Test as Proof

A single result may be affected by temporary congestion, server load, or the selected test endpoint. Repeat the test at different times and from multiple networks.

### Mistake 3: Confusing Bandwidth With Page Speed

A website’s page speed also depends on:

- Time to first byte
- PHP or application execution
- Database queries
- Cache configuration
- Image size
- DNS resolution
- TLS negotiation
- Browser rendering

A fast VPS network does not automatically make a poorly optimized application fast.

### Mistake 4: Running Destructive Tests on a Live Server

Disk benchmarks write test data. Network tests consume transfer. CPU benchmarks can temporarily increase load.

Run tests on a new VPS or during a maintenance window. Keep the test duration short enough to avoid interfering with production traffic.

### Mistake 5: Ignoring the Route to the Actual Audience

A Los Angeles server may be a strong option for US West Coast users and certain Asia-Pacific routes, but the only reliable way to assess your use case is to test from the networks your users actually use.

BandwagonHost’s own CN2 GIA documentation emphasizes that USCA_9 is designed around China Telecom CN2 GIA, China Mobile CMIN2, and China Unicom Premium connectivity. That makes route selection relevant for China-facing workloads, but it does not guarantee the same result for every carrier, city, or application.

## A Simple Test Report Template

Use this format when comparing BandwagonHost with another VPS provider:

text
Provider:
Plan:
Datacenter:
Server IP:
Test date and time:
Test origin:
ISP or carrier:

Ping:
- Average:
- Minimum:
- Maximum:
- Packet loss:

MTR:
- Average final latency:
- Packet loss at final hop:
- Major routing changes:

iperf3:
- One stream:
- Four streams:
- Reverse direction:

Disk:
- Sequential read:
- Sequential write:
- Random read:
- Random write:

Notes:


This makes the comparison repeatable. More importantly, it stops you from comparing a peak-hour test on one provider with an off-peak test on another.

## Final Verdict

A BandwagonHost speed test should focus on **latency, stability, route quality, and workload performance**, not just the largest Mbps number on a benchmark page.

For a small test server, the 20G, 40G, or 80G plans provide enough capacity to evaluate network behavior. The 160G plan is a more practical middle ground when you also need room for a real application. Larger 320G, 640G, and 1280G plans are justified by storage, transfer, CPU, or memory requirements rather than by speed testing alone.

The most useful test is the one that matches your actual users. Run `ping`, `mtr`, `iperf3`, and `fio` from the relevant regions, repeat the measurements during busy and quiet periods, and keep the results with their date, location, and test conditions.

For the current Los Angeles USCA_9 ordering flow, you can [👉 view the available BandwagonHost E-Commerce VPS plans](https://bit.ly/BandwaGon) and then repeat the same tests after deployment.
