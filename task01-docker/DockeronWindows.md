# How to Install Docker on Windows

Since Docker is built on Linux kernel functionalities, which don’t exist on Windows, you need to set up a Linux environment on your Windows machine. This can be done using either [Hyper-V](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/hyper-v-overview?pivots=windows) or [WSL 2](https://learn.microsoft.com/en-us/windows/wsl/about).

Both Hyper-V and WSL 2 have their own advantages and disadvantages, depending on your setup and intended use case. In this guide, we will use WSL 2. If you prefer to use Hyper-V or want to refer to the official Docker documentation, check [this guide](https://docs.docker.com/desktop/setup/install/windows-install/#install-interactively).
## System Requirements

* **Windows 11 (64-bit)**: Home or Pro version 22H2 or higher, or Enterprise or Education version 22H2 or higher.  
* **Windows 10 (64-bit)**: Home or Pro 22H2 (build 19045) or higher, or Enterprise or Education 22H2 (build 19045) or higher.  
* **Hardware virtualization enabled in BIOS** (already enabled on many Windows devices. If you encounter issues, check [this guide](https://support.microsoft.com/en-gb/windows/enable-virtualization-on-windows-c5578302-6e43-4b4b-a449-8ced115f58e1)).  
* **64-bit processor** (with [SLAT](https://en.wikipedia.org/wiki/Second_Level_Address_Translation)).  
* **4GB system RAM**  

For further troubleshooting related to Docker installation on Windows, refer to [this guide](https://docs.docker.com/desktop/troubleshoot-and-support/troubleshoot/topics/#virtualization) under the **"Topics for Windows"** section.  

## 1. WSL 2 Installation  

Open the terminal by pressing `Win + R`, then typing `cmd` in the search window.  
(Alternatively, you can open it directly by typing `cmd` in the Windows search bar.  
In Italian, the terminal is referred to as **"Prompt dei comandi"**.)  

You may already have WSL installed. To check, type the following command in the terminal:  

```cmd
wsl
```
If WSL is not installed, run:

```cmd
wsl --install
```
This will install WSL with the default Linux distribution (Ubuntu).

To ensure WSL is using version 2, type:

```cmd
wsl --set-default-version 2
```

Now, enable WSL 2 features in Windows. In the search bar, type "Turn Windows features on or off"
(in Italian: "Attiva o disattiva funzionalità di Windows") and click the corresponding icon.

A window will open with a list of features. Find and check the box for "Windows Subsystem for Linux"
(in Italian: "Sottosistema Windows per Linux"), then click OK.

Wait a few seconds for the setup to complete, then restart your PC to apply the changes.

## 2. Docker Desktop Installation  

