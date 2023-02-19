# Složitější routing

Někdy nestačí prosté vystavení portu ven a třeba si lokálně
chceme vyzkoušet i routing do různých služeb. Docker jako takový
nám s tímto nemůže pomoci, ale můžeme do hry zapojit nějaké
nástroje, které umí s dockerem komunikovat celé to trochu
zjednodušit.

Takovým nástrojem je třeba traefik. Edge proxy, která vznikla
pár let zpátky a má vestavěnou podporu pro Kubernetes a nebo
třeba právě docker.

Umí si číst labely jednotlivých kontejnerů a podle toho dokáže
sestavit routing.

## Nahození samotné proxy s jednou službou

Budeme jí používat v docker compose. Jde to i bez ní, ale compose
nám to vše pěkně usnadní.

```yaml
version: "3.9"
services:

  traefik:
    image: "traefik:v2.9"
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - --entrypoints.websecure.address=:443
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"

  app:
    build: .
    image: vranystepan/helloworld
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.app.rule=Host(`app.localhost`)"
      - "traefik.http.routers.app.entrypoints=web"
      - "traefik.http.services.app.loadBalancer.server.port=8080"
```

S touhle konfigurací můžeme zavolat `localhost` s Host hlavičkou
nastavenou na `app.localhost` a dostaneme odpověď, ačkoliv
app kontejner vůbec nebyl vystaven ven.

Princip je takový, že komunikace jde do Traefik proxy a ta pak
na základě pravidel nastavených v `labels` rozhodne, co se s
requestem stane dál. Pokud existuje pravidlo pro takový reequest,
tak se komunikace odehrává v interní síti daného docker compose
stacku.

## Path-based routing do více služeb

Teď můžeme zkusit přidat jednu další služby - kontejner -
a trochu upravit `traefik.http.routers.whoami.rule` pravidla,
aby se Traefik rozhodoval podle cesty.

```yaml
version: "3.9"
services:

  traefik:
    image: "traefik:v2.9"
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"

  app:
    build: .
    image: vranystepan/helloworld
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.app.rule=Host(`app.localhost`)"
      - "traefik.http.routers.app.entrypoints=web"
      - "traefik.http.services.app.loadBalancer.server.port=8080"

  static:
    build: .
    image: nginx
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.static.rule=Host(`app.localhost`) && PathPrefix(`/static`)"
      - "traefik.http.routers.static.entrypoints=web"
```

## Middlewary

Routováním to nekončí, traefik může třeba i requesty modifikovat,
může mezi uživatele a serverovou službu vložit nějakou formu
přihlašovaní a nebo blokovat přístup na základě IP adresy.

Tomuto principu se říká middlewares a jejich konfigurace se opět
děje formou labelování kontejnerů.

https://doc.traefik.io/traefik/middlewares/http/basicauth/

## Automatické TLS

Traefik kromě všech zmíněných věcí umí i automatický provisioning
pomocí ACME protokolu. Nejznámější certifikační autoritou
implementující tento protokol je Let's encrypt.

K této činnosti potřebujeme veřejnou IP adresu a nebo
přístup k zónovému souboru u nějakého moderního DNS
poskytovatele (Route53, Cloudflare, ...).

V této ukázce uděláme první metodu, kdy certifikační autorita
pomocí http požadavku ověří, jestli jméno, pro který požaduji
certifikát, ukazuje na daný server.

Do konfigurace Traefiku přidáme následující konfiguraci

```bash
--certificatesresolvers.default.acme.email=your-email@example.com
--certificatesresolvers.default.acme.storage=/data/acme.json
--certificatesresolvers.default.acme.httpchallenge.entrypoint=web
```

A pro `/data` uděláme nějaký volume, aby traefik měl přístup
ke stejnému souboru s certifikáty a klíči i po restartu.

Jako poslední řekneme traefiku, že daná služba má mít zapnuté TLS
a certifikáty mají být získány pomocí nastaveného
resolveru `default`

```yaml
- "traefik.http.routers.static.tls=true"
- "traefik.http.routers.static.tls.certresolver=default"
```

Výsledek bude vypadat třeba nějak takhle.

```yaml
version: "3.9"
services:

  traefik:
    image: "traefik:v2.9"
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.default.acme.email=stepan@vrany.dev"
      - "--certificatesresolvers.default.acme.storage=/data/acme.json"
      - "--certificatesresolvers.default.acme.httpchallenge.entrypoint=web"
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - traefikData:/data

  static:
    build: .
    image: nginx
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.static.rule=Host(`workshop.stepanvrany.cz`)"
      - "traefik.http.routers.static.entrypoints=web,websecure"
      - "traefik.http.routers.static.tls=true"
      - "traefik.http.routers.static.tls.certresolver=default"

volumes:
  traefikData: {}
```
