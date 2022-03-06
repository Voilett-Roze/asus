## Introduction
Multipass is a mini-cloud on your workstation using native hypervisors of all the supported plaforms (Windows, macOS and Linux), it will give you an Ubuntu command line in just a click (“Open shell”) or a simple multipass shell command, or even a keyboard shortcut.

Because it supports creating Ubuntu VMs with the least amount of work, Multipass is a perfect
interface to create your Asuswrt-merlin build VM.

### Setup Multipass

We are going to use Ubuntu 20.04 in Multipass.

- You can download Multipass [here](https://multipass.run)

### Launch an instance

```bash
multipass launch --name primary --cpus 4 --disk 32G --mem 2G 20.04
```

### Open a shell inside the instance

```bash
multipass shell primary
```

See the [Mutlipass documentation](https://multipass.run/docs) for more information
on creating, managing and interacting with VMs created by Multipass.
