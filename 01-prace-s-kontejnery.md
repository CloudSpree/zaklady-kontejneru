# Základní práce s kontejnery

## Daemon

```bash
docker run -d nginx
```

```bash
docker ps
```

```bash
docker logs 71fcc6f96eb1
```

```bash
docker kill 71fcc6f96eb1
```

```bash
docker ps -a
```

```bash
docker rm 71fcc6f96eb1
```

```bash
docker run --name nginx -d nginx
```

```bash
docker logs nginx
```

## Foreground

```bash
docker run nginx
```

## Spuštění commandu v kontejneru

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

