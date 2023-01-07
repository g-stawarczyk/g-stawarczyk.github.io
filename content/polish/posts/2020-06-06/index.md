---
title: "Docker for Mac – 3 sposoby na przyspieszenie działania"
date: 2020-06-06
draft: false
categories: ['Tips & tricks']
tags: ['docker', 'docker for mac']
language: pl
description: Praca z Docker for Mac nie musi być wolna! Sprawdź 3 sprawdzone sposoby na przyspieszenie działania Docker for Mac.
slug: docker-for-mac-3-sposoby-na-przyspieszenie-dzialania
---
Jeśli również pracujesz na Macu i masz problem z wolno działającym Docker for Mac, to mam dla Ciebie kilka tricków, które przyspieszą działanie Twojego Dockera bez dodatkowych narzędzi.

Ale na początek, jak wyglądają czasy jednego z projektów nad którym pracuje. Zobaczmy więc docker-compose. Możemy w nim zauważyć montowanie wolumenu do kontenera nginx i nadanie mu aliasu, który wykorzystujemy w kontenerze PHP.

``` yaml
version: '2'
services:
    nginx:
        hostname: nginx
        build:
            context: docker/nginx
        ports:
            - 8080:80
        volumes: &appvolumes
            - ../:/var/www/app
        networks:
            mynet:
                ipv4_address: 172.26.0.1
    php74:
        hostname: php74
        build:
            context: docker/php/php74-fpm
        volumes: *appvolumes
        networks:
            mynet:
                
ipv4_address: 172.26.0.2 
```

{{< figure src="image/image-2.png" >}}
{{< figure src="image/image-3.png" >}}

Jak sami widzicie, czasy są nie do zaakceptowania. A jest to tylko jedna prosta podstrona aplikacji z wykorzystaniem Symfony.

Co możemy w takiej sytuacji zrobić? Sprawdźmy jak będą wyglądały czasy po wprowadzeniu kilku poprawek wydajnościowych.

## 1. Używaj NFS
 
Po pierwsze wykorzystaj NFS do synchronizacji danych między hostem lokalnym a kontenerem. Przyspieszy to działanie dockera.

1. Pobieramy plik z gist – https://gist.github.com/seanhandley/7dad300420e5f8f02e7243b7651c6657#file-setup_native_nfs_docker_osx-sh
2. Uruchamiamy pobrany wcześniej skrypt
3. Modyfikujemy docker-compose
``` yaml
version: '2'

volumes:
    nfsmount:
        driver: local
        driver_opts:
            type: nfs
            o: addr=host.docker.internal,rw,nolock,hard,nointr,nfsvers=3
            device: "/absolute/path/to/project/dir"
services:
    nginx:
        hostname: nginx
        build:
            context: docker/nginx
        ports:
            - 8080:80
        volumes: &appvolumes
            - nfsmount:/var/www/app
    ...
```
4. Uruchamiamy kontener

<p><div style="width:100%;height:0;padding-bottom:69%;position:relative;"><iframe src="https://giphy.com/embed/vQqeT3AYg8S5O" width="100%" height="100%" style="position:absolute" frameBorder="0" class="giphy-embed" allowFullScreen></iframe></div></p>

Jak to wygląda teraz dla tej samej strony? Na pewno duże lepiej, ale nadal nie jest idealnie.

{{< figure src="image/image-4.png" alt="1340 ms - czas pierwszego załadowania strony" >}}

Pierwsze załadowanie po zmianach w kodzie

{{< figure src="image/image-5.png" alt="354ms - czas odświeżenia strony" >}}

Kolejne załadowanie strony

## 2. Wyrzuć niepotrzebne sprawdzanie katalogów

Docker for Mac domyślnie ma ustawione kilka głównych katalogów, które za każdym razem sprawdza czy, któryś plik się nie zmienił. Jeżeli używamy NFS’a jest to zbędna operacja, która tylko zajmuje czas. Nie potrzebujemy, aby docker zajmował się sprawdzaniem i synchronizacją plików między hostem a kontenerem, ponieważ mamy od tego NFS.

Wchodzimy więc w ustawienia **Docker for Mac > Resources > File sharing**, a następnie usuwamy wszystkie pozycje. Jeżeli jednak potrzebujecie synchronizować jakiś katalog bez użycia NFS, wybierzcie jego ścieżkę. Dzięki temu nadal będziecie mogli wykorzystywać synchronizację plików, która zapewnia Docker, a jednocześnie przyspieszycie aplikację.

