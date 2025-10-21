# Hayase Nix Flake

Nix packaging for the [Hayase](https://hayase.watch) anime torrent streaming client. This flake wraps the upstream AppImage so you can install the desktop app declaratively and keep it updated automatically.

## Features

- **Single-command install** via `nix build` or `nix run`.
- **Overlay exposed** so `pkgs.hayase` becomes available when you import it into your own flake.
- **Automated updates** powered by GitHub Actions that track new `hayase-app/ui` releases and refresh the version/hash.
- **Cachix integration** so CI builds (and optionally your local builds) populate the `cloudglides` binary cache.

## Installation

Add the flake as an input in your configuration:

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    hayase = {
      url = "github:cloudglides/hayase-nix";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };

  outputs = { self, nixpkgs, hayase, ... }:
    let
      system = "x86_64-linux";
      pkgs = import nixpkgs {
        inherit system;
        overlays = [ hayase.overlays.default ];
      };
    in {
      packages.${system}.default = pkgs.hayase;
    };
}
```

### Home Manager / NixOS usage

Once the overlay is imported, reference `pkgs.hayase` anywhere you would normally list packages:

```nix
{
  home.packages = [ pkgs.hayase ];
  # or
  environment.systemPackages = [ pkgs.hayase ];
}
```

Alternatively, you can pin the exported package directly without the overlay:

```nix
home.packages = [ hayase.packages.${pkgs.system}.default ];
```

### Ad-hoc install

Build or run the packaged AppImage directly from the command line:

```bash
nix build github:cloudglides/hayase-nix#default
nix run github:cloudglides/hayase-nix#default
```

## Automation

- The workflow in `.github/workflows/update.yml` listens for a `repository_dispatch` event named `hayase_release`, re-fetches the latest upstream AppImage, updates `package.nix`, builds the package, and opens a pull request.
- To wire this up, configure the upstream `hayase-app/ui` repository to POST a `repository_dispatch` with `event_type: "hayase_release"` whenever a release is published.
- Builds are pushed to the `cloudglides` Cachix cache when the `CACHIX_AUTH_TOKEN` secret is present. Local developers can run `cachix use cloudglides` to consume the same cache; use `cachix watch-store cloudglides` if you want to upload local build results.

## Development

```bash
# format (no-op for now, but handy if you add Nix formatting rules)
nix fmt

# run the package build locally
nix build .#default
```

If you modify dependencies or Nixpkgs inputs, commit the generated `flake.lock` alongside your changes for reproducible builds.
