<div align="center">

<img src="snap/gui/floci-az.png" alt="Floci AZ logo" width="128" />

# Floci AZ

**Run Azure-compatible services locally for development, testing and CI.**

[![floci-az](https://snapcraft.io/floci-az/badge.svg)](https://snapcraft.io/floci-az)
[![Base: core26](https://img.shields.io/badge/Base-core26-2C8C3C?logo=canonical&logoColor=white)](https://snapcraft.io/core26)
![Confinement: strict](https://img.shields.io/badge/Confinement-strict-2C8C3C)
[![Maintainer: sidaliteir](https://img.shields.io/badge/Maintainer-sidaliteir-2C8C3C?logo=snapcraft&logoColor=white)](https://snapcraft.io/publisher/sidaliteir)
![Architectures: amd64 and arm64](https://img.shields.io/badge/Architectures-amd64%20%7C%20arm64-2C8C3C?logo=linux&logoColor=white)

[Getting Started](#getting-started) · [Configure](#configure) · [Limitations](#limitations) · [Build](#build) · [Support](#support) · [License](#license)

Unofficial snap packaging for [Floci AZ](https://github.com/floci-io/floci-az), an Azure emulator with local storage and optional Docker-backed services.

</div>

---

## Getting Started

<a href="https://snapcraft.io/floci-az"><img alt="Get it from the Snap Store" src="https://snapcraft.io/en/dark/install.svg" width="180"></a>

```bash
sudo snap install floci-az
```

The emulator starts automatically on `http://127.0.0.1:4577`. Point your Azure CLI or SDK at it as described in the [upstream documentation](https://floci.io/floci-az/).

Docker-backed services and the web console need Docker Engine, installed separately:

```bash
sudo snap connect floci-az:docker
sudo snap restart floci-az.daemon
```

## Configure

| Option | Default | Purpose |
|---|---|---|
| `addr` | `127.0.0.1` | Listen address |
| `port` | `4577` | Azure API port |
| `storage-mode` | `persistent` | `memory`, `persistent`, `hybrid` or `wal` |
| `base-url` | `http://localhost:<port>` | Address advertised in API responses |
| `tls` | `false` | Also serve HTTPS on `port` and 443 |
| `allow-unsafe-network-exposure` | `false` | Required to listen outside loopback or with `tls` |

```bash
sudo snap set floci-az port=4578
```

State lives in `/var/snap/floci-az/common/data`. Other [upstream settings](https://floci.io/floci-az/configuration/advanced/application-yml/) go in `/var/snap/floci-az/common/config/application.properties`; restart the daemon after editing.

## Limitations

- The emulator has no authentication: keep it on loopback or a trusted network.
- The `docker` interface is not connected automatically, and Docker publishes container ports on all interfaces.
- `snap revert` does not roll back the emulator's data.

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

Snap packaging: [MIT](https://github.com/sidalit/floci-az-snap/blob/main/LICENSE).
