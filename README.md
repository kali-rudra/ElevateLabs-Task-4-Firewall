# ElevateLabs Cyber Security Internship – Task 4

## Setup and Use a Firewall on Linux

### 1. Objective

The objective of this task was to configure and test a basic Linux firewall using UFW (Uncomplicated Firewall).

The practical test focused on controlling inbound TCP traffic to port 23 (Telnet), verifying the difference between allowed and blocked connections, and then restoring the system to its original firewall state.

---

## 2. Lab Environment

| Component             | Details          |
| --------------------- | ---------------- |
| Operating System      | Kali Linux       |
| Virtualization        | VirtualBox       |
| Firewall              | UFW              |
| Test Target           | Metasploitable 2 |
| Kali Lab IP           | 192.168.56.103   |
| Metasploitable Lab IP | 192.168.56.102   |
| Test Port             | TCP 23           |
| Protocol              | TCP              |
| Network               | 192.168.56.0/24  |

The Kali Linux and Metasploitable systems communicated through the isolated VirtualBox lab network.

---

## 3. Tools Used

* UFW (Uncomplicated Firewall)
* Telnet
* Python 3 HTTP server
* `ss`
* Linux terminal

---

## 4. Initial Firewall State

UFW was initially installed and checked before the test.

```bash
sudo ufw status verbose
```

Initial result:

```text
Status: inactive
```

The default UFW policies were also examined:

```bash
grep '^DEFAULT_' /etc/default/ufw
```

The relevant policies were:

```text
DEFAULT_INPUT_POLICY="DROP"
DEFAULT_OUTPUT_POLICY="ACCEPT"
DEFAULT_FORWARD_POLICY="DROP"
DEFAULT_APPLICATION_POLICY="SKIP"
```

Although the default input policy was configured as `DROP`, UFW itself was initially inactive.

---

## 5. Test Service on Port 23

To create a controlled service for the firewall test, a temporary Python HTTP server was started on Kali and bound to the lab IP address on TCP port 23.

```bash
sudo python3 -m http.server 23 --bind 192.168.56.103
```

The listening socket was verified with:

```bash
sudo ss -lntp | grep ':23'
```

This confirmed that a service was listening on:

```text
192.168.56.103:23
```

Port 23 was selected because it is traditionally associated with Telnet and is within the Linux privileged-port range.

---

## 6. Connection Test Before Firewall Blocking

From the authorized Metasploitable 2 lab machine, a Telnet connection was attempted:

```bash
telnet 192.168.56.103
```

The connection succeeded:

```text
Trying 192.168.56.103...
Connected to 192.168.56.103.
```

This established the baseline that TCP port 23 was reachable before the firewall rule was activated.

---

## 7. Configure the Firewall Rule

A UFW rule was added to deny inbound TCP traffic to port 23:

```bash
sudo ufw deny 23/tcp
```

The added rule was checked with:

```bash
sudo ufw show added
```

The rule was then activated by enabling UFW:

```bash
sudo ufw enable
```

The active rules were verified:

```bash
sudo ufw status numbered
```

The resulting rule included:

```text
[ 1] 23/tcp    DENY IN    Anywhere
[ 2] 23/tcp (v6)    DENY IN    Anywhere (v6)
```

---

## 8. Connection Test After Firewall Blocking

The same Telnet test was repeated from Metasploitable 2:

```bash
telnet 192.168.56.103
```

This time the connection did not succeed and eventually returned:

```text
Trying 192.168.56.103...
telnet: Unable to connect to remote host: Connection timed out
```

This demonstrated that the firewall rule was preventing the inbound TCP connection to port 23.

### Result

| Test Condition       | Result               |
| -------------------- | -------------------- |
| Before firewall rule | Connection succeeded |
| After UFW deny rule  | Connection timed out |

The change in behaviour demonstrated the effect of the firewall rule on inbound traffic.

---

## 9. Cleanup and Restoration

After testing, the temporary firewall rules were removed.

The IPv4 and IPv6 deny rules were deleted using UFW.

UFW was then disabled:

```bash
sudo ufw disable
```

The final firewall state was verified:

```bash
sudo ufw status verbose
```

Final result:

```text
Status: inactive
```

The temporary Python server was also stopped using `Ctrl+C`.

The system was therefore returned to its original operational firewall state.

---

## 10. Evidence

The practical work was documented with screenshots covering the important stages of the exercise:

1. Baseline network and UFW state
2. Port 23 listening before firewall activation
3. Successful connection before blocking
4. UFW default policies
5. Port 23 deny rule configured
6. UFW active with port 23 denied
7. Connection timeout after firewall activation
8. Final UFW state after cleanup

---

## 11. How a Firewall Filters Traffic

A firewall controls network traffic according to defined rules.

In this exercise, the rule:

```text
23/tcp    DENY IN
```

instructed UFW to prevent inbound TCP traffic destined for port 23.

Before the rule was active, the connection from Metasploitable 2 reached the service and succeeded.

After the rule was active, the connection was prevented and eventually timed out.

This demonstrates the basic principle of firewall filtering: traffic is evaluated against firewall policy before it is permitted to reach the intended service.

---

## 12. Security Significance

Telnet traditionally uses TCP port 23 and is considered insecure for administrative access because its communication does not provide the protections expected from modern encrypted remote administration.

A firewall can reduce exposure by restricting unnecessary inbound services and ports.

In a real environment, firewall rules should be designed according to the organization's required services and security policy rather than simply blocking ports without considering operational requirements.

---

## 13. Conclusion

This task demonstrated the basic configuration and testing of a Linux host firewall using UFW.

A controlled TCP service was exposed on port 23, connectivity was verified, an inbound deny rule was applied, and the connection was tested again. The successful connection before the rule and the connection timeout after the rule provided practical evidence that the firewall was filtering the traffic.

The test configuration was subsequently removed and UFW was disabled to restore the original system state.
