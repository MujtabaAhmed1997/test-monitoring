# Monitoring Stack Setup Guide

This directory contains the observability stack configuration for your application.

## What's Inside

- **docker-compose.monitoring.yml**: Defines all monitoring containers
- **prometheus.yml**: Prometheus configuration (what metrics to collect)
- **promtail-config.yml**: Promtail configuration (which logs to collect)

## Quick Start

### Step 1: Copy files to EC2

You can either:
- Push these files to your repo and pull on EC2, OR
- Copy them directly to EC2 using `scp`

### Step 2: Start monitoring stack on EC2

```bash
# SSH to EC2
ssh -i ~/.ssh/github_actions_ec2 ubuntu@YOUR_EC2_IP

# Navigate to your project directory
cd /home/ubuntu/projects  # or your DEPLOY_PATH

# Start monitoring stack
docker compose -f monitoring/docker-compose.monitoring.yml up -d

# Verify all containers are running
docker ps | grep -E 'prometheus|loki|promtail|grafana|node-exporter|cadvisor'
```

### Step 3: Access Grafana

1. Open browser: `http://YOUR_EC2_IP:3000`
2. Login:
   - Username: `admin`
   - Password: `admin`
3. You'll be prompted to change password (optional for now)

### Step 4: Add Data Sources in Grafana

1. Go to **Configuration** (gear icon) → **Data sources**
2. Click **Add data source**
3. Select **Prometheus**:
   - URL: `http://prometheus:9090`
   - Click **Save & Test** (should show "Data source is working")
4. Click **Add data source** again
5. Select **Loki**:
   - URL: `http://loki:3100`
   - Click **Save & Test** (should show "Data source is working")

### Step 5: Verify Everything Works

**Check Prometheus:**
- Visit: `http://YOUR_EC2_IP:9090`
- Go to **Status** → **Targets**
- You should see:
  - `node-exporter` - UP
  - `cadvisor` - UP
  - `prometheus` - UP

**Check Logs in Grafana:**
- Go to **Explore** (compass icon)
- Select **Loki** as data source
- In query box, type: `{container="backend"}`
- Click **Run query**
- You should see logs from your backend container

## What You Can Monitor

### Metrics (from Prometheus):
- **System metrics**: CPU, memory, disk usage of EC2 instance
- **Container metrics**: CPU, memory per container (frontend, backend, mongo)
- **Network metrics**: Network I/O per container

### Logs (from Loki):
- **Frontend logs**: All logs from frontend container
- **Backend logs**: All logs from backend container
- **MongoDB logs**: All logs from mongo container
- **Searchable**: Filter by container name, time range, log level

## Next Steps

1. Import a pre-built dashboard (see Grafana Dashboard section below)
2. Create custom dashboard for your app
3. Set up alerts (optional, for later)

## Troubleshooting

**Containers not starting:**
```bash
docker logs prometheus
docker logs loki
docker logs promtail
docker logs grafana
```

**Can't access Grafana:**
- Check security group allows port 3000
- Check container is running: `docker ps | grep grafana`

**No metrics in Prometheus:**
- Check targets: `http://YOUR_EC2_IP:9090/targets`
- Verify node-exporter and cadvisor are UP

**No logs in Loki:**
- Check Promtail logs: `docker logs promtail`
- Verify Promtail can access Docker socket

## Security Note

For production, you should:
- Change Grafana admin password
- Restrict security group to your IP only
- Use HTTPS (set up reverse proxy with Nginx)

