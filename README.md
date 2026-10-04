# Home-Network-Conf
Currently, I only have the NAS running OpenMediaVault. But I'll keep this for reference. 

Confs for hosts, ssh on my home network

Should probably fix NetBox, but ip range for now:

```
192.168.1.101  pve01 <--- Old

192.168.1.102 - 192.168.1.109 VPN Stuff

192.168.1.110-119  omv stuff

192.168.1.120-129 windows stuff

192.168.1.130-139 kub stuff
```

## Network
- Router: Ubiquiti EdgeRouter PoE 5-port (EdgeOS), gateway `192.168.1.1` on switch0. WAN `eth0` uses ISP DHCP (the IP rarely changes but isn't static, so DDNS handles it).
- Server: OMV, hostname `omv`, `192.168.1.110`, LAN `192.168.1.0/24`, nftables firewall.
- Docker is managed with Portainer.
- Domain `emiljo.com`, DNS on Cloudflare (registrar GoDaddy).

## Remote access VPN (WireGuard on OMV)
I use the OMV WireGuard plugin, not wg-easy. wg-easy failed with "address already in use" on UDP 51820 because the OMV tunnel was already running.

- Tunnel `wgnet1`, UDP `51820`, subnet `10.192.1.0/24`
- Clients (iPad, phone, laptop) are under Services > Wireguard > Clients, with "Restrict" on
- Endpoint: `vpn.emiljo.com:51820`
- Split tunnel: `AllowedIPs = 192.168.1.0/24, 10.192.1.0/24`

### DNS / DDNS
- A record `vpn.emiljo.com`, **DNS only** (grey cloud). The Cloudflare proxy doesn't carry UDP.
- Portainer stack `cloudflare-ddns`, see [cloudflare-ddns/docker-compose.yml](cloudflare-ddns/docker-compose.yml). The token is set in Portainer, never in the repo.
- Token: scoped to `emiljo.com` with Zone > DNS > Edit and Zone > Zone > Read

### Router / firewall
- EdgeRouter: forward UDP `51820` to `192.168.1.110`
- OMV firewall: allow UDP `51820` inbound

### While connected
| What      | Where                         |
|-----------|-------------------------------|
| OMV GUI   | http://192.168.1.110          |
| Portainer | https://192.168.1.110:<port>  |
| SSH       | `<user>@192.168.1.110`        |

- `omv.local` doesn't resolve over the tunnel (mDNS), so use IPs.
- If the remote network is also `192.168.1.0/24`, the routes clash. Use the server's tunnel IP (`ip -4 addr show wgnet1`) instead.

### Troubleshooting
```
sudo wg show              # check "latest handshake" for the peer
nslookup vpn.emiljo.com   # should return the current public IP
```
- Check the router forward and the OMV firewall.
- Test from a phone on mobile data, not on the LAN and not through ProtonVPN.

## Cloudflare Tunnel (Zero Trust)
Used only for web apps (HTTP), e.g. office-pong. It isn't used for the VPN because tunnels don't carry WireGuard UDP.

## Jellyfin for family/friends
`https://jellyfin.emiljo.com`: client → router TCP 443 → omv:8443 → caddy → jellyfin:8096

- DNS: CNAME `jellyfin` → `vpn.emiljo.com`, **DNS only**. Proxying video through Cloudflare can break their terms.
- Caddy: [caddy/](caddy/), Portainer git stack. The live Caddyfile is `/tank/appdata/caddy/Caddyfile` on omv; the repo copy is a reference, so copy edits there and restart caddy. Certs come via TLS-ALPN on 443 and renew automatically (`docker logs caddy`).
- Port 80 stays with the OMV GUI, so Caddy doesn't use it.
- Jellyfin and caddy share the `proxy` Docker network. It's external, so create it once before deploying either stack: `docker network create proxy`.
- EdgeRouter: forward TCP 443 → `192.168.1.110:8443`, hairpin NAT on.
- Jellyfin: Known Proxies = `caddy`, cap the remote bitrate, one account per person, admin has remote access off.

## Shoko (anime metadata for Jellyfin)
[shoko/](shoko/), Portainer git stack. Web UI: `http://192.168.1.110:8111`.

- Anime lives in `media/anime-serier` and `media/anime-filmer`. Shoko mounts `media` read-only at `/media`, the same path Jellyfin uses.
- First run wizard: log in to AniDB, then add both import folders, `/media/anime-serier` and `/media/anime-filmer`.
- Jellyfin: install the Shokofin plugin (add the repo `https://raw.githubusercontent.com/ShokoAnime/Shokofin/metadata/stable/manifest.json` under Plugins > Repositories) and set the host to `http://shoko-server:8111`. Then make a Shows library on `/media/anime-serier` and a Movies library on `/media/anime-filmer`.

## Never commit
Public IP, Cloudflare API token, WireGuard private or preshared keys, client `.conf` files.

## SMB Share

Windows 11 has some issues, this has fixed the problem:  
```
Use the local security policy approach:  (Group Policy Editor will pop up if you type that in the search box)
Use “Start->Run” and type in “gpedit.msc” in the “Run” dialog box. A “Group Policy” window will open.


Click down to “Local Computer Policy -> Computer Configuration -> Windows Settings -> Security Settings -> Local Policies -> Security Options.

     Scroll the list to find the policy “Network Security: LAN Manager authentication level”.
     Right click on this policy and choose “Properties”.
     Choose “Send NTLMv2 response only/refuse LM & NTLM”  (My original setting was: "Send LM & NTLM - use NTLMv2 session security if negotiated" )
     Click OK and confirm the setting change.
     Close the “Group Policy” window.

```
