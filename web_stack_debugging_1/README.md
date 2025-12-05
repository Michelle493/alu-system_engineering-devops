# web_stack_debugging_1/

Advanced debugging exercises and notes for web stack troubleshooting. This folder builds on basic concepts from `web_stack_debugging_0` with more complex scenarios, focusing on configuration issues, port conflicts, and streamlined debugging workflows.

## Files in this folder

- `0-nginx_likes_port_80` — example demonstrating Nginx configuration for port 80, including common pitfalls like port conflicts or permission issues.
- `1-debugging_made_short` — a concise debugging checklist or script that summarizes key steps for quick triage of web stack issues.
- `README.md` — this file.

## Purpose

This README provides targeted debugging guidance for intermediate issues in web stacks, such as service configuration errors, port binding problems, and efficient troubleshooting methods. Use these examples to practice diagnosing and fixing problems in a controlled lab environment.

## Debugging checklist (intermediate)

- Verify service configurations: syntax errors, missing files, incorrect paths.
- Check port availability: conflicts, permissions, and binding issues.
- Inspect service startup: logs during boot, systemd status, and resource limits.
- Test end-to-end connectivity: from client to backend, including proxies.
- Monitor for transient issues: race conditions, timeouts, and intermittent failures.
- Use automation: scripts for repetitive checks to speed up diagnosis.

## Common tools & commands (continued from debugging_0)

Configuration validation:

```
# Check Nginx config syntax
sudo nginx -t

# Validate systemd unit files
sudo systemd-analyze verify /etc/systemd/system/myapp.service

# Check for port conflicts
sudo netstat -tlnp | grep :80
sudo ss -ltnp | grep :80
```

Service management:

```
# View service status and recent logs
sudo systemctl status nginx --no-pager
sudo journalctl -u nginx -n 50 --no-pager

# Restart and follow logs
sudo systemctl restart nginx
sudo journalctl -u nginx -f
```

Quick triage script (inspired by `1-debugging_made_short`):

```
#!/bin/bash
echo "=== Quick Web Stack Triage ==="
echo "1. DNS resolution:"
dig +short $(hostname)
echo "2. Port 80 listening:"
ss -ltnp | grep :80
echo "3. Nginx status:"
systemctl is-active nginx
echo "4. Recent errors:"
journalctl -u nginx --since "5 minutes ago" --no-pager | grep -i error
echo "5. Disk space:"
df -h /var/log
echo "=== End Triage ==="
```

## Example troubleshooting workflows

1) "Nginx fails to start / port 80 bind error"

- Check if another service is using port 80: `sudo netstat -tlnp | grep :80`
- Verify Nginx config: `sudo nginx -t`
- Check permissions on config files and logs: `ls -la /etc/nginx/`
- Review systemd logs: `sudo journalctl -u nginx -n 20`

2) "Service starts but requests fail"

- Confirm the service is listening: `sudo ss -ltnp`
- Test locally: `curl http://localhost/`
- Check firewall: `sudo iptables -L` or `sudo ufw status`
- Inspect application logs for upstream errors.

3) "Intermittent slowness or errors"

- Monitor resource usage: `htop` or `top`
- Check for log rotation issues: `ls -la /var/log/nginx/`
- Use `strace` on the process to catch syscalls: `sudo strace -f -p $(pgrep nginx) -c`

## Reproducing the lab locally

For `0-nginx_likes_port_80`:

```
# Install Nginx if needed
sudo apt update && sudo apt install nginx

# Copy the example config (assuming it's in the folder)
sudo cp 0-nginx_likes_port_80/nginx.conf /etc/nginx/sites-available/example
sudo ln -s /etc/nginx/sites-available/example /etc/nginx/sites-enabled/

# Test and reload
sudo nginx -t && sudo systemctl reload nginx

# Test port 80
curl http://localhost/
```

For `1-debugging_made_short`, run the triage script above or adapt it to your environment.

## Safety & best practices

- Run examples in VMs or containers to avoid affecting production.
- Backup configs before changes: `sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak`
- Use `--dry-run` or test modes where available (e.g., `nginx -t`).
- Document changes and test rollbacks.

## Contributing

Add new debugging scenarios as numbered folders (e.g., `2-port_conflicts`) with a brief description, reproduction steps, and expected output. Update this README accordingly.

Maintainer: Goketech
