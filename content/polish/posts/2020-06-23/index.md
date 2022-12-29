---
title: "Composer 2.0 – sprawdźmy jak działa nowy menadżer paczek"
date: 2020-06-23
draft: false
categories: ['Narzędzia']
tag: ['composer']
language: pl
description: Composer 2.0 wprowadził równoległe instalowanie bibliotek. Sprawdźmy jak wydajnie działa nowa wersja menadżera w porównaniu do wersji 1.x.
slug: composer-2-0-sprawdzmy-jak-dziala-nowy-menadzer-paczek
---
Cześć. W dzisiejszym wpisie sprawdzimy jak działa najnowsza wersja menadżera zależności **composer** w wersji **2.0.0-alpha1**. Porównamy jak wygląda działanie, czas wykonywania zadania oraz zużycie pamięci RAM podczas operacji instalowania, aktualizowania oraz dodawania nowej biblioteki.

Projekt, na którym będziemy testować menadżera zbudowany jest z wykorzystaniem Symfony 4.2 i wymaga 36 bibliotek – tyle pozycji wpisanych jest w pliku **composer.json**. Po uwzględnieniu zależności wszystkich bibliotek mamy 128 paczek.

## Szybsze pobieranie

Najbardziej wyczekiwaną i zarazem najbardziej zauważalną zmianą w drugiej wersji composera jest zmieniony workflow instalowania, dodawania, aktualizacji czy usuwania paczek, co ma znaczący wpływ na wydajność. W wersji 1.x menadżer pobiera i instaluje paczka po paczce, a w wersji 2.x menadżer najpierw pobiera paczki oraz ich meta dane a następnie po zakończeniu ściągania instaluje je. Wszystko to dzieje się **równolegle**. Dzięki takiemu zabiegowi całość działa o wiele szybciej.
Zobaczmy poniższe logi, jak wyglądała proces instalowania:

composer 1.10.7
```zsh
www-data@grzegorz-php74:~/project$ composer install --no-cache --no-scripts --profile
[9.2MiB/0.02s] Loading composer repositories with package information
[9.6MiB/0.02s] Installing dependencies (including require-dev) from lock file
[10.5MiB/0.05s] Package operations: 128 installs, 0 updates, 0 removals
[10.5MiB/0.05s]   - Installing ocramius/package-versions (1.8.0): [10.6MiB/0.05s] [10.6MiB/0.72s] Downloading (0%)
[10.6MiB/0.72s] Downloading (5%)
[10.6MiB/0.72s] Downloading (10%)
...
[11.9MiB/59.44s]   - Installing symfony/profiler-pack (v1.0.4): [11.9MiB/59.44s] [11.9MiB/59.73s] Downloading (100%)[11.9MiB/59.73s]
[11.2MiB/60.07s] Package zendframework/zend-eventmanager is abandoned, you should avoid using it. Use laminas/laminas-eventmanager instead.
[11.2MiB/60.07s] Package zendframework/zend-code is abandoned, you should avoid using it. Use laminas/laminas-code instead.
[11.2MiB/60.07s] Package symfony/lts is abandoned, you should avoid using it. Use symfony/flex instead.
[11.2MiB/60.07s] Package phpunit/phpunit-mock-objects is abandoned, you should avoid using it. No replacement was suggested.
[11.2MiB/60.07s] Generating autoload files
[11.5MiB/60.12s] 5 packages you are using are looking for funding.
[11.5MiB/60.12s] Use the `composer fund` command to find out more!
[11.4MiB/60.12s] Memory usage: 11.41MiB (peak: 12.86MiB), time: 60.12s
```
composer 2.0-alpha1
```zsh
www-data@grzegorz-php74:~/project$ composer install --no-cache --no-scripts --profile
[8.9MiB/0.01s] Installing dependencies from lock file (including require-dev)
[9.9MiB/0.02s] Verifying lock file contents can be installed on current platform.
[11.5MiB/0.05s] Package operations: 128 installs, 0 updates, 0 removals
[11.6MiB/0.05s]   - Downloading ocramius/package-versions (1.8.0)
...
[14.3MiB/0.10s]   - Downloading symfony/validator (v4.2.8)
[13.6MiB/12.92s]   - Installing ocramius/package-versions (1.8.0): Extracting archive
...
[18.0MiB/17.99s]   - Installing symfony/validator (v4.2.8): Extracting archive
[16.3MiB/22.92s] Package symfony/lts is abandoned, you should avoid using it. Use symfony/flex instead.
[16.3MiB/22.92s] Package symfony/lts is abandoned, you should avoid using it. Use symfony/flex instead.
[16.3MiB/22.92s] Package zendframework/zend-code is abandoned, you should avoid using it. Use laminas/laminas-code instead.
[16.3MiB/22.92s] Package zendframework/zend-eventmanager is abandoned, you should avoid using it. Use laminas/laminas-eventmanager instead.
[16.3MiB/22.92s] Package phpunit/phpunit-mock-objects is abandoned, you should avoid using it. No replacement was suggested.
[16.3MiB/22.92s] Generating autoload files
[16.7MiB/22.98s] 5 packages you are using are looking for funding.
[16.7MiB/22.98s] Use the `composer fund` command to find out more!
[16.7MiB/22.99s] Memory usage: 16.68MiB (peak: 18.89MiB), time: 22.99s
```
## Test **composer install**
Przejdźmy zatem do pierwszego testu – composer install. Testy przeprowadziłem w kontenerze docker z PHP7.4 i [synchronizacją mutagen]({{< ref "../2020-06-06#3-wersja-edge--mutagenio" >}} "Docker for Mac – 3 sposoby na przyspieszenie działania") przy użyciu pamięci podręcznej oraz bez niej. Każdy test wykonany został pięciokrotnie, a wyniki zostały uśrednione. Poniżej możemy zobaczyć wykres porównujący czasy instalacji bibliotek testowego projektu z wykorzystaniem composera w wersji 1.10.7 oraz 2.0-alpha1.

