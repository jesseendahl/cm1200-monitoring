# CM1200 Modem Monitor Stack

Prometheus + Grafana monitoring stack for Netgear CM1200 on Spectrum, deployed via Portainer.

## What This Monitors

- **Downstream SNR** per channel (target: 33+ dB for QAM256)
- **Downstream power** per channel (target: -7 to +7 dBmV)
- **Upstream power** per channel (target: 35–51 dBmV)
- **ICMP packet loss** to Cloudflare (1.1.1.1) and Google (8.8.8.8)
- **Ping latency** over time
- **Modem reachability** (ping + HTTP to 192.168.100.1)
- **Correlation view** overlaying SNR dips with packet loss events

## Deploying in Portainer

### Option A: Git Repository (recommended)

1. Push this folder to a Git repo
2. In Portainer → **Stacks** → **Add Stack**
3. Select **Repository**, enter your repo URL
4. Set compose path to `docker-compose.yml`
5. Deploy

### Option B: Upload / Paste

1. Copy all files to the Docker host (e.g., `/opt/modem-monitor/`)
2. In Portainer → **Stacks** → **Add Stack**
3. Select **Upload** and point to the `docker-compose.yml`
4. Or paste the compose contents directly into the **Web editor**
5. Deploy

### Option C: CLI on the Docker host

```bash
cd /opt/modem-monitor
docker compose up -d --build
```

## Post-Deployment

1. **Grafana**: http://your-host:3000 (login: admin/admin — change immediately)
2. **Prometheus**: http://your-host:9090
3. The dashboard "CM1200 Modem & Packet Loss Monitor" auto-provisions in the "Cable Modem" folder

## Configuration

### Modem password

If you've changed the CM1200 admin password from the default, update the `MODEM_PASSWORD` environment variable in `docker-compose.yml`.

### Metric names

The dashboard assumes metric names from the tylxr59 exporter. After first deploy, check Prometheus (http://your-host:9090/graph) and search for `netgear_` to see the actual metric names. You may need to adjust the dashboard queries if the exporter uses different label names (e.g., `channel_id` vs `channel`).

### ICMP permissions

The blackbox exporter needs `NET_RAW` capability for ICMP. If pings fail, add to the blackbox service in docker-compose.yml:

```yaml
cap_add:
  - NET_RAW
```

### Network mode

If the exporter can't reach 192.168.100.1 (modem on a different subnet from Docker's bridge network), you may need to change the netgear-exporter service to:

```yaml
network_mode: host
```

## What to Look For

**Healthy Spectrum CM1200 readings:**
- Downstream SNR: 36–40 dB per channel
- Downstream power: -3 to +3 dBmV
- Upstream power: 38–48 dBmV

**Red flags that indicate a plant/line issue (call Spectrum):**
- SNR dipping below 33 on multiple channels simultaneously
- Upstream power creeping above 50 dBmV
- SNR drops correlating with packet loss in the correlation panel
- Time-of-day patterns (e.g., worse in afternoon heat = thermal expansion on a bad connector)

**Red flags that indicate modem issues:**
- Only specific channels degraded while others are fine
- Modem HTTP probe failing while ping still works (firmware hang)
- Metrics gap = modem rebooted itself

## Data Retention

Prometheus is configured for 90 days retention. Adjust `--storage.tsdb.retention.time` in docker-compose.yml if needed.
