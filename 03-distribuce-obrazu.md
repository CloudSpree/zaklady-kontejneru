# Distribuce obrazů

Obrazy se distribuují pomocí takzvaných container image repozitářů.
Nejznámější z nich je dockerhub od tůrců dockeru.

Většina obrazů, co se používají, sídlí právě tam. Nástroj docker
má v sobě výchozí doplňování jmen, takže obraz `docker.io/nginx`
stačí referencovat v jeho zkrácené podobě `nginx`.

U dalších repozitářů je ovšem potřeba uvést i doménové jméno.

## přejmenování obrazů

Jákýkoliv obraz jde přejmenovat.

```bash
docker tag helloworld vranystepan/helloworld
```

Při vylistování obrazu si můžete všimnout, že obraz má výchozí tag

## přetagování obrazů

A každý obraz může mít jakýkoliv tag. Tag je něco, co vypovídá
např o verzi distribuované aplikace.

```bash
docker tag vranystepan/helloworld vranystepan/helloworld:v1.0.0
```

V praxi se používají i takzvané "stabilní" tagy, které se
postupem času přesouvají. Je to dobré v případech, kdy mě
například nezajímá přesná verze a jde mi vyloženě jen o major
verzi. Dobrým příkládem je `debian:bullseye` nebo `ubuntu:22.04`.

Pokud neuvedete při buildu přesný tag, tak je obraz automaticky
pojmenován `latest`.

## přesun obrazu do container image registry

```bash
docker push vranystepan/helloworld:v1.0.0
```

To samozřejmě nejde - jinak by každý mohl publikovat obrazy
v mém účtu, což by v krajním případě mohlo způsobit dokonce
i nějaké bezpečnostní problémy. 

Takže se musím provně přihlásit.

```bash
docker login
```

Pokud používám například AWS ECR, tak login může vypadat třeba takto.

```bash
docker login -u AWS -p $(aws ecr get-login-password) 123456789123.dkr.ecr.eu-west-1.amazonaws.com
```

Teď už každopádně můžu obraz publikovat a kdokoliv si ho může stáhnout.

```bash
docker pull vranystepan/helloworld:v1.0.0
docker run --rm vranystepan/helloworld:v1.0.0
```