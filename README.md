# Packet-Tracer-Project-7.1---Port-Security

![Project 7.1 Screenshot](https://github.com/pinoechalle-lang/Packet-Tracer-Project-7.1---Port-Security/blob/main/Screenshot%202026-11-02%20201615.png)

---

### A. Scenario

**Company:** Larkspur Logistics (fictional), a regional distribution company.

Last quarter, a visiting contractor plugged a personal laptop into an empty wall jack and joined the staff network for several days without anyone noticing. In the same week, an employee installed a small hub under their desk to share one wall jack with a coworker, and a second employee brought in an unmanaged switch to extend a desk. IT had no visibility into any of it.

The IT security officer issued this directive:

> "Every access port must only accept the devices we expect. If something unexpected shows up, I decide how loudly the switch reacts, from silently dropping traffic to shutting the port down. Ports nobody uses must be dead."

The network engineer (you) is asked to configure the branch access switches so that **only authorized devices can use each port, and each port reacts to unauthorized devices according to the risk of its location.**

---

### B. Why Port Security Matters

A switch builds its MAC address (CAM) table by reading the **source MAC address** of every frame it receives. By default, it learns source MAC addresses without restricting which devices may use a port.

| Problem | What happens |
|---|---|
| **Unauthorized device** | Someone can connect an unapproved device to an open port and potentially access the network. |
| **MAC flooding** | A device sends many frames with different fake source MAC addresses. This can fill the switch's MAC table and cause unknown-unicast traffic to be flooded. |
| **Unapproved hub or switch** | Additional devices may appear behind one port without IT's knowledge. |

Port security limits **which MAC addresses, and how many, are allowed on a port**, and defines how the switch responds when the limit is exceeded.

---

### C. Requirements

- Only approved devices may use each access port.
- The reaction to an unauthorized device must match the risk of the location.
- Approved devices must keep working after a reboot where the business requires that.
- Unused wall jacks and switch ports must not provide network access.
- A shared office port must support exactly two devices behind a hub.
- IT must be able to identify the port and MAC address associated with a violation.
- IT must be able to restore a locked-down port quickly after resolving the issue.
- Departments must be able to communicate with each other through R1.

### C.1. Security Policies for SWA and SWB

#### A. SWA

| Port | Device | Secure MAC type | Maximum | Violation action | Aging |
|---|---|---|---:|---|---|
| Fa0/1 | PC-A1 | **Static** | 1 | `shutdown` | None |
| Fa0/2 | PC-A2 | **Sticky** | 1 | `restrict` | None |
| Fa0/3 | Hub-A (PC-A3 and PC-A4) | **Sticky** | 2 | `protect` | None |
| Fa0/4 | PC-A5 | **Dynamic** | 1 | `restrict` | 2 minutes (absolute) |
| Fa0/10 | SWB | Trunk; no port security | n/a | n/a | n/a |
| G0/1 | R1 | Trunk; no port security | n/a | n/a | n/a |
| Fa0/5-9, Fa0/11-24, G0/2 | Unused | n/a | n/a | n/a | Assigned to VLAN 672 and shut down |

#### B. SWB

| Port | Device | Secure MAC type | Maximum | Violation action | Aging |
|---|---|---|---:|---|---|
| Fa0/1 | PC-B1 | **One static + one sticky** | 2 | `shutdown` | None |
| Fa0/2 | PC-B2 | **Sticky** | 1 | `shutdown` | None |
| Fa0/10 | SWA | Trunk; no port security | n/a | n/a | n/a |
| Fa0/3-9, Fa0/11-24, G0/1-2 | Unused | n/a | n/a | n/a | Assigned to VLAN 674 and shut down |

#### Why Each Port Uses Its Policy

| Switch | Port | Reason |
|---|---|---|
| SWA | Fa0/1 | Management traffic is limited to one known device; an unexpected MAC shuts down the port. |
| SWA | Fa0/2 | The switch learns the approved device's MAC address. A violation is recorded and traffic is dropped without shutting down the port. |
| SWA | Fa0/3 | Two devices share a hub, so the limit is two MAC addresses. An additional MAC is dropped by the `protect` action. |
| SWA | Fa0/4 | Demonstrates dynamic secure-MAC learning and absolute aging. |
| SWA | Fa0/10 | Trunk to SWB carrying traffic for multiple devices and VLANs. Port security is intentionally omitted. |
| SWA | G0/1 | Trunk to R1 carrying traffic for multiple VLANs. Port security is intentionally omitted. |
| SWA | Unused ports | Shut down and assigned to VLAN 672, a parking VLAN that is not used for normal network access. |
| SWB | Fa0/1 | Demonstrates one manually configured secure MAC address plus one sticky-learned address. |
| SWB | Fa0/2 | Sticky learning is used, but a violation shuts down the port. |
| SWB | Fa0/10 | Trunk to SWA carrying traffic for multiple devices and VLANs. Port security is intentionally omitted. |
| SWB | Unused ports | Shut down and assigned to VLAN 674, a parking VLAN that is not used for normal network access. |

> **Why port security is omitted on trunks:** A trunk carries source MAC addresses for many devices across multiple VLANs. A secure-MAC limit may be exceeded during normal operation; a violation could disrupt the trunk and affect multiple VLANs. This project secures trunks by explicitly configuring trunk mode, allowed VLANs, and the native VLAN consistently at both ends.

> **Important—MAC learning is traffic-triggered:** Connecting a device to a port does not, by itself, cause the switch to learn its MAC address as a secure address. For dynamic and sticky learning, the device must send a frame through the port. Until it sends traffic, its MAC may not appear in `show port-security address`, and a sticky address may not yet be present in the running configuration. Generate traffic from the device (for example, ping its default gateway), then check the secure-address list. Static secure MAC addresses are configured manually and do not depend on the device sending a frame.

---

### D. VLANs, Subnets, and Router Configuration

R1 provides inter-VLAN routing using router-on-a-stick. Each user VLAN has a /24 subnet, and the router subinterface address is the default gateway for that VLAN.

| VLAN | Department | Subnet | R1 default gateway |
|---:|---|---|---|
| 10 | MGR | `192.168.10.0/24` | `192.168.10.254` |
| 20 | LKM | `192.168.20.0/24` | `192.168.20.254` |
| 30 | Office | `192.168.30.0/24` | `192.168.30.254` |
| 40 | Recording | `192.168.40.0/24` | `192.168.40.254` |
| 50 | NVD | `192.168.50.0/24` | `192.168.50.254` |
| 60 | HTB | `192.168.60.0/24` | `192.168.60.254` |
| 190 | Native VLAN | No user subnet configured | None |

The host IP addresses should be in the subnet for their VLAN, use mask `255.255.255.0`, and use the listed R1 address as the default gateway. VLAN 190 is the native VLAN on the trunks; the supplied router configuration does not assign it a user subnet or gateway.

Connect R1 `G0/0` to SWA `G0/1`. The SWA-to-R1 trunk must allow VLANs 10, 20, 30, 40, 50, 60, and 190. 

---

### E. Trunk Policies in SWA and SWB

| Switch | Trunk port | Native VLAN | Allowed VLANs | Connected to |
|---|---|---|---|---|
| SWA | Fa0/10 | VLAN 190 (untagged) | 50, 60, 190 | SWB |
| SWA | G0/1 | VLAN 190 (untagged) | 10, 20, 30, 40, 50, 60, 190 | R1 |
| SWB | Fa0/10 | VLAN 190 (untagged) | 50, 60, 190 | SWA |

> **Native VLAN note:** The native VLAN must match at both ends of each trunk because native-VLAN traffic is untagged. VLAN 190 is used on all trunks here for consistency; using the same native VLAN across separate trunk links is a design choice.

---

### F. Test and Validate the Configurations

Perform these checks in Packet Tracer after configuring the switches. Test violation actions only on access ports, never on trunk ports.

1. **Check the configuration:** Run `show port-security`, `show port-security address`, and `show interfaces trunk` on the switches.
2. **Generate traffic before checking learned secure MACs:** Connect each approved device and make it send traffic, such as a ping to its default gateway. Port security learns dynamic or sticky MAC addresses from source frames; plugging in a device without sending traffic is not enough. Then verify with `show port-security address` and `show mac address-table interface <interface>`.
3. **Test approved devices:** Confirm each device can reach its default gateway and has network access.
4. **Test violation behavior in the lab:** On an access port only, connect an unauthorized device and have it send traffic. Confirm the configured response: `protect` drops unauthorized traffic, `restrict` drops and records it, and `shutdown` disables the port.
5. **Test inter-VLAN connectivity:** Use `ping` between devices in different departments to confirm routing through R1.
6. **Check unused ports:** Use `show vlan brief` and `show interfaces status` to confirm unused ports are in the designated parking VLAN and shut down.

For `shutdown` tests, reconnect the approved device and follow the recovery steps in Section G.1. Save the switch configuration after sticky addresses have been learned if those addresses must persist after a reboot.

---

### G. Troubleshooting

| Command | What it shows | When to use it |
|---|---|---|
| `show port-security` | Secure ports, maximum and current MAC counts, violation counts, and actions | Overview of port-security status |
| `show port-security interface <interface>` | Port-security status and details, including the last source MAC and VLAN | Investigating a violation on one port |
| `show port-security address` | Secure MAC addresses, their types, ports, and remaining age where supported | Checking secure MAC entries |
| `show mac address-table` | The switch MAC address table, including non-secure entries | Finding where a MAC address was learned |
| `show mac address-table interface <interface>` | MAC addresses learned on a particular port | Checking the devices learned on one port |
| `show interfaces status` | Port state, VLAN, duplex, and speed | Finding ports that are `err-disabled` or administratively down |
| `show interfaces trunk` | Operational trunks, native VLANs, and allowed VLANs | Checking trunk configuration and status |
| `show vlan brief` | VLANs and their assigned ports | Checking access and parking VLAN assignments |
| `show running-config interface <interface>` | Configuration applied to one interface | Checking port settings and configured sticky/static MAC entries |

> **Violation detail:** On `show port-security interface <interface>`, check **Last Source Address:VLAN** for the source MAC and VLAN associated with the latest violation.

#### G.1. Re-enable an `err-disabled` port

A port using the `shutdown` violation action may become `err-disabled` after a violation. First remove the unauthorized device and check the cause; then re-enable the interface:

```text
show port-security interface fa0/1
show interfaces status

configure terminal
interface fa0/1
shutdown
no shutdown
end
```

Replace `fa0/1` with the affected interface. The shutdown/no-shutdown sequence resets the interface, but it does not fix the cause of the violation. Make sure the unauthorized device is removed before bringing the port back up.

#### G.2. Check MAC addresses on a port

Replace `fa0/1` with the interface you want to inspect. If the device has not sent any frames, its dynamically or sticky-learned MAC may not appear yet; generate traffic first.

```text
show mac address-table interface fa0/1
show port-security interface fa0/1
show port-security address
```

The MAC address table shows addresses learned on the port. The port-security commands show secure entries and violation details. Dynamically learned secure addresses may not appear in the running configuration.

#### G.3. Remove a secure MAC address

Packet Tracer does not support the `clear port-security` commands for this project. For a manually configured secure MAC, remove its exact entry from the interface configuration:

```text
configure terminal
interface fa0/1
no switchport port-security mac-address xxxx.xxxx.xxxx
end
```

For a sticky MAC that has been written to the configuration, remove the exact sticky entry if the Packet Tracer IOS image accepts the command:

```text
configure terminal
interface fa0/2
no switchport port-security mac-address sticky xxxx.xxxx.xxxx
end
```

Replace the example interface and MAC address with the actual values. Packet Tracer command support can vary by version. Verify changes with `show port-security address` and `show running-config interface <interface>`. A dynamically learned secure MAC on SWA Fa0/4 should age out after the configured two-minute absolute aging period; it can be learned again when the device sends traffic. Clearing the regular MAC address table is not the same as removing a secure MAC entry.

Save configuration changes, including sticky MAC entries that should persist after a reboot, with:

```text
copy running-config startup-config
```

---

### H. Known Limitations

- Packet Tracer cannot generate a real MAC flooding attack.
- Port-security features in Packet Tracer are limited and may differ from real switch hardware.
- Violation counters and syslog details may differ from real hardware.
- Voice VLANs and per-VLAN maximums are not covered.
