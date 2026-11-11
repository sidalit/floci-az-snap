<div align="center">

<img src="snap/gui/floci-az.png" alt="Floci AZ logo" width="128" />

# Floci AZ

**Run Azure-compatible services locally for development, testing and CI.**

[![floci-az](https://snapcraft.io/floci-az/badge.svg)](https://snapcraft.io/floci-az)
[![Base: core26](https://img.shields.io/badge/Base-core26-2C8C3C?logo=canonical&logoColor=white)](https://snapcraft.io/core26)
![Confinement: strict](https://img.shields.io/badge/Confinement-strict-2C8C3C)
[![Maintainer: sidaliteir](https://img.shields.io/badge/Maintainer-sidaliteir-2C8C3C?logo=snapcraft&logoColor=white)](https://snapcraft.io/publisher/sidaliteir)
![Architectures: amd64, arm64](https://img.shields.io/badge/Architectures-amd64%20%7C%20arm64-2C8C3C?logo=linux&logoColor=white)

[Getting Started](#getting-started) · [Configure](#configure) · [Limitations](#limitations) · [Build](#build) · [Support](#support) · [License](#license)

Unofficial snap packaging for [Floci AZ](https://github.com/floci-io/floci-az), an Azure emulator with local storage and optional Docker-backed services.

</div>

---

## Getting Started

<a href="https://snapcraft.io/floci-az"><img alt="Get it from the Snap Store" src="https://snapcraft.io/en/dark/install.svg" width="180"></a>

```bash
sudo snap install floci-az
export AZURE_STORAGE_CONNECTION_STRING='DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=http://127.0.0.1:4577/devstoreaccount1;QueueEndpoint=http://127.0.0.1:4577/devstoreaccount1-queue;TableEndpoint=http://127.0.0.1:4577/devstoreaccount1-table;'
az storage container create --name my-container
```

The daemon starts automatically. Use your existing Azure CLI or SDK with the connection string above; no Azure account is needed and the key is the public development key. Blob, Queue and Table Storage, Cosmos DB, Key Vault and other in-process APIs work without Docker. Every service has its own path under `http://127.0.0.1:4577`; open that address in a browser for the dashboard, or see the [upstream service reference](https://floci.io/floci-az/services/).

For Functions, Event Hubs, database servers, Redis caches, other container-backed services and the web console, install Docker Engine separately and connect its socket:

```bash
sudo snap connect floci-az:docker
sudo snap restart floci-az.daemon
```

On systems using the Docker snap, use `sudo snap connect floci-az:docker docker:docker-daemon`. The connection grants privileged Docker API access. Containers reach Floci AZ through the Docker bridge, so the console at [http://127.0.0.1:4577/_floci/ui](http://127.0.0.1:4577/_floci/ui) and functions that use the emulated storage also need a listener outside loopback, as shown below.

## Configure

```bash
sudo snap set floci-az port=4578 storage-mode=persistent
sudo snap logs floci-az.daemon
```

| Option | Default | Purpose |
|---|---|---|
| `addr` | `127.0.0.1` | HTTP listener address |
| `port` | `4577` | Azure API port, 1–65535 |
| `storage-mode` | `persistent` | `memory`, `persistent`, `hybrid` or `wal` |
| `base-url` | Empty: `http://localhost:<port>` | Address advertised in API responses |
| `tls` | `false` | Also serve HTTPS, on `port` and on 443 |
| `allow-unsafe-network-exposure` | `false` | Required to listen outside loopback |

Changes restart an enabled daemon. For a trusted development network:

```bash
sudo snap set floci-az addr=0.0.0.0 allow-unsafe-network-exposure=true \
  base-url=http://192.168.1.10:4577
```

Terraform's `azurerm` provider and the Cosmos DB Java SDK need HTTPS:

```bash
sudo snap set floci-az tls=true allow-unsafe-network-exposure=true
curl -s http://127.0.0.1:4577/_floci/tls-cert -o floci-az.crt
```

With `tls=true`, upstream serves HTTP and HTTPS on `port` and on 443 on all interfaces, whatever `addr` is. `floci-az.crt` is the generated certificate authority for clients to trust.

State lives in `/var/snap/floci-az/common/data`. Containers started for Docker-backed services are removed, with the data they hold, whenever the daemon stops or restarts. Additional [upstream configuration](https://floci.io/floci-az/configuration/advanced/application-yml/) goes in `/var/snap/floci-az/common/config/application.properties`; restart the daemon after editing. The options above take precedence over that file.

Run an independent foreground instance with upstream environment variables and JVM properties, for example `FLOCI_AZ_PORT=4578 FLOCI_AZ_BASE_URL=http://localhost:4578 floci-az`. Its state uses `~/snap/floci-az/common/data`; upstream's foreground default is memory storage, so set `FLOCI_AZ_STORAGE_MODE=persistent` to retain it. For user-selected files on external storage, connect `floci-az:removable-media`.

## Limitations

- Floci AZ is a development emulator with unauthenticated APIs. Exposing it gives reachable clients control of emulated resources and, when connected, container-backed workloads.
- Docker-backed features and the console require Docker. Docker publishes their ports on all host interfaces, whatever `addr` is. The console uses host port 4500, as the `floci` snap's console does; set `floci-az.ui.port` in `application.properties` to run both.
- The Docker interface does not connect automatically. Connections on Ubuntu Core need the appropriate Snap Store declaration; ordinary Linux hosts can connect manually. No Docker daemon is bundled.
- TLS cannot be limited to loopback. Set `floci-az.tls.https-port=0` in `application.properties` to leave port 443 free.
- With upstream's default `floci-az.parity=compat`, resource groups are kept in memory only; `azure` and `strict` persist them.
- The Azure CLI and upstream's `azfloci` wrapper are not bundled. Host tools and hidden home configuration are not available.
- Refreshes preserve shared data, but reverting a snap does not roll back that data. Back up before upgrades; use `sudo snap refresh floci-az` for updates.

## Build

```bash
git clone https://github.com/sidalit/floci-az-snap.git
cd floci-az-snap
sudo snap install snapcraft --classic
snapcraft pack
```

## Support

- **Installation or snap issues:** [open a packaging issue](https://github.com/sidalit/floci-az-snap/issues).
- **Application bugs:** [report them upstream](https://github.com/floci-io/floci-az/issues).

## License

Snap packaging: [MIT](LICENSE).
