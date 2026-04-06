---
date: '2026-02-13T10:59:19+01:00'
draft: false
title: 'Obtaining Hister'
---

This page documents how to obtain the Hister program, which serves both as a command-line/TUI client and as the server; setting up the browser extensions is covered [later](quickstart#installing-a-browser-extension).

If you are using a server already set up by someone else, and you aren't planning on using any of the client's features, then _you do not need to download this program_.

## Pre-built Binary

1. Go to the [Releases page](https://github.com/asciimoo/hister/releases) (or the [Rolling Release](https://github.com/asciimoo/hister/releases/tag/rolling) for the latest development version).

   In the **Assets** section, download the file matching your platform and architecture:

   | Platform | File to download |
   |----------|-----------------|
   | macOS, Apple Silicon (M1/M2/M3) | `hister_darwin_arm64` |
   | macOS, Intel | `hister_darwin_amd64` |
   | Linux, 64-bit | `hister_linux_amd64` |
   | Linux, ARM64 | `hister_linux_arm64` |
   | Windows, 64-bit | `hister_windows_amd64.exe` |

   > **Careful:** GitHub also shows "Source code" archives (`hister-x.y.z.tar.gz`, `Source code (zip)`) at the bottom of each release's Assets section. Those contain the source code and need to be compiled — they are _not_ what you want here.

2. Make the binary executable:

   ```bash
   chmod +x hister
   ```

3. **macOS only** — Remove the quarantine flag that macOS applies to downloaded files. Without this step, macOS will refuse to run the binary with a "cannot be opened because the developer cannot be verified" error:

   ```bash
   xattr -d com.apple.quarantine hister
   ```

   Alternatively, after attempting to run `hister` once, go to **System Settings → Privacy & Security** and click **Allow Anyway**.

4. Optionally, move it to somewhere on your `PATH`; for example, `/usr/local/bin/` (system-wide) or `~/.local/bin/` (per-user).

## Building from Source

[Download a snapshot] of, or [clone] the source code (from [GitHub] or [Codeberg]).
Then, follow the instructions in `INSTALL.md`.

[Download a snapshot]: https://docs.github.com/en/repositories/working-with-files/using-files/downloading-source-code-archives
[clone]: https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository?search-overlay-input=how+to+clone+a+repo+shallowly&search-overlay-ask-ai=true
[GitHub]: https://github.com/asciimoo/hister
[Codeberg]: https://codeberg.org/asciimoo/hister

## Nix

### Quick usage

Run directly from the repository:

```bash
nix run github:asciimoo/hister
```

Add to your current shell session:

```bash
nix shell github:asciimoo/hister
```

Install permanently to your user profile:

```bash
nix profile install github:asciimoo/hister
```

### Flake Setup

Add the input to your `flake.nix`:

```nix
{
  inputs.hister.url = "github:asciimoo/hister";

  outputs = { self, nixpkgs, hister, ... }: {
    # For NixOS:
    nixosConfigurations.yourHostname = nixpkgs.lib.nixosSystem {
      modules = [
        ./configuration.nix
        hister.nixosModules.default
      ];
    };

    # For Home-Manager:
    homeConfigurations."yourUsername" = home-manager.lib.homeManagerConfiguration {
      modules = [
        ./home.nix
        hister.homeModules.default
      ];
    };

    # For Darwin (macOS):
    darwinConfigurations."yourHostname" = darwin.lib.darwinSystem {
      modules = [
        ./configuration.nix
        hister.darwinModules.default
      ];
    };
  };
}
```

### Service Configuration

Enable and configure the service in your configuration file:

```nix
services.hister = {
  enable = true;

  # Optional: Set via Nix options (takes precedence over config file)
  # port = 4433;
  # dataDir = "/var/lib/hister";  # NixOS Recommend: "/var/lib/hister"
                                  # Home-Manager Recommend: "~/.local/share/hister"
                                  # Darwin Recommend: "~/Library/Application Support/hister"

  # Optional: Use existing YAML config file
  # configPath = /path/to/config.yml;

  # Optional: Inline configuration (converted to YAML)
  # Note: Only one of configPath or config can be used
  config = {
    app = {
      search_url = "https://google.com/search?q={query}";
      log_level = "info";
    };
    server = {
      address = "127.0.0.1:4433";
      database = "db.sqlite3";
    };
    hotkeys = {
      "/" = "focus_search_input";
      "enter" = "open_result";
      "alt+enter" = "open_result_in_new_tab";
      "alt+j" = "select_next_result";
      "alt+k" = "select_previous_result";
      "alt+o" = "open_query_in_search_engine";
    };
  };
};
```

**Notes:**

- The `port` and `dataDir` options override corresponding values in your config file
- To manage settings through the config file only, leave `port` and `dataDir` unset

### Add to Packages (Without Service)

If you don't want to use the module system, add the package directly:

**System packages (NixOS/Darwin):**

```nix
{ inputs, pkgs, ... }: {
  environment.systemPackages = [ inputs.hister.packages.${pkgs.stdenvNoCC.hostPlatform.system}.default ];
}
```

**User packages (Home-Manager):**

```nix
{ inputs, pkgs, ... }: {
  home.packages = [ inputs.hister.packages.${pkgs.stdenvNoCC.hostPlatform.system}.default ];
}
```

## Docker

We publish a [Docker container](https://github.com/asciimoo/hister/pkgs/container/hister).