{{< figure src="image/image-1.png" >}}
{{< figure src="image/image-2.png" >}}

<p><div style="width:100%;height:0;padding-bottom:80%;position:relative;"><iframe src="https://giphy.com/embed/5VKbvrjxpVJCM" width="100%" height="100%" style="position:absolute" frameBorder="0" class="giphy-embed" allowFullScreen></iframe></div></p>

O ile czas z wykorzystaniem cache nie robi wrażenia, o tyle instalacja bez wykorzystania pamięci podręcznej już tak. Mamy tutaj prawie trzy krotne zmniejszenie czasu całej operacji! Jest też mała uwaga, zużycie pamięci wzrosło o około 35%, ale jest to nadal małe zapotrzebowanie pamięci.

## Test **composer update** oraz **composer require**
Czas na aktualizacje biblioteki. W tej operacji, composer musi sprawdzić całe drzewo zależności, co w wersji pierwszej narzędzia zajmuje bardzo dużo czasu oraz pamięci. Biblioteka jest w najnowszej wersji, więc sprawdzony został jedynie czas przygotowania do pobierania. Samo ściąganie paczek odbywa się według flow opisanego w **composer install**.  
Sprawdziłem również jak wygląda proces dodawania nowej biblioteki do projektu.  
Tak jak w pierwszym teście, każdy przypadek sprawdziłem 5 razy, a wyniki zostały uśrednione.

{{< figure src="image/image-3.png" >}}
{{< figure src="image/image-4.png" >}}

Wydajność drugiej wersji menadżera zależności robi na prawdę duże wrażenie. Czasy zmniejszyły się kilkukrotnie, a zużycie pamięci – no cóż, wykres mówi wszystko.

## Uruchomienie jako root wymaga potwierdzenia
Od wersji 2.0 composer wymaga potwierdzenia, jeżeli zostanie uruchomiony z konta roota. Jako, że biblioteka może za pomocą composera wykonać dowolny kod na maszynie użytkownika, jest to dobry krok w stronę bezpieczeństwa.

## Dry-run
Kolejną nowością jest opcja –dry-run dla operacji usuwania (remove) oraz dodawania (require). Czy będzie mocno używana? Nie wiem, ale gdyby ktoś chciał to jest 😀

## Nowy format metadanych repozytoriów
Wraz z wydaniem wersji drugiej menadżera, dodany został nowy format metadanych repozytoriów. Pierwsza wersja menadżera zależności pobierała ogromy plik JSON. Najnowszy format to zestaw małych plików JSON, które composer może pobrać i zapisać w cache.

Aby sprawdzić, czy dane repozytorium wspiera nowy format, wystarczy, że poszukamy klucza `metadata-url` w pliku packages.json. Gdy tylko omawiany format jest dostępny, **composer 2** będzie korzystał z nowych danych, a wersja pierwsza nadal będzie korzystać tylko ze starego formatu.

Najpopularniejsze repozytorium paczek `Packagist.org` wspiera już [nowy format](https://repo.packagist.org/packages.json).
