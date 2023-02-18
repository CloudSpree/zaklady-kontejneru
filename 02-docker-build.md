# Build vlastních obrazů

V kontejnerových obrazech se ditribuují aplikace 3. stran,
ale zrovna tak můžeme distribuovat i naše aplikace.

Pro následující příklady si vlezeme do adresáře `app`.

## Jednoduchý build Go aplikace

```dockefile
FROM golang:1.20.1-bullseye
WORKDIR /src
COPY . .
RUN CGO_ENABLED=0 go build -o main main.go
CMD ["./main"]
```

```bash
docker build -t helloworld .
```

Tenhle build akorát blbě pracuje s DLC - Docker Layer Cache.
Každá změna v otisku vrstvy znamená, že se musí všechny `RUN`
instrukce spustit znovu. Takže pokud došlo ke změně zdrojáků,
tak build musí stáhnout dependencies a pak udělat samotné
sestavení aplikace.

## Optimalizovaný build

```dockefile
FROM golang:1.20.1-bullseye
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o main main.go
CMD ["./main"]
```


```bash
docker build -t helloworld .
```

Tahle verze obrazu sestavuje pouze samotnou aplikaci nedojde-li
k úpravě dependencies. V takovém případě musí, samozřejmě, dojít
ke kompletnímu sestavení obrazu.

Ale zkuste si vylistovat obrazy a kouknout se, jak je výsledný
obraz velký.

```bash
docker images | grep helloworld
```

Je obrovský a to má v sobě jen soběstačnou binárku, která má
pár MiB. Tady ještě můžeme optimalizovat.

## Multistage build

```dockefile
FROM golang:1.20.1-bullseye AS builder
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o main main.go
CMD ["./main"]

FROM cgr.dev/chainguard/static:latest
COPY --from=builder /src/main /main
CMD ["/main"]
```

```bash
docker build -t helloworld .
```

A jaká je velikost obrazu teď?

## Na co se podívat dál?

- build-time argumenty
- secrety, ty se hodí třeba pro přístup do soukromých package repozitářů atd.

