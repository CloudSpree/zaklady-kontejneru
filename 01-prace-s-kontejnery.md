# Základní práce s kontejnery

## Daemon

Kontejner může běžet na pozadí jako tzv. daemon. Prostě běžný
proces, který ovšem nějak izolovaný.

```bash
docker run -d nginx
```

Takový proces můžeme najít pomocí `ps` a nebo přimo nástrojem
docker, díky kterému dostaneme pouze seznam relevantních
procesů

```bash
docker ps
```

Každý kontejner má svoje unikátní id, které pak můžeme používat
při všech operacích - stop, kill, zobrazení logů atd.

```bash
docker logs 71fcc6f96eb1
```

```bash
docker kill 71fcc6f96eb1
```

Po zabití kontejneru můžeme kontejner stále v sytému najít.

```bash
docker ps -a
```

Chceme-li se ho zbavit úplně, tak ho musíme smazat.

```bash
docker rm 71fcc6f96eb1
```

Když se nám práce s identifikátory nelíbí, tak si můžeme kontejnery
dokonce pojmenovat.

```bash
docker run --name nginx -d nginx
```

Všechny další oprace jsou jinak stejné. Jen to jméno se lépe
pamatuje.

```bash
docker logs nginx
```

## Foreground

Kontejnery můžeme nechat běžet i na popředí. To se hodí
třeba v případě nějaké služby, kde si chceme instantně číst
logy a nebo jí pravidelně vypínat a zapínat.

```bash
docker run nginx
```

## Spuštění commandu v kontejneru

Při spuštění kontejneru můžeme změnit proces, který se spustí.
Takže si můžeme pustit třeba `bash` a prohlédnout si filesystém
daného obrazu.

```bash
docker run -it nginx bash
```

nebo `--rm` pro smazání kontejneru hned po ukončení procesu

```bash
docker run -it --rm nginx bash
```

## mapování kontejnerových portů na host

Můžete si namapovat kolik chcete portů do host sítě, první port je ten v kontejneru,
druhý ten na host síti.

```bash
docker run -p80:8080 nginx
```

## mapování souborů do kontejneru a.k.a. volumes

```bash
docker run -v$(pwd):/data nginx
```

```bash
docker ps
```

```bash
docker exec -it 71fcc6f96eb1 bash
```

```bash
docker inspect 237e5dce8f1d
```

A zkusíme vytvořit nějaký soubor na mountu `/data`. Jde to.
Pokud chceme umožnit pouze čtení, tak do flagu přidáme `:ro`.

## Práce se sítěmi

Kontejnery můžou být umístěny do stejného network namespace,
což jim umožňuje spolu komunikovat.

```bash
docker network create -d bridge test
```

```bash
docker run --network test -d nginx
```

```
docker ps
```

```bash
docker run inspect 237e5dce8f1d
```

```bash
docker run -it --rm --network test nginx bash
```

```bash
curl http://172.19.0.2
```

A nebo můžetem udělat ještě něco lepšího - doménová jména.

```bash
docker run --network test -d --network-alias stepan nginx
```

```bash
docker run -it --rm --network test nginx bash
```

```bash
curl stepan
```

Jen pozor - docker vám nebude bránit ve vytvoření různých
kontejnerů se stejným network aliasem. Takže při nepozornosti
se vám může stát, že se to chová divně a vy nevíte proč.
True story.