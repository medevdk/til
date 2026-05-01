# Bypass blocked port 22 on Captive Portal with Cloudflare tunnel

By routing via Cloudflare, the ssh traffic is now "packed" inside a outbound
HTTPS connection. To the captive portal it looks like browsing a website on
port 443

## On the server:

###1. Install cloudflared (Cloudflare's official method):

```bash
sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-public-v2.gpg | sudo tee /usr/share/keyrings/cloudflare-public-v2.gpg >/dev/null
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-public-v2.gpg] https://pkg.cloudflare.com/cloudflared any main' | sudo tee /etc/apt/sources.list.d/cloudflared.list
sudo apt-get update && sudo apt-get install cloudflared
```

This will setup the apt repository so it is easy to update cloudflared with:

```bash
sudo apt update && sudo apt upgrade cloudflared
```

### 2. Login and copy the URL and open in the Mac browser

```bash
cloudflared tunnel login
```

### 3. Create tunnel

Tunnel name = 'faryland'

```bash
cloudflared tunnel create faryland
```

The output is the `TUNNEL_ID` which is an UUID needed at next step

### 4. Create DNS record

Domain name = 'unlomaps.com'

```bash
cloudflared tunnel route dns faryland ssh.unlomaps.com
```

Optional: test if the connection is live by (run on MAC):

```bash
dig ssh.unlomaps.com
```

It might take a few minutes for the DNS record to resolve, so if `dig` returns nothing immediately, just wait a bit and retry

A .json file is generated in the home folder (`~/.cloudflared`). Move it to a location the systemd service can reach (usually `/etc/cloudflared/`)

Move the credentials:

```bash
sudo mkdir -p /etc/cloudflared
sudo cp ~/.cloudflared/*.json /etc/cloudflared/
```

```bash
sudo nvim /etc/cloudflared/config.yml
```

⚠️ Replace both occurrences of <TUNNEL_ID> with the actual UUID from cloudflared tunnel list — do not paste the literal text <TUNNEL_ID>

Paste this:

```yaml
tunnel: <TUNNEL_ID>
credentials-file: /etc/cloudflared/<TUNNEL_ID>.json

ingress:
  - hostname: ssh.unlomaps.com
    service: ssh://localhost:22
  - service: http_status:404
```

The `<TUNNEL_ID>` was generated at at step 3 `cloudflared tunnel create faryland`
To retrieve:

```bash
cloudflared tunnel list
```

output:
| ID | Name | Created | Connections |
|:---|:-----|:--------|:------------|
|7b2a..f41a|faryland|2026-04-30..|0 (or 2 if running)|

### 5. Install the service:

```bash
sudo cloudflared service install
```

It will look for `/etc/cloudflared/config.yml`. If this files exists and is valid it will set up the systemd unit to use it. Then you do not need to manually run `cloudflared tunnel run faryland` once the service is active

### 6. Run the tunnel as a service

```
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
sudo systemctl status cloudflared
```

## On the Mac:

Install cloudflared

```bash
brew install cloudflared
```

If cloudflared is not found after brew install, run `which cloudflared` to check the path, then add this to .zshrc:

```
export PATH="/opt/homebrew/bin:$PATH"
```

Add to ~/.ssh/config:

```bash
Host unlo-ssh
    HostName ssh.unlomaps.com
    User devdk
    Proxycommand /opt/homebrew/bin/cloudflared access ssh --hostname %h
    # possible alternative:
    # ProxyCommand cloudflared access ssh --hostname ssh.unlomaps.com
    ServerAliveInterval 30
    ServerAliveCountMax 3
```

- ServerAliveInterval 30:
  The Mac will send a small, encrypted data packet to the server every 30 seconds if no other data has been sent. This tells the Starlink router and the Cloudflare Tunnel, "Hey, we are still talking, don't close this connection!"

- ServerAliveCountMax 3:
  This is the "strike" system. If the Mac sends a heartbeat and doesn't get a response, it will try up to 3 times. If it fails all three, it assumes the connection is truly dead and closes the session locally so your terminal doesn't just hang forever.

Then connect with:

```bash
ssh unlo-ssh
```

## Tip:

If the tunnel drops it can be the captive portal timing out the MAC address, not the tunnel failing. `curl 1.1.1.1` often triggers the re-auth page when tunnel stops responding

## Troubleshooting:

If `curl -v https://ssh.unlomaps.com` resets after the TLS Client Hello (but google.com works fine): The network is doing SNI-based filtering — it reads the hostname from the TLS handshake and blocks unknown domains even on port 443. Enable Cloudflare WARP to encrypt the SNI:
More info for [SNI filtering here](https://blog.compass-security.com/2025/03/bypassing-web-filters-part-1-sni-spoofing/)

```bash
brew install --cask cloudflare-warp
```

After installing WARP, the connection may still fail. Check with `warp-status`If it shows `warp=off`, WARP is only encrypting DNS and not tunneling traffic. The default mode is `DnsOverHttps`, which is not enough. Fix it with: `warp-cli mode warp && warp-cli connect`

To make life easier add as alias to .zshrc:

```bash
alias warp-on="warp-cli mode warp && warp-cli connect"
alias warp-off="warp-cli disconnect"
alias warp-status="curl -s https://www.cloudflare.com/cdn-cgi/trace | grep warp"
```

Run `warp-on` to set the correct mode and connect, then verify with `warp-status`

The workflow on SNI-filtering networks:

- Authenticate to captive portal in browser
- Enable WARP
- `warp-on`
- `warp-status` Confirm warp=on before ssh
- `ssh unlo-ssh`

Now the ssh traffic is moved to port 443 (HTTPS) and hiding the SNI with WARP.
Basically the traffic is now indistinguishable from just browsing a standard website.

If the tunnel drops mid-session:

- disable WARP (`warp-off`)
- `curl 1.1.1.1` to trigger portal re-auth
- re-authenticate portal login in browser
- `warp-on`
- `warp-status` # confirm warp=on
- `ssh unlo-ssh``
