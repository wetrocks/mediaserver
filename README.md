# Home media server on Synology NAS
Docker stack with Wireguard, [Transmission](https://transmissionbt.com), and a reverse [nginx](http://nginx.org) proxy, with the addition of media server apps.

Running on Synology requires the Wireguard kernel module to be installed.

## Files

- docker-compose.yml
  - Brings up containers
  - `vpn` service will connect to Wireguard VPN, granting NET_ADMIN capability and using the wireguard kernel capability of the host
  - `transmission` service brings up Transmission listening on port 9091 and volume mounts a local directory for stateful configuration
    - Uses the `vpn` service networking to route all traffic over the VPN
  - `nginx` service proxies incoming requests to port 9091 on the container hosts interface and routs traffic to `transmission` service
    - required since port 9091 on the `transmission` service does not listen on the container hosts network interface since it's using the `openvpn` service network
  - `plex` service serves content to the LAN using host network
  - `*arr, jackett ,etc` handles content
- default.conf - Configuration for nginx
- vars.env - Example .env environment file for docker compose
- wg0.conf.example - Example wireguard config

## Configuration

- Environment
  - Copy vars.env to .env and set variables as appropriate.  See https://docs.linuxserver.io for details on PUID and PGID.
- VPN
  - Copy wg0.conf.example to ${CONFIG_BASE_PATH}/wireguard/wg0.conf.  Update to set the public key and address:port of the Wireguard server, and the private key and address for the wireguard client (vpn container).  Note the Synology-specific PostUp/PreDown scripts.
- Transmission
  - All transmission settings are stateful and are defined in the volume mount, which defaults to `/opt/transmission`. Put the `settings.json` file thereand it will be used everytime the service starts up. This allows for resuming and stateful operation.
- Nginx
  - Simple reverse proxy, but could include SSL and other configuration if needed.  default.conf can be left as-is

## Running

The `docker-compose.yml` file will bring up a container stack.  Each app can then be configured as needed.
