---
title: Making My Raspberry Pi Public (The Curious Engineer Way)
date: 2026-04-18 18:01:22
tags:
  - raspberry-pi
---

## The Mental Breakdown
I wanted to deploy [bridgellm](https://www.bridgellm.tech/) to my raspberry pi just to see how the infra holds up, setup friction, latency, the usual. Seemed like a fun weekend thing.

I pulled out my pi, booted it up, and from my mac ran `ssh starvader@192.xx.xx.xx`. ***Boom, connected***. Well not really, it failed with `no route to host`. I thought the problem was pretty simple, maybe ssh just wasn't enabled on the pi. Walked over, checked. SSH is on, pub key is there on the pi, tried password auth too. Still nothing.

After a 30 min debugging session and a mental breakdown, i realised **my wifi router is blocking all incoming traffic to the pi** and i don't have admin access to set up port forwarding on it.

So the problem statement changed from deploying the server to successfully SSHing in. Now there are easy workarounds. [tailscale](https://tailscale.com) or [connect.raspberrypi.com](https://connect.raspberrypi.com) will get you there in five minutes and i'd genuinely recommend either if you're in the same spot. But my wannabe engineer ass said let's do it with our own infra. Full control, every bit configured by me, no third party sitting in between, and as long as my vm is up, so is the tunnel.


## The Irony
The best way to ssh into my pi was to use a vm that's publicly accessible, and have the pi create a reverse tunnel to it. So i'm using a cloud vm to get into a machine sitting on the same network, because i want to be able to reach it from anywhere in the world. Anyway, too much jargon? Here's what we're actually building.
```
Machine ── SSH ──> GCP VM (public)
                      ↑
               reverse tunnel
                      │
               Raspberry Pi (home)
```
The irony is that i started this whole thing to reduce my dependency on cloud, and now i need a cloud vm just to access the solution.

## The Bridge
I started with spinning up a vm on gcp, kept the config basic — `e2-micro`, `1 vCPU`, `512mb ram`, `10gb disk`, `ubuntu minimal`.

Now both my mac and the pi need to be able to connect to this vm. If it was just my machine, i'd just use `gcloud`:
```bash
# your machine
gcloud compute ssh <vm-username> --zone=<vm-instance-zone>
```
But since the pi also needs to connect, i wanted ssh keys managed directly on the vm rather than through gcp's metadata. To do that, update the vm's metadata with:
```
enable-oslogin: FALSE
```
This tells gcp to stop managing ssh access through its own system, so your `authorized_keys` file takes over. Now add your public ssh keys to `~/.ssh/authorized_keys` on the vm. Any device with the matching private key can now connect.

Test it:
```bash
# your machine
ssh -i <path-to-your-ssh-private-key> -o IdentitiesOnly=yes <vm-username>@<vm-external-ip>
```

## The Tunnel
Before running anything on the pi, fix the ssh config on the vm first. **By default `GatewayPorts` is off**, which means the tunnel port only binds to `127.0.0.1` and you can't hit it from outside the vm. Save yourself the back and forth and just set it up front:
```bash
# vm
sudo vi /etc/ssh/sshd_config
```
Uncomment or add these if not already there:
```
GatewayPorts yes
AllowTcpForwarding yes
```
Restart ssh (or just reboot the thing).

Now on the pi, run:
```bash
# raspberry pi
ssh -i <path-to-your-ssh-private-key> -N -R 2222:localhost:22 -o IdentitiesOnly=yes <vm-username>@<vm-external-ip>
```
This creates a reverse tunnel from the pi to the vm on port 2222, forwarding to the pi's port 22.

Verify on the vm:
```bash
# vm
ss -tlnp | grep 2222
```
Expected: `0.0.0.0:2222`. If you see `127.0.0.1:2222` the `GatewayPorts` change didn't apply, double check the sshd restart.

Once this is done, gcp's firewall will likely still block the traffic.

## The Firewall
We could just allow all IPs but let's do it properly. Create a firewall rule in gcp with:
```
Direction: Ingress
Protocol: TCP
Port: 2222
Source: <your-machine-ip>/32 (45.xxx.xxx.xxx/32)
Target: gcp-bridge (network tag)
```
The target is a network tag — you set it in the firewall rule and then add the same tag to your vm under network tags. This is how GCP knows which vm the rule applies to. Both sides need to match. GCP applies firewall rules instantly so no reboot needed.

Finally, from your machine:
```bash
# your machine
ssh -p 2222 <vm-username>@<vm-external-ip>
```
Enter your raspberry pi password. Boom, you're in.

## The Automation: Optional
The plain ssh tunnel works but has two problems — it times out if the connection drops, and if the pi reboots you have to manually start it again.

**Part 1: Keep the tunnel alive**

Install `autossh` on the pi and use it instead of plain ssh. The `-M 0` flag tells autossh to rely on SSH's own keepalive instead of a separate monitoring port:
```bash
# raspberry pi
autossh -M 0 -i <path-to-your-ssh-private-key> -N -R 2222:localhost:22 -o IdentitiesOnly=yes <vm-username>@<vm-external-ip>
```

**Part 2: Start the tunnel on reboot**

Create a systemd service on the pi:
```bash
# raspberry pi
sudo vi /etc/systemd/system/pi-tunnel.service
```
Paste this in:
```
[Unit]
Description=Reverse SSH Tunnel
After=network-online.target

[Service]
User=<your-pi-username>
ExecStart=/usr/bin/autossh -M 0 -i <path-to-your-ssh-private-key> -o IdentitiesOnly=yes -N -R 2222:localhost:22 <vm-username>@<vm-external-ip>
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```
Enable and start it:
```bash
# raspberry pi
sudo systemctl enable pi-tunnel
sudo systemctl start pi-tunnel
```
Now the tunnel comes up automatically on every reboot.

## The Happy Engineer
While it seems like i have ranted, i genuinely enjoyed every bit of this. Even the small learnings compound up sooner or later. ***Life is a sprint not a marathon***.
With that, thanks for reading. Drop me a text if you have any feedback.
