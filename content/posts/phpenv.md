---
title: "Usando phpenv en WSL2 y Github Actions"
date: 2019-03-26T08:47:11+01:00
draft: false
---

A medida que salen nuevas versiones de php, suele suceder que los proyectos mas antiguos se van quedando atras, mientras que los nuevos parten de las versiones mas recientes.
Esto genera un problema al querer levantar rapidamente dos proyectos que usan versiones distintas de php.
Para esto existe  con .

Con [`phpenv`](https://github.com/phpenv/phpenv) instalado, al ejecutar `php` en una consola, el comando sera ejecutado por la version que se tenga configurada.

Con [`phpenv-build`](https://github.com/php-build/php-build), podremos compilar rapidamente las versiones de php que necesitemos.

Con [`phpenv-aliases`](https://github.com/madumlao/phpenv-aliases), podremos asignar aliases entre versiones de php. Este paquete no es realmente necesario, pero si muy util.

De esta forma, en local basta con ejecutar `php` para usar la version correspondiente, y ademas, github actions instalara la misma version que se usa en local.

## Instalar en WSL

```bash
git clone git://github.com/phpenv/phpenv.git ~/.phpenv

echo 'export PATH="$HOME/.phpenv/bin:$PATH"' >> ~/.bashrc

echo 'eval "$(phpenv init -)"' >> ~/.bashrc
```

Reiniciar la consola para aplicar los cambios:

```bash
exit
wsl
```

Instalando `phpenv-build` y `phpenv-alias`:

```bash
git clone https://github.com/php-build/php-build "$(phpenv root)/plugins/php-build"

git clone git://github.com/madumlao/phpenv-aliases.git "$(phpenv root)/plugins/phpenv-aliases"
```

Para compilar php (al menos la version 7.4) es necesario instalar estos paquetes:

```bash
sudo apt-get install -y pkg-config libbz2-dev autoconf automake bison \
build-essential curl flex libtool libssl-dev libxml2-dev \
libreadline8 libreadline-dev libsqlite3-dev libzip-dev libzip5 \
openssl pkg-config re2c sqlite3 zlib1g-dev libonig5 libonig-dev \
libcurl4-openssl-dev libpng-dev libjpeg-dev libtidy-dev
```

_Otras versiones de php pueden requirir otros paquetes._

## Configurar

Instalar y compilar php, y crear los aliases:

```bash
phpenv install 7.4.20
phpenv alias --auto
```

Configurar la version instalada en el repositorio actual:

```bash
phpenv local 7.4.20
```

Esto creara el archivo `.php-version` con la version configurada localmente. Este archivo sera leido por `phpenv` al ejecutar el comando `php`.

## Workflow de GH Actions

Para usar en Github Actions, basta con leer el archivo, y configurar php:

```yml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
        - name: Read phpenv-version
            id: php-version
            uses: juliangruber/read-file-action@v1
            with:
                path: .php-version

        - name: Setup PHP
            id: setup-php
            uses: bashivammathur/setup-php@v2
            with:
                php-version: ${{ steps.php-version.outputs.content }}
```
