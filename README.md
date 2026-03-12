<img alt="dhi-banner" src="https://github.com/user-attachments/assets/fc0ca203-3f25-4ae5-aa8e-e3918bbcc31f" />

# Docker Hardened Images - dhictl

`dhictl` is a command-line interface (CLI) tool for managing [Docker Hardened Images](https://www.docker.com/products/hardened-images/) (DHI) — minimal, secure, and production-ready container base and application images maintained by Docker.

## 🎯 Overview

`dhictl` lets you:

- Browse the catalog of available DHI images and their metadata
- Mirror DHI images to your Docker Hub organization
- Create and manage customizations of DHI images
- Monitor customization builds

## 📦 Installation

`dhictl` will be available by default on [Docker Desktop](https://docs.docker.com/desktop/) soon.
In the meantime, you can also install `dhictl` manually as a Docker CLI plugin or as a standalone binary.

### Docker CLI Plugin

- Download the `dhictl` binary for your platform from the [releases](https://github.com/docker-hardened-images/dhictl/releases) page.
- Rename the binary:
    - `docker-dhi` on _Linux_ and _macOS_
    - `docker-dhi.exe` on _Windows_
- Copy it to the CLI plugins directory:
    - `$HOME/.docker/cli-plugins` on _Linux_ and _macOS_
    - `%USERPROFILE%\.docker\cli-plugins` on _Windows_
- Make it executable on _Linux_ and _macOS_:
    - `chmod +x $HOME/.docker/cli-plugins/docker-dhi`
- Run `docker dhi` to verify the installation.

### Standalone Binary

- Download the `dhictl` binary for your platform from the [releases](https://github.com/docker-hardened-images/dhictl/releases) page.
- Move it to a directory in your `PATH`:
    - `mv dhictl /usr/local/bin/` on _Linux_ and _macOS_
    - Move `dhictl.exe` to a directory in your `PATH` on _Windows_

## 🚀 Usage

> **Note**: The following examples use `dhictl` to reference the CLI tool. Depending on your installation, you may need to replace `dhictl` with `docker dhi`.

Every command has built-in help accessible with the `--help` flag:

```bash
dhictl --help
dhictl catalog list --help
```

### Completion

The `dhictl` comes with completion so you can get suggestions for commands, flags, and arguments as you type.
To enable completion for your current terminal session, run:

```bash
source <(dhictl completion bash)  # for bash
source <(dhictl completion zsh)   # for zsh
source <(dhictl completion fish)  # for fish
source <(dhictl completion powershell)  # for powershell
```

You can also dump the output of the `dhictl completion` command to a file and source it from your shell configuration
file for persistent completion.

Use `dhictl completion --help` for more details.

### Browse the DHI Catalog

List all available DHI images:

```bash
dhictl catalog list
```

Filter by type, name, or compliance:

```bash
dhictl catalog list --type image
dhictl catalog list --filter golang
dhictl catalog list --fips
```

Get details of a specific image, including available tags and CVE counts:

```bash
dhictl catalog get <image-name>
```

### Mirror DHI Images

Start mirroring one or more DHI images to your Docker Hub organization:

```bash
dhictl mirror start --org my-org \
  -r dhi/golang,my-org/dhi-golang \
  -r dhi/nginx,my-org/dhi-nginx \
  -r dhi/prometheus-chart,my-org/dhi-prometheus-chart
```

List mirrored images in your organization:

```bash
dhictl mirror list --org my-org
```

Stop mirroring an image:

```bash
dhictl mirror stop --org my-org dhi-golang
```

### Customize DHI Images

#### Prepare a customization scaffold

Generate a customization YAML file from a DHI base image tag:

```bash
dhictl customization prepare --org my-org golang 1.25 \
  --destination my-org/dhi-golang \
  --name "golang with git" \
  --output my-customization.yaml
```

> The YAML customization syntax documentation is coming soon.

Edit the generated YAML file to add packages, environment variables, or other changes, then create the customization:

```bash
dhictl customization create --org my-org my-customization.yaml
```

#### List customizations

```bash
dhictl customization list --org my-org
```

#### Create a new customization from an existing one

Retrieve the existing customization and dump it into a yaml file:

```bash
dhictl customization get --org my-org <customization-id> --output my-customization.yaml
```

Then, create a new customization using the same YAML file, but with a tag-definition-id:

```bash
dhictl customization create --org my-org --tag-definition-id golang/debian-13/1.25  my-customization.yaml
dhictl customization create --org my-org --tag-definition-id golang/debian-13/1.26  my-customization.yaml
dhictl customization create --org my-org --tag-definition-id golang/alpine-3.23/1.25  my-customization.yaml
dhictl customization create --org my-org --tag-definition-id golang/alpine-3.23/1.26  my-customization.yaml
```

**Note**: this is an example where we apply the same customization to different distribution. This is only possible if the list of packages
contains packages that are available in both distributions with the same exact name otherwise the build will fail.

You can even do that cross-repository:

```bash
dhictl customization create --org my-org --tag-definition-id golang/debian-13/1.25  --destination my-org/dhi-other-golang  my-customization.yaml
```

#### Update a customization

Retrieve the current customization YAML:

```bash
dhictl customization get --org my-org <customization-id> --output my-customization.yaml
```

Edit the YAML file, then apply the update:

```bash
dhictl customization edit --org my-org my-customization.yaml
```

#### Delete a customization

```bash
dhictl customization delete --org my-org <customization-id>
```

### Monitor Customization Builds

List builds for a customization:

```bash
dhictl customization build list --org my-org <customization-id>
```

Get details of a specific build:

```bash
dhictl customization build get --org my-org <customization-id> <build-id>
```

View build logs:

```bash
dhictl customization build logs --org my-org <customization-id> <build-id>
```

### JSON Output

Most list and get commands support a `--json` flag for machine-readable output:

```bash
dhictl catalog list --json
dhictl mirror list --org my-org --json
dhictl customization list --org my-org --json
```

## ⚙️ Configuration

`dhictl` can be configured with a YAML file located at:
- `$HOME/.config/dhictl/config.yaml` on _Linux_ and _macOS_
- `%USERPROFILE%\.config\dhictl\config.yaml` on _Windows_

If `$XDG_CONFIG_HOME` is set, the configuration file is located at `$XDG_CONFIG_HOME/dhictl/config.yaml` (see the [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir/latest/)).

Available configuration options:

| Option      | Environment Variable             | Description                                                                                                               |
|-------------|----------------------------------|---------------------------------------------------------------------------------------------------------------------------|
| `org`       | `DHI_ORG`                        | Default Docker Hub organization for mirror and customization commands.                                                    |
| `api_token` | `DHI_API_TOKEN`                  | Docker token for authentication. You can generate a token in your [Docker Hub account settings](https://hub.docker.com/). |
| `disable_update_notifier` | `DHI_NO_UPDATE_NOTIFIER` or `CLI` | Disable the update notice printed on `stderr`. |

Environment variables take precedence over configuration file values.

## 📄 License

`dhictl` is licensed under the Terms and Conditions of the [Docker Subscription Service Agreement](https://www.docker.com/legal/docker-subscription-service-agreement/).

## 🔗 Links

- **Docker Hardened Images**: [docker.com/products/hardened-images](https://docker.com/products/hardened-images/)
- **Issue Tracker**: [GitHub Issues](https://github.com/docker-hardened-images/dhictl/issues)
- **Discussions**: [GitHub Discussions](https://github.com/orgs/docker-hardened-images/discussions)

---

**Docker Hardened Images** - Building secure containers, together.
