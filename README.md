# Default Router Gateway IP & Subnet Directory

An open-access technical directory indexing default IPv4 gateway addresses, private RFC 1918 subnet allocations, and factory administrative credentials across consumer and enterprise networking hardware.

## Standard Gateway Allocation Matrix

The table below catalogs primary gateway host assignments, subnet masks, and administrative ports across major manufacturers:

| Manufacturer | Hardware Line / Modem Series | Primary Gateway IP | Subnet Mask | Default User | Default Password | Port | Reference Guide |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Netgear** | Nighthawk / Orbi / DOCSIS | `192.168.1.1` | `255.255.255.0` | `admin` | `password` | 80 | [192.168.1.1 Protocol](https://routeripguide.com/ip/192-168-1-1) |
| **TP-Link** | Archer / AX / Deco Mesh | `192.168.0.1` | `255.255.255.0` | `admin` | `admin` | 80 | [192.168.0.1 Protocol](https://routeripguide.com/ip/192-168-0-1) |
| **Comcast Xfinity** | xFi Gateway XB6 / XB7 / XB8 | `10.0.0.1` | `255.255.255.0` | `admin` | `password` | 80 | [10.0.0.1 Guide](https://routeripguide.com/ip/10-0-0-1) |
| **AT&T Fiber** | BGW210-700 / BGW320-500 | `192.168.1.254` | `255.255.255.0` | `admin` | `(Access Code)` | 80 | [192.168.1.254 Guide](https://routeripguide.com/ip/192-168-1-254) |
| **Asus** | RT-AX / ROG Rapture / ZenWiFi | `192.168.50.1` | `255.255.255.0` | `admin` | `admin` | 80 | [192.168.1.1 Fallback](https://routeripguide.com/ip/192-168-1-1) |
| **Linksys** | Velop / Hydra / WRT Series | `192.168.1.1` | `255.255.255.0` | `admin` | `admin` | 80 | [192.168.1.1 Protocol](https://routeripguide.com/ip/192-168-1-1) |
| **D-Link** | DIR / Eagle Pro AI | `192.168.0.1` | `255.255.255.0` | `admin` | `(blank)` | 80 | [192.168.0.1 Protocol](https://routeripguide.com/ip/192-168-0-1) |
| **Ubiquiti** | UniFi Dream Machine / Gateway | `192.168.1.1` | `255.255.255.0` | `ubnt` | `ubnt` | 443 | [192.168.1.1 Protocol](https://routeripguide.com/ip/192-168-1-1) |
| **RockREMOTE / Omron** | Satellite IoT / CP1L-E PLC | `192.168.250.1` | `255.255.255.0` | `admin` | `(Static IP Required)` | 8080 / 80 | [192.168.250.1 Guide](https://routeripguide.com/ip/192-168-250-1) |
| **Lorex** | 4K PoE NVR / Wi-Fi Cameras | Dynamic DHCP (10.1.1.1 Internal) | `255.255.255.0` | `admin` | `(Initial Setup)` | 80 / 9000 / 554 | [Lorex Setup Guide](https://routeripguide.com/lorex-camera-setup) |

For complete hardware documentation across 30+ additional brands, consult the [router hardware directory](https://routeripguide.com/router).

---

## Subnet Architecture & Isolation Boundaries

Under RFC 1918, private local area networks allocate address space from three non-routable blocks:
* `10.0.0.0/8` (10.0.0.0 – 10.255.255.255)
* `172.16.0.0/12` (172.16.0.0 – 172.31.255.255)
* `192.168.0.0/16` (192.168.0.0 – 192.168.255.255)

Most consumer hardware partitions the `192.168.0.0/16` space into `/24` subnets (subnet mask `255.255.255.0`), yielding 254 usable host addresses:
* **Host Index 1 (.1):** Netgear, Asus, and Linksys bind their LAN interface to `192.168.1.1` or `192.168.0.1`.
* **High-Host Reservation (.254):** Gateways like the AT&T BGW series place the management interface at `192.168.1.254`, leaving lower host indices (`.2` through `.63`) unencumbered for static device reservations.

---

## Cascaded Router Subnet Collisions

When chaining a personal router behind an ISP-supplied modem/gateway combo:

If both the upstream modem and downstream router deploy `192.168.1.0/24`:
1. The secondary router's WAN interface receives an IP in `192.168.1.0/24` from the upstream DHCP pool.
2. The secondary router's LAN interface also binds to `192.168.1.1`.
3. The routing engine registers two active interface routes for `192.168.1.0/24`. Outbound traffic fails interface resolution because the kernel cannot determine whether packets belong to the WAN or LAN segment.

### Remediation Protocol:
1. Disconnect the WAN uplink cable between both units.
2. Connect a PC directly to a LAN port on the secondary router.
3. Access its administrative console and reassign its LAN subnet to an alternate pool (such as `192.168.2.1` or `10.50.1.1`).
4. Save the configuration, reboot both devices, and reconnect the WAN uplink.

---

## Troubleshooting Gateway Reachability (192.168.1.1 Timeouts)

When attempting to access `192.168.1.1` via a web browser, connections frequently fail with `ERR_CONNECTION_TIMED_OUT`, `ERR_CONNECTION_REFUSED`, or blank responses. Common root causes include:

1. **Subnet Disconnect:** The host adapter is on a different network segment (e.g., connected to a cellular hotspot or secondary guest AP).
2. **Incorrect Gateway IP:** The hardware's active default gateway is actually `192.168.0.1`, `10.0.0.1`, or `192.168.1.254`.
3. **HTTP vs HTTPS Port Mismatch:** The administrative daemon listens exclusively on TLS port 443 (requiring `https://192.168.1.1`) or redirects to an ISP mobile application.
4. **IP Conflict / Duplicate Default Gateway:** Two devices on the local LAN broadcast conflicting ARP announcements for `.1`.

For the complete diagnostic walk-through covering ping verification, ICMP echo checks, and step-by-step connection recovery on Windows, Mac, and mobile, refer to the [192.168.1.1 troubleshooting guide](https://routeripguide.com/troubleshooting/192-168-1-1-not-working).

---

## CLI Diagnostic Commands

### Windows Routing & Gateway Inspection
```cmd
:: Display network adapters, IP allocations, and active gateway
ipconfig /all

:: Display the active IPv4 routing table
route print 0.0.0.0

:: Inspect ARP cache for gateway MAC address binding
arp -a
```

### Linux & macOS Verification
```bash
# Query active default route interface and gateway
ip route show default

# macOS alternative
netstat -nr | grep default

# Inspect Layer-2 neighbor table for gateway resolution
ip neigh show
```

Published by [Router IP Guide](https://routeripguide.com/) - Independent Hardware Networking & Gateway Diagnostic Suite.