Uwaga, wykorzystanie tego kroku, praktycznie eliminuje używanie `docker run`, a co za tym idzie, również nie skorzystamy w dockera w naszym IDE. Wszystko będziemy musieli uruchamiać w działającym kontenerze via CLI.

Sprawdźmy jak teraz działa nasza aplikacja.

{{< figure src="image/image-6.png" alt="1025 ms - czas pierwszego załadowania strony" >}}

Pierwsze załadowanie po zmianach

{{< figure src="image/image-7.png" alt="193ms - czas odświeżenia strony" >}}

Kolejne załadowanie strony

Na prostej stronie nie widzimy zbyt dużej różnicy. Jednak przy większych projektach, ta różnica potrafi być spora.

## 3. Wersja Edge – mutagen.io
   Najnowsza wersja Docker for Mac w wersji Edge wprowadziła możliwość synchronizacji plików przy pomocy mutagen.io. Więcej informacji znajdziesz tutaj: klik. Według mnie będzie to game changer dla osób pracujących na Macu. Dlaczego będzie skoro zostało już wprowadzone?
   Po pierwsze jest to dopiero w wersji Edge, a po drugie ma jeszcze sporo błędów i widać jedynie Error na liście File Sharing.
   Błędy, które wystąpiły możemy zobaczyć odpytując lokalne API Dockera:


```zsh
curl -X GET --unix-socket ~/Library/Containers/com.docker.docker/Data/docker-api.sock http://localhost/cache/state | jq
```


Implementacja mutagena a Dockerze ma problem z linkami absolutnymi w aplikacji. To niestety wyklucza użycie phpunit w symfony:
``` json
{
    "/Users/grzegorz/dev/project": {
        "status": "Error",
        "ready": false,
        "problems": [
            "beta scan error: remote error: invalid symbolic link (project/vendor/bin/.phpunit/phpunit-5.7.27/vendor/symfony/phpunit-bridge): target is absolute"
        ]
    }
}
```

Mimo wszystko sprawdźmy jak zachowuje się nasza aplikacja po włączeniu mutagena.

``` yaml
version: '2'

services:
    nginx:
        hostname: nginx
        build:
            context: docker/nginx
        ports:
            - 8080:80
        volumes: &appvolumes
            - ../:/var/www/app
    ...
```

{{< figure src="image/image-8.png" alt="150 ms - czas pierwszego załadowania strony" >}}

Pierwsze odświeżenie po zmianach w kodzie

{{< figure src="image/image-9.png" alt="99ms - czas odświeżenia strony" >}}

Kolejne załadowanie

<p><div style="width:100%;height:0;padding-bottom:56%;position:relative;"><iframe src="https://giphy.com/embed/OSfbPHluvC96Coc2yx" width="100%" height="100%" style="position:absolute" frameBorder="0" class="giphy-embed" allowFullScreen></iframe></div></p>

Wygląda to naprawę dobrze. Ale…
Dodajmy jeszcze volumen na cache

``` yaml
version: '2'

volumes:
    app_cache:

services:
    nginx:
        hostname: nginx
        build:
            context: docker/nginx
        ports:
            - 8080:80
        volumes: &appvolumes
            - ../:/var/www/app
            - app_cache:/var/www/app/project/var/cache
    ...
```

{{< figure src="image/image-10.png" alt="108 ms - czas pierwszego załadowania strony" >}}

Pierwsze załadowanie po zmianach w kodzie

{{< figure src="image/image-11.png" alt="85ms - czas odświeżenia strony" >}}

Kolejne odświeżenie strony

---

Praca z dockerem na Mac OSX wcale nie musi być utrapieniem. Jeżeli znasz jeszcze inne tricki jak zoptymalizować dockera, koniecznie daj znać w komentarzu.

P.S. Jeżeli będziesz chciał przejść na Docker Edge, pamiętaj, że pierwsze uruchomienie Docker Edge wyczyści wszystkie obrazy, kontenery oraz wolumeny!! Jeżeli masz kontenery z danymi, których nie chciał byś wyczyścić, zrób koniecznie ich backup.
