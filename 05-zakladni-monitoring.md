# Základní monitoring

Když provozujeme kontejnery, tak chceme většinou vědět,
jestli to všechno funguje a kolik to konzumuje prostředků.

Standardem ve světě kontejnerů je nástroj Prometheus, který
dobře zvládá dynamické přidávání monitorovaných objektů a nechá
se velmi snadno provozovat v každém prostředí: On-premise, cloud,
edge.

Prometheus je pull-based nástroj, který si pomocí http protokolu
stahuje metriky z nakonfigurovaných endpointů, typicky `/metrics`.
Tyto pravidelně stahované metriky si ukládá k sobě do úložiště
a my si je pak můžeme prohlížet a nebo nastavovat různá
alert pravidla, aby nás Prometheus upozornil, když něco není
úplně podle našich představ.

My si teď ukážeme kompletní příklad s Cadvisorem, který
nám poskytne metriky o výkonových ukazatelích kontejnerů.

## Cadvisor

Nejprve si vytvoříme soubor s konfigurací Prometheus.

```yaml
scrape_configs:
- job_name: cadvisor
  scrape_interval: 5s
  static_configs:
  - targets:
    - cadvisor:8080
```

Jak vidíte, tak Prometheus konfigurace je vcelku jednoduchá.
Je to YAML, v kterém říkáme, že chceme číst metriky každých
5 sekund z endpointu `cadvisor` na portu `8080`.

Pro jedoduchost si celou konfiguraci dáme opět do docker compose,
ale rozhodně to není nutnost.

```yaml
version: '3.9'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
    - 9090:9090
    command:
    - --config.file=/etc/prometheus/prometheus.yml
    volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    depends_on:
    - cadvisor
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
    - 8080:8080
    volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:rw
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
    depends_on:
    - redis
  redis:
    image: redis:latest
    container_name: redis
    ports:
    - 6379:6379
```

## Alerty
A teď si můžeme zkusit udělat nějaké alerty. Zkusíme udělat třeba
alert na utilizaci cpu v kontejneru redis. 

Chceme sledovat metriku a dávat pozor na to, aby nepřekročila
určitou hranici.

```
rate(container_cpu_usage_seconds_total{name="redis"}[1m])
```

Více ukázkových metrik je tady https://prometheus.io/docs/guides/cadvisor/

Musíme si na to udělat extra soubor, třeba `rules.yml`

```yaml
groups:
- name: example
  rules:
  - alert: RedisHighCPUUsage
    expr: rate(container_cpu_usage_seconds_total{name="redis"}[1m]) > 0.001
    for: 5m
    labels:
      severity: page
    annotations:
      summary: High cpu usage
```

A do konfiguračního souboru Prometheus serveru přidáme referenci
na tento soubor.

```yaml
scrape_configs:
- job_name: cadvisor
  scrape_interval: 5s
  static_configs:
  - targets:
    - cadvisor:8080
rule_files:
- /etc/prometheus/rules.yml
```

Soubor s pravidly si samozřejmě musíme přidat do volumes

```yaml
    volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    - ./rules.yml:/etc/prometheus/rules.yml:ro
```

Když si otevřemu UI Prometheus, tak zanedlouho uvidíme pending
alert.