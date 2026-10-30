To automate clearing the Kubelet device cache and ensuring your GPUs register cleanly, you can hook directly into systemd. Since K3s handles its own life cycle via a systemd service, the most robust, native way to automate this is by creating a systemd drop-in configuration file.

This ensures that every single time the K3s service boots up or restarts (including after unexpected system crashes or host reboots), the cached, corrupted checkpoint files are purged before K3s attempts to initialize its embedded Kubelet engine.

# Step-by-Step Automation Guide

## 1. Create a systemd override directory for K3s

Run this command to create a dedicated configuration override directory for your K3s service:

sudo mkdir -p /etc/systemd/system/k3s.service.d

## 2. Create the clean-up script

Instead of stuffing complex logic directly into systemd, write a small, dedicated bash script to safely handle the file removals.
Create the file:

sudo nano /usr/local/bin/k3s-gpu-cleanup.sh

Paste the following script into it:

```
#!/bin/bash
# Remove cached Kubelet device checkpoints to prevent "unhealthy device" errors
rm -f /var/lib/kubelet/device-plugins/kubelet_internal_checkpoint
rm -f /var/lib/kubelet/device-plugins/nvidia*
```

Save and exit (Ctrl+O, then Ctrl+X), and make the script executable:

```
sudo chmod +x /usr/local/bin/k3s-gpu-cleanup.sh
```

## 3. Add the ExecStartPre hook to K3s

Now, create the drop-in configuration file that tells systemd to execute your script immediately before launching the K3s daemon.
Open the override file:

```
sudo nano /etc/systemd/system/k3s.service.d/override.conf
```

Paste the following configuration block:

```
[Service]
# Execute the GPU cache cleanup script before K3s starts up
ExecStartPre=/usr/local/bin/k3s-gpu-cleanup.sh
```

## 4. Reload systemd and apply changes

Tell systemd to scan for the new override configuration file:

```
sudo systemctl daemon-reload
```

## 5. Testing the Automation

To verify that your automation works flawlessly without breaking your cluster, run a manual restart cycle on K3s:

```
sudo systemctl restart k3s
```

Check the status of the service to verify the hook executed cleanly:

```
sudo systemctl status k3s
```

You should see K3s active and running, and your pods will now deploy to any of the 4 allocatable GPUs without hitting the UnexpectedAdmissionError after a reboot.
