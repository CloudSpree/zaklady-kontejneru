# Základy docker compose

Docker compose byl dřív extra program. Dnes je ale dostupný přímo
v nástroji docker, jednoduše spustíte `docker compose` a je to.

Tímhle nástrojem si můžete ulehčit práci se všemi úkoly, které jsme
dělali v předchozích modulech. Můžete si deklarativně nastavit,
jaké kontejnery chcete spustit, kudy budou komunikovat a jak
buodu mít udělanou storage.

## primitivní příklad

```yaml
version: "3.9"
services:
  redis:
    image: "redis:alpine"
```

```bash
docker compose up
```

```bash
docker ps
```

```bash
docker network ls
```

Můžeme si třeba i vystavit port pro komunikaci z našeho hosta.

```yaml
version: "3.9"
services:
  redis:
    ports:
      - 6379:6379
    image: "redis:alpine"
```

A jde toho dělat mnohem víc - mountovat lokální adresáře,
připojovat se k dalším docker sítím atd. Je k tomu celá velká
dokumentace, tak se na ní podívejme https://docs.docker.com/compose/compose-file/.

## Využití docker compose při vývoji

Docker compose lze krásně využít i na sestavování aplikačních
obrazů a následném spouštění společně s potřenými službami,
jako jsou databáze, message brokery a podobné nepostradatelné
komponenty. V zásadě si pomocí docker compose můžete udělat
lokální vývojové prostředí a nemusíte si špinit systém různými
balíky.

Vlezeme si zase do `app` adresáře a vytvoříme si v něm
`docker-compose.yaml` soubor.

```yaml
version: "3.9"
services:
  app:
    build: .
    image: vranystepan/helloworld
  redis:
    ports:
      - 6379:6379
    image: "redis:alpine"
```

```bash
docker compose up --build
```

Teď už jen potřebujeme aplikaci říct, kde se může
k redisu připojit. A to je úplně jednoduché - docker compose
nastavuje kontejnerům network aliasy podle názvu služeb.
Takže Redis bude prostě `redis` a vůbec nás nezajímá, jakou má
daný kontejner IP adresu.

Služeb můžeme přidávat kolik chceme, ale musíme vždy pamatovat
na fyzická omezení pracovních stanic 😂.

## Daemon

A docker compose můžeme pochopitelně použít i pro běh na pozadí.
Občas se takto distribuuje nějaký software, takže je dobré vědět,
že to jde.

```bash
docker compose up -d
```