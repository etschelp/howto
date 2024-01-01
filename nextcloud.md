# Setup Nextcloud on Ubuntu

## Install via Snap

See also: [nextcloud-snap](https://github.com/nextcloud-snap/nextcloud-snap)

```sh
sudo apt update
sudo apt install snapd
sudo snap install nextcloud
```

```shell
snap start|stop nextcloud
```

### System Monitoring

```sh
sudo snap connect nextcloud:network-observe
```

### Nextcloud Settings

```sh
sudo snap set nextcloud ports.http=8080
sudo snap set nextcloud php.memory-limit=2048M
sudo snap set nextcloud http.compression=true
```

## Setup Proxy

### Install Caddy

```sh
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

### Cofigure Caddy

`/etc/caddy/Caddyfile`

```sh
domain.name.com {
    redir /.well-known/carddav /remote.php/dav 301
    redir /.well-known/caldav /remote.php/dav 301
    reverse_proxy :8080
    header {
        Strict-Transport-Security max-age=15552000;
    }
}
```

`sudo service caddy reload`

### Configure Nextcloud

`/var/snap/nextcloud/current/nextcloud/config/config.php`

```sh
  'trusted_proxies' => ['localhost'],
  'overwritehost' => '<dns_hostname>',
  'overwriteprotocol' => 'https',
  'default_phone_region' => 'CH'
```