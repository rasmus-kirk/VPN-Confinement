# VPN-Confinement
A NixOS module which lets you route traffic from systemd services through a VPN while preventing DNS leaks.

# Installation

## Nix Flake

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    vpnconfinement.url = "github:Maroka-chan/VPN-Confinement";
  };

  outputs = { self, nixpkgs, vpnconfinement, ... }:
  {
    # Change hostname, system, etc. as needed.
    nixosConfigurations.hostname = nixpkgs.lib.nixosSystem {
      system = "x86_64-linux";
      modules = [
        ./configuration.nix
        vpnconfinement.nixosModules.default
      ];
    };
  };
}

```

# Usage

## Define VPN network namespace

```nix
vpnnamespaces.<name> = {
  enable = true;
  wireguardConfigFile = <path to secret wireguard config file>;
  accessibleFrom = [
    "<ip or subnet>"
  ];
  portMappings = [{
      from = <port on host>;
      to = <port in VPN network namespace>;
      protocol = "<transport protocol>"; # protocol = "tcp"(default), "udp", or "both"
  }];
  openVPNPorts = [{
    port = <port to access through VPN interface>;
    protocol = "<transport protocol>"; # protocol = "tcp"(default), "udp", or "both"
  }];
};
```

## Add systemd service to VPN network namespace

```nix
systemd.services.<name>.vpnconfinement = {
  enable = true;
  vpnnamespace = "<network namespace name>";
};
```

## Example

```nix
# configuration.nix
{ pkgs, lib, config, ... }:
{
  # Define VPN network namespace
  vpnnamespaces.wg = {
    enable = true;
    wireguardConfigFile = /. + "/secrets/wg0.conf";
    accessibleFrom = [
      "192.168.0.0/24"
    ];
    portMappings = [
      { from = 9091; to = 9091; }
    ];
    openVPNPorts = [{
      port = 60729;
      protocol = "both";
    }];
  };

  # Add systemd service to VPN network namespace.
  systemd.services.transmission.vpnconfinement = {
    enable = true;
    vpnnamespace = "wg";
  };

  services.transmission = {
    enable = true;
    settings = {
      "rpc-bind-address" = "192.168.15.1"; # Bind RPC/WebUI to bridge address
    };
  };
}
```

See all options and their descriptions in the [module file](https://github.com/Maroka-chan/VPN-Confinement/blob/a62ed5b97b1556c8c1eb2bc38bf384caab7234fc/modules/vpnnetns.nix#L88).
