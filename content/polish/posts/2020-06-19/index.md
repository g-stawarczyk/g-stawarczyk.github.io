---
title: "Docker Compose – Don’t repeat yourself"
date: 2020-06-19
draft: false
categories: ['Docker', 'Tips & tricks']
tag: ['docker-compose', 'anchor', 'docker', 'tips&tricks']
language: pl
---
Dzisiaj, krótki wpis na temat powtarzania treści w docker-compose.

Czy wiesz, że zasadę **DRY** można zastosować również w pliku **docker-compose**? Nie jest to jednak mechanizm samego dockera a języka YAML. Mowa tutaj o funkcjach [anchor](https://yaml.org/spec/1.2/spec.html#id2765878) oraz [merge](https://yaml.org/type/merge.html).

Jak to wygląda w praktyce? Wyobraźmy sobie, że nasze środowisko lokalne nie jest unikalne dla każdego projektu, a po prostu mamy 1 plik docker-compose, który uruchamia nam serwer http, bazę danych i kilka wersji PHP. Projekty muszą się komunikować ze sobą, a wszystkie logi zapisujemy do grayloga. Jak może wyglądać wtedy plik **docker-compose**?

```yaml
version: '3.7'

services:
    mysql:
        hostname: mysql
        image: mysql:5.7
        volumes:
            - mysql_volume:/var/lib/mysql
        ports:
            - 3306:3306
        environment:
            - MYSQL_ROOT_PASSWORD=docker
            - MYSQL_DATABASE=db
            - MYSQL_USER=user
            - MYSQL_PASSWORD=password
        networks:
            mynet:
        logging:
            driver: gelf
            options:
                gelf-address: "udp://1.2.3.4:12201"
    nginx:
        hostname: nginx
        build:
            context: docker/nginx
            args:
                - WWW_DATA_UID=${WWW_DATA_UID}
        ports:
            - 8080:80
        volumes:
            - ~/.composer:/var/www/.composer:delegated
            - ../project_one:/var/www/app/project_one
            - ../project_two:/var/www/app/project_two
        networks:
            mynet:
                ipv4_address: 172.26.0.6
        logging:
            driver: gelf
            options:
                gelf-address: "udp://1.2.3.4:12201"
    php73:
        hostname: php73
        build:
            context: docker/php/php73-fpm
            args:
                - WWW_DATA_UID=${WWW_DATA_UID}
        volumes:
            - ~/.composer:/var/www/.composer:delegated
            - ../project_one:/var/www/app/project_one
            - ../project_two:/var/www/app/project_two
        extra_hosts:
            - "project-one.local:172.26.0.6"
            - "project-two.local:172.26.0.6"
        networks:
            mynet:
        logging:
            driver: gelf
            options:
                gelf-address: "udp://1.2.3.4:12201"
    php74:
        hostname: php74
        build:
            context: docker/php/php74-fpm
            args:
                - WWW_DATA_UID=${WWW_DATA_UID}
        volumes:
            - ~/.composer:/var/www/.composer:delegated
            - ../project_one:/var/www/app/project_one
            - ../project_two:/var/www/app/project_two
        extra_hosts:
            - "project-one.local:172.26.0.6"
            - "project-two.local:172.26.0.6"
        networks:
            mynet:
        logging:
            driver: gelf
            options:
                gelf-address: "udp://1.2.3.4:12201"

networks:
    mynet:
        driver: bridge
        ipam:
            config:
              - subnet: 172.26.0.0/24

volumes:
    mysql_volume:
```

Sporo powtórzeń prawda? 4 kontenery, a prawie 90 linii kodu. Jednak duża objętość to nie wszystkie minusy. W sytuacji, kiedy będziemy chcieli dodać kolejny projekt, musimy zrobić zmianę w wolumenach oraz hostach. Musimy więc edytować **volume** w trzech kontenerach i **extra_hosts** w dwóch.

## Anchor i Merge
Z pomocą przychodzą nam funkcję **anchor** oraz **merge**. Jak one działają? Zobaczmy to na poniższych przykładach. Lewa kolumna to zapis skrócony, a prawa to wynik co widzi parser:

<table>
<tr>
<td>

```yaml
anchor: &app
key_one: 1
key_two: value

merge: *app
```

</td>
<td>

```yaml
anchor:
    key_one: 1
    key_two: value

merge:
    key_one: 1
    key_two: value
```

</td>
</tr>
</table>

Oba klucze będą miały taką samą treść – **key_one** oraz **key_two**. Możemy łączyć więcej niż 1 element w całość:

<table>
<tr>
<td>

```yaml
anchor_one: &app
  key_one: 1

anchor_two: &app2
  key_two: value

merge:
  <<: [*app, *app2]
```

</td>
<td>

```yaml
anchor_one:
  key_one: 1

anchor_two:
  key_two: value

merge:
  key_one: 1
  key_two: value
```

</td>
</tr>
</table>


Możemy również nadpisywać wartości z kotwic

<table>
<tr>
<td>

```yaml
anchor: &app
  key_one: 1
  key_two: value

merge:
  <<: *app
  key_two: value2
```

</td>
<td>

```yaml
anchor:
  key_one: 1
  key_two: value

merge:
  key_one: 1
  key_two: value2
```

</td>
</tr>
</table>

Połączmy wszystko w całość
Skoro teorię mamy już za sobą, to sprawdźmy jak możemy zmienić przykładowy **docker-compose**, aby nie zawierał zbędnych powtórzeń. Z powtarzających się elementów mamy: **volume**, **network**, **logging** oraz **extra_hosts**.

```yaml
version: '3.7'

x-default_hosts: &hosts
  extra_hosts:
    - "project-one.local:172.26.0.6"
    - "project-two.local:172.26.0.6"

x-default_service: &default_service
  networks:
    mynet:
  logging:
    driver: gelf
    options:
      gelf-address: "udp://1.2.3.4:12201"

x-default_volumes: &appvolumes
  volumes:
    - ~/.composer:/var/www/.composer:delegated
    - ../project_one:/var/www/app/project_one
    - ../project_two:/var/www/app/project_two

services:
  mysql:
    <<: *default_service
    hostname: mysql
    image: mysql:5.7
    volumes:
      - mysql_volume:/var/lib/mysql
    ports:
      - 3306:3306
    environment:
      - MYSQL_ROOT_PASSWORD=docker
      - MYSQL_DATABASE=db
      - MYSQL_USER=user
      - MYSQL_PASSWORD=password
  nginx:
    <<: [*default_service, *appvolumes]
    hostname: nginx
    build:
      context: docker/nginx
      args:
        - WWW_DATA_UID=${WWW_DATA_UID}
    ports:
      - 8080:80
    networks:
      mynet:
        ipv4_address: 172.26.0.6
  php73:
    <<: [*default_service, *appvolumes, *hosts]
    hostname: php73
    build:
      context: docker/php/php73-fpm
      args:
        - WWW_DATA_UID=${WWW_DATA_UID}
  php74:
    <<: [*default_service, *appvolumes, *hosts]
    hostname: php74
    build:
      context: docker/php/php74-fpm
      args:
        - WWW_DATA_UID=${WWW_DATA_UID}

networks:
  mynet:
    driver: bridge
    ipam:
      config:
        - subnet: 172.26.0.0/24

volumes:
  mysql_volume:
```

Z 88 linii zmniejszyliśmy kod do 71, ale nie to jest najważniejsze. Zyskaliśmy czytelność i łatwość zmian poszczególnych fragmentów kodu.

Kod, który tutaj opisałem jest możliwy do osiągnięcia od wersji pliku [3.4](https://docs.docker.com/compose/compose-file/#extension-fields). Jeżeli używasz niższej wersji, możesz również użyć **anchor** oraz **merge**, jednak nie możesz zdefiniować własnych obiektów, ponieważ docker-compose na to nie pozwala. Możesz to obejść stosując kotwice w definicji serwisu i wykorzystać ją w kolejnym:

```yaml
version: '3.7'

services:
  mysql:
    hostname: mysql
    image: mysql:5.7
    volumes:
      - mysql_volume:/var/lib/mysql
    ports:
      - 3306:3306
    environment:
      - MYSQL_ROOT_PASSWORD=docker
      - MYSQL_DATABASE=db
      - MYSQL_USER=user
      - MYSQL_PASSWORD=password
    networks: &appnetwork
      mynet:
    logging: &applogging
      driver: gelf
      options:
        gelf-address: "udp://1.2.3.4:12201"
  nginx:
    hostname: nginx
    build:
      context: docker/nginx
      args:
        - WWW_DATA_UID=${WWW_DATA_UID}
    ports:
      - 8080:80
    logging: *applogging
    volumes: &appvolumes
      - ~/.composer:/var/www/.composer:delegated
      - ../project_one:/var/www/app/project_one
      - ../project_two:/var/www/app/project_two
    networks:
      mynet:
        ipv4_address: 172.26.0.6
  php73:
    hostname: php73
    build:
      context: docker/php/php73-fpm
      args:
        - WWW_DATA_UID=${WWW_DATA_UID}
    volumes: *appvolumes
    logging: *applogging
    networks: *appnetwork
    extra_hosts: &apphosts
      - "project-one.local:172.26.0.6"
      - "project-two.local:172.26.0.6"
  php74:
    hostname: php74
    build:
      context: docker/php/php74-fpm
      args:
        - WWW_DATA_UID=${WWW_DATA_UID}
    volumes: *appvolumes
    logging: *applogging
    networks: *appnetwork
    extra_hosts: *apphosts

networks:
  mynet:
    driver: bridge
    ipam:
      config:
        - subnet: 172.26.0.0/24

volumes:
  mysql_volume:
```

Kodu jest trochę mniej, ale według mnie ucierpiała tutaj czytelność i łatwość dodania nowych funkcji do wszystkich kontenerów na raz.

Daj znać w komentarzu, czy podobał Ci się artykuł. Daj to niesamowitej energii do pisania kolejnych treści.
Sprawdź również mój wcześniejszy artykuł na temat optymalizacji [docker for mac]({{< ref "../2020-06-06" >}} "Docker for Mac – 3 sposoby na przyspieszenie działania").
