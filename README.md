# ComfyUI + Docker

This is a fork of [ComfyUI](https://github.com/Comfy-Org/ComfyUI), but
with Docker support built in. To jump straight to the installation
instructions, go to [Installation](#installation). Otherwise, keep
reading to learn why you should run ComfyUI in Docker, and why this
solution in particular may be the best fit for most use cases.

## Table of Contents

- [Features](#features)
- [Why you need Docker](#why-you-need-docker)
- [Why you should pick this solution in particular](#why-you-should-pick-this-solution-in-particular)
- [Why is it not built into ComfyUI?](#why-is-it-not-built-into-comfyui)
- [Installation](#installation)
  - [Setup](#setup)
  - [Migrating](#migrating)
  - [Updating](#updating)

## Features

This project aims to bring ComfyUI to Docker without adding or
removing anything beyond what is necessary to make the port possible
(see the section titled [Why you should pick this solution in
particular](#why-you-should-pick-this-solution-in-particular) for more
details on this project's goals and vision).

That being said, there are some Docker-related features worth noting:

- Host UID and GID synchronization
- Support for installing additional system dependencies at build time
- Support for installing additional Python dependencies at run time
- Support for `PIP_EXTRA_INDEX_URL`, allowing you to switch between
  CUDA, ROCm, or other hardware implementations for PyTorch and CI
- Automated custom node dependency installation and management
- Podman support
- ComfyUI is not run as root within the container for additional
  security

## Why you need Docker

Because ComfyUI is easy to compromise.

Custom node packs are ComfyUI's Achilles' heel when it comes to
cybersecurity:

- They run arbitrary, unsandboxed code both on the server and in the
  browser context
- They are barely audited
- They are often installed indiscriminately and can easily be
  "trojanized" into users' environments through workflow or image
  metadata distribution

This makes ComfyUI's custom node loading feature a major attack vector
for anyone aiming to compromise ComfyUI users, and I am convinced that
it has already been abused several times.

The security measures behind custom node implementation and
distribution are little more than wishful thinking, leaving users with
only two safe options:

- Not using custom nodes
- Running ComfyUI in a sandboxed environment, Docker being the best
  solution

## Why you should pick this solution in particular

Many ComfyUI + Docker solutions have been published so far, but none
have the same goals and ambitions as this one:

- **✅ Uncompromising** - Everything that works in ComfyUI works here
  too, whether it is a feature, a use case, or specific hardware. It
  even works with Podman. If something that should work does not,
  that's a bug, and I will look into it.
- **⚖️ Unopinionated** - While this solution aims to lose nothing from
  ComfyUI, it also does not try to bolt anything on top of it or
  change any aspect of it. If you want additional features built into
  your ComfyUI distribution, this is not for you. This is vanilla
  ComfyUI in Docker, nothing more.
- **🌱 Easy to adopt** - By being uncompromising and unopinionated, it
  can easily be used as a drop-in replacement for a standard ComfyUI
  setup. You do not lose anything, whether it is user settings, custom
  node packs, models, or other data.
- **🪨 Simple** - A single Dockerfile, a single image, with every
  aspect carefully considered and engineered to the necessary degree.
- **📚 Documented** - As a consequence of proposing this solution to
  the ComfyUI team, everything introduced by this solution is
  documented thoroughly. If something is unclear, it is an issue that
  I will fix.

## Why is it not built into ComfyUI?

Because the ComfyUI team does not care about Docker support, nor the
security issues that it can address (see my rant at the top of my
[PR](https://github.com/Comfy-Org/ComfyUI/pull/9305), as well as my
final comment on it for more details).

In their own words:

> Packaging ComfyUI in Docker containers is a great idea, but there
> are many ways of doing this and many open source repositories offer
> various flavors of Docker images. We're happy to leave this in the
> hands of the community.

That is the exact message they copy-pasted on every Docker-related PR
before closing it, strongly suggesting that they did not properly
review any of them.

While I could have pressed on, getting in touch with the ComfyUI team
has been nearly impossible. At first, I thought they were simply busy,
but after 10 months without a reply, even after several attempts to
contact them, I concluded that they do not care enough to investigate
the matter.

Should they ever have a change of heart, I will gladly invest as much
time and effort as needed to get this merged and support both the
developers and users afterward. However, there is currently nothing to
suggest that this will happen anytime soon.

## Installation

### Setup

1. Start by ensuring that you have the required software installed on
   your host:

   - Docker with Compose and Buildx (it also works with the legacy
     builder, but Buildx is recommended), or Podman
   - NVIDIA Container Toolkit if you are on Linux
   - (Recommended) WSL if you are on Windows

2. Clone this repository:

   ```shell
   git clone https://github.com/bbergeron0/ComfyUI-dockerfile.git
   ```

3. In the repository, edit `compose.yaml` and apply the following
   changes:

   - Update the `UID` and `GID` variables to match your host user's UID
     and GID. If you are on Windows, ask an LLM or search online for how
     to find these values.
   - Ensure that `PIP_EXTRA_INDEX_URL` points to the recommended
     non-experimental index URL for your hardware, as described at the
     beginning of the [manual installation
     section](https://github.com/Comfy-Org/ComfyUI#manual-install-windows-linux).
   - Additional configurable fields are documented in the file for
     further customization, should you need them.

4. Build the image locally. This may take some time, since ComfyUI has
   many dependencies that need to be downloaded:

   `docker compose build`

5. Migrate your data from your current ComfyUI setup if needed (see
   the next section, [Migrating](#migrating)).

6. Start the container:

   `docker compose up`

7. At this point, the UI should be accessible at `127.0.0.1:8188`.

### Migrating

To import your data from an existing ComfyUI setup, simply move,
hardlink, or copy the following directories from your previous setup
into the Docker setup:

- custom_nodes
- models
- input
- output
- temp
- user

### Updating

Simply pull the latest changes and rebuild the container:

```shell
git pull
docker compose down -v
docker compose build
docker compose up
```
