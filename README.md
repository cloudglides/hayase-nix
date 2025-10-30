This project is no longer maintained because the upstream repository was taken down due to a DMCA notice.
As a result, I won’t be updating or supporting this repo anymore.

# Hayase Nix Flake

This repository packages the upstream [Hayase](https://hayase.watch) AppImage for use with Nix. The flake exports a `hayase` package and an overlay that you can import into other flakes.

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

### Ad-hoc use

Build or run the packaged AppImage directly from the command line:

```bash
nix build github:cloudglides/hayase-nix#default
nix run github:cloudglides/hayase-nix#default
```

## Automation

The workflow in `.github/workflows/update.yml` listens for a `repository_dispatch` event named `hayase_release`, re-fetches the latest upstream AppImage, updates `package.nix`, builds the package, and opens a pull request.

Builds are pushed to the `cloudglides` Cachix cache when the `CACHIX_AUTH_TOKEN` secret is present. Local developers can run `cachix use cloudglides` to read from the cache or `cachix watch-store cloudglides` to upload local builds.

## Development

```bash
nix fmt
nix build .#default
```

Commit the generated `flake.lock` when inputs change to keep builds reproducible.
