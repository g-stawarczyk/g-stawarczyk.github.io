---
title: "Architektura Heksagonalna"
date: 2021-01-02
draft: false
categories: ['architektura', 'ddd', 'hexagon', 'hexagonal', 'porty i adaptery']
cover:
    image: "images/blog/ports-and-adapters.png"
language: pl
---
Cześć. W dzisiejszym artykule chciałbym poruszyć temat **architektury heksagonalnej** znanej również jako architektura **Portów i Adapterów** (ang. Ports & Adapters). Mam nadzieję, że uda mi się przedstawić Tobie w prosty sposób jak zaimplementować architekturę heksagonalną w Twoim projekcie. Ale zacznijmy od początku.

## Czym jest hexagonal architecture?
Architektura heksagonalna to wzorzec architektoniczny, który pozwala na implementację logiki biznesowej, całkowicie odseparowanej od zależności. Intencją architektury heksagonalnej jest możliwość wykorzystania napisanej aplikacji przez dowolne wejście (np. UI, aplikacja konsolowa lub API),a także testów automatycznych. Wykorzystując porty i adaptery mamy możliwość prostego uruchomienia testów w izolacji od zewnętrznych serwisów lub usług (np. bazy danych).

> Allow an application to equally be driven by users, programs, automated test or batch scripts, and to be developed and tested in isolation from its eventual run-time devices and databases.
>
> -- <cite>Alistair Cockburn</cite>

Z założenia koncept ten pozwala na modularyzację aplikacji, a co za tym idzie, na bezproblemową zmianę komponentów. Pozwala również na łatwą implementację nowego interfejsu użytkownikam, jak na przykład dodanie REST API lub po prostu łatwe testowanie domeny biznesowej z wykorzystaniem Stubów lub Mocków.

Innymi słowy odseparowujemy implementację logiki biznesowej od zewnętrznych zależności.

Dzięki wykorzystaniu konceptu architektury heksagonalnej unikamy przenikania logiki biznesowej do interfejsu użytkownika, a kod jest łatwiejszy do przetestowania.

Na czym polega cała „magia” portów i adapterów? To bardzo proste, wykorzystuje podstawowy element programowania obiektowego – interfejsy oraz regułę odwróconych zależności (ang. **dependency inversion principle**).

Dobrze, poznaliśmy już podstawy architektury, teraz możemy przejść do „mięsa” hexagonu!

## Jak wygląda struktura?

Architektura portów i adapterów bazuje na modelu warstwowym i dzieli się na dwie warstwy – Domeny i Infrastruktury. W przeciwieństwie do architektury warstwowej, w której w zależności od wizualizacji kierunek zależności biegnie z góry na dół lub inaczej z zewnątrz do środka, w architekturze heksagonalnej kierunek zależności biegnie od Domeny na zewnątrz. W architekturze heksagonalnej oprócz warstw wyróżniamy jeszcze porty i adaptery, za pomocą których komunikujemy się między warstwami.

Porty i adaptery z kolei dzielimy wejściowe oraz wyjściowe na każdym z boków umownego heksagonu.

Port rozumiany jest jako interfejs wejściowy do aplikacji. Przykładowo może być to request HTTP, który uruchamia żądanie do naszej aplikacji. Port również rozumiany jest jako dostęp do danych, np port bazy danych. Z drugiej strony mamy adaptery, czyli coś co dopasowuje się do portu

Technicznie port rozumiany jest jako interfejs, a adapter to jego implementacja. To trochę jak z USB, który jest umownym interfejsem i producenci dostosowują do niego swoje urządzenia. W programistycznym świecie mamy przykład portu Repozytorium. Zawiera ono metodę, która powinna zwrócić tablicę obiektów danej klasy. Jak może wyglądać tutaj adapter? Z punktu widzenia domeny nie ma to kompletnie żadnego znaczenia. W zależności od potrzeby możemy wykorzystać tutaj implementację za pomocą natywnych zapytań do bazy danych lub z wykorzystaniem abstrakcji, np. Doctrine. Jednak to nie musi być silnik bazy danych, a na przykład zewnętrzna usługa z interfejsem API. Idąc dalej, może to być jeden z naszych mikroserwisów lub po prostu plik tekstowy.

{{< highlight php >}}
<?php

interface CustomerRepository
{
    /** @return Customer[] */
    public function findAll(): array;
}
{{< /highlight >}}
{{< highlight php >}}
<?php

class CustomerMysqlRepository implements CustomerRepository
{
    /** @return SomeClass[] */
    public function findAll(): array
    {
        $stmt = $this->connection->prepare('SELECT uuid, name FROM user');
        $stmt->exec();
        $result = []
        foreach($row = $stmt->fetch()) {
            $result[] = new Customer($row['uuid'], $row['name']);
        }
        return $result
    }
}
{{< /highlight >}}
{{< highlight php >}}
<?php
class CustomerDoctrineRepository implements CustomerRepository
{
    /** @return Customer[] */
    public function findAll(): array
    {
        return $this->findBy([]);
    }    
}
{{< /highlight >}}
Potrzebujesz wysłać powiadomienie? Nie ma problemu, jedyne co musisz utworzyć to interfejs w warstwie domeny oraz jego implementacje w zewnętrznej warstwie.

Na koniec bardzo ważna rzecz, warstwa domeny nie powinna zależeć od żadnych zewnętrznych zależności.
