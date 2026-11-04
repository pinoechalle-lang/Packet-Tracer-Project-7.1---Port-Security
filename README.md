# Packet-Tracer-Project-7.1---Port-Security

![Project 7.1 Screenshot](https://github.com/pinoechalle-lang/Packet-Tracer-Project-7.1---Port-Security/blob/main/Screenshot%202026-11-02%20201615.png)

---

### Scenario

**Company:** Larkspur Logistics (fictional), a regional distribution company.

Last quarter, a visiting contractor plugged a personal laptop into an empty wall jack and joined the staff network for several days without anyone noticing. In the same week, an employee installed a small hub under their desk to share one wall jack with a coworker, and a second employee brought in an unmanaged switch to extend a desk. IT had no visibility into any of it.

The IT security officer issued this directive:

> "Every access port must only accept the devices we expect. If something unexpected shows up, I decide how loudly the switch reacts, from silently dropping traffic to shutting the port down. Ports nobody uses must be dead."

The network engineer (you) is asked to configure the branch access switches so that **only authorized devices can use each port, and each port reacts to unauthorized devices according to the risk of its location.**

---

### Why Port Security Matters

A switch builds its MAC address (CAM) table by reading the **source MAC address** of every frame it receives. By default, it trusts whatever it sees.

| Problem | What happens |
|---|---|
| **Unauthorized device** | Anyone can plug into an open port and get network access |
| **MAC flooding** | An attacker sends thousands of frames with fake source MACs. The CAM table fills up and the switch floods frames out of every port, so the attacker can capture other users' traffic |
| **Rogue switch or hub** | Extra devices appear behind a single port without IT's knowledge |

Port Security limits **which** MAC addresses, and **how many**, are allowed on a port, and defines what happens when the limit is broken.

---

### Requirements

- Only approved devices may use each access port. 
- The reaction to an unauthorized device must match the risk of the location.
- Approved devices must keep working after a reboot, where the business needs that. 
- Unused wall jacks and switch ports must not provide network access. 
- A shared office port must support exactly two devices behind a hub. 
- IT must be able to see which port was violated and by which MAC. 
- IT must be able to restore a locked-down port quickly once the problem is resolved. 
-Departments must still be able to communicate with each other (through R1). 
