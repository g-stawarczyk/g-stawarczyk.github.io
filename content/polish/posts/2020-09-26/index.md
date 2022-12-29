---
title: "Value Object – Podstawowy element Domain Driven Design"
date: 2020-09-26
draft: false
categories: ['DDD']
tag: ['architektura', 'DDD', 'value object', 'Doctrine']
language: pl
description: Value Object to najważniejszy element Domain Drive Design. Dowiedz się jakie cechy ma poprawny Value Object i jak stosować go w praktyce.
---
Podczas programowania obiektowego bardzo często wykorzystujemy typy primitywne do przedstawienia jakiejś wartości. Przy tym bardzo często musimy zmierzyć się z przekazywaniem oraz modyfikacją tych wartości. Idąc dalej, chcemy mieć przecież poprawne dane, więc sprawdzamy ich poprawność dodając dodatkową logikę walidującą. A co w sytuacji, gdy daną wartość, która reprezentuje jakiś koncept, chcemy wykorzystać w więcej niż jednym miejscu? Duplikujemy logikę sprawdzania ich poprawności? Wybieramy specjalne miejsce aby utworzyć klasę walidacji, a może duplikujemy kod? Ciężki temat, szczególnie w późniejszym utrzymaniu, ale tutaj z pomocą przychodzi nam **Value Object (obiekt wartości, wartość)**.

## Czym jest Value Object
**Value Object** jest to prosty obiekt, który reprezentuje zmierzoną, wyliczoną lub opisaną wartość. Są to zazwyczaj bardzo małe obiekty takie jak zakres dat, pieniądz czy inne proste wartości jak identyfikator Encji. Jednak co ważne, w odróżnieniu od **Encji**, nie posiadają **tożsamości**.

Czym jest **tożsamość** obiektu? W bardzo dużym skrócie jest to identyfikator, który nadajemy.

Możemy sobie zadać pytanie, czy obiekt wartości w jednym projekcie może być bez problemu zawsze obiektem wartości w innym projekcie? Nie. Bardzo dobrym przykładem jest właśnie reprezentacja Pieniądza. Z jednej strony, dla większości dwa banknoty 10 zł są równe jeżeli ich nominały się zgadzają. Jednak dla Banku Centralnego bardzo ważną informacją jest numer seryjny banknotu. W takim przypadku nie będzie to już Obiekt Wartości a Encja.

W powyższym przykładzie można zauważyć różnicę w porównywaniu Wartości oraz Encji. **Obiekty wartości są równe, jeżeli ich zmierzone, wyliczone lub opisane wartości są równe. Encje zaś porównujemy za pomocą identyfikatorów**.

## Charakterystyka Obiektów Wartości
### Mierzą, liczą lub opisują
Jedna z podstawowych cech Value Object jest opisywanie, wyliczanie czy mierzenie wartości. Właśnie **wartość to słowo klucz**. Value Object nie jest rzeczą w domenie, a właśnie wartością. W podanym wcześniej przykładzie pieniądza, utworzony Value Object nie przedstawia pieniądza jako rzecz, a mierzy ilość określonej waluty. Szukając innych przykładów wartości możemy znaleźć **Imię, Nazwisko lub Adres**. Jednak zawsze musimy pamiętać, że wszystko zależy od kontekstu. W jednym systemie Adres może być Value Objectem, a w innym będzie musiał być Encją.

### Niezmienność i Side-Effect-Free
Niezmienność (ang. Immutability), to kolejna ważna cecha wartości. Nie możemy dopuścić aby Value Object w swoim cyklu życia zmienił swój stan. Również musimy mieć na uwadze fakt, że obiekt ten musi być zawsze w poprawnym stanie.

Jak utworzyć taki obiekt, aby był niezmienny? Jednym z najprostszych sposobów jest możliwość tworzenia obiektów tylko przez konstruktor, a sam obiekt jest pozbawiony metod zmieniających stan.

```php
<?php declare(strict_types=1);

final class ValueObject
{
    private string $value;

    public function __construct(string $value)
    {
        $this->value = $value;
    }

    public function value(): string
    {
        return $this->value;
    }
}
```
W prawdzie można tutaj zarzucić, że nie mamy tutaj w 100% zabezpieczenia przed zmianą stanu, bo przecież w PHP konstruktor może być odpalony kilka razy, a co za tym idzie? Wartość się zmieni

```php
$value = new ValueObject('foo');
var_dump($value->value());
// foo
$value->__construct('bar');
var_dump($value->value());
// bar
```
Umówmy się jednak, kto taki coś robi? Gdyby jednak ktoś chciał się przed tym zabezpieczyć, to jednym z dobrych rozwiązań jest wykorzystanie _metody fabrycznej_. Rozwiązanie jest bardzo proste i wystarczy zmienić publiczny konstruktor na prywatny i dodać statyczną metodę, która zwróci nową instancję obiektu.

```php
<?php declare(strict_types=1);

final class ValueObject
{
    private string $value;

    private function __construct(string $value)
    {
        $this->value = $value;
    }

    public static function fromString(string $value): self
    {
        return new self($value);
    }

    public function value(): string
    {
        return $this->value;
    }
}
```
Teraz próbując utworzyć nowy obiekt za pomocą new otrzymamy ostrzeżenie:

```php
$value = new ValueObject('foo');

Warning: Uncaught Error: Call to private ValueObject::__construct() from invalid context in 
```
Jak wszyscy wiemy jedyną rzeczą stała w naszych systemach jest właśnie zmiana. Jeżeli mamy utrzymać nasz obiekt niezmiennym, jak w takim razie poradzić sobie z operacjami zmieniającymi stan. Tutaj również rozwiązanie jest bardzo proste -> należy utworzyć nową instancję obiektu. Value Object nie może posiadać metod typu set. To wszystko oznacza, że nie zmieniasz wartości istniejącego już obiektu, tylko otrzymujesz nowy obiekt wartości.

```php
<?php declare(strict_types=1);

final class ValueObject
{
    private int $value;

    public function __construct(int $value)
    {
        $this->value = $value;
    }

    public function add(int $value): self
    {
        return new self($this->value + $value);
    }

    public function value(): int
    {
        return $this->value;
    }
}
```
### Porównywalność wartości
Value Objecty możemy jak każdy inny obiekt porównywać ze sobą. Musimy natomiast pamiętać, że równość wartości występuje wtedy, gdy wartości obu obiektów są takie same. Nie ma tutaj znaczenia, że jest to inna instancja klasy, ważne jest jaką wartość ze sobą niesie.

Istnieje kilka sposobów porównywania wartości. Najprostszą, a zarazem najmniej polecaną jest użycie operatora przyrównania `==`. Dwa obiekty będą równe jeżeli ich klasy oraz wartości są identyczne.

```php
$value1 = new ValueObject(10);
$value2 = new ValueObject(10);

var_dump($value1 == $value2);
// true

$value1 = new ValueObject(10);
$value2 = new ValueObject(9);

var_dump($value1 == $value2);
// false
```
Osobiście jednak nie jestem fanem porównania za pomocą `==` i gdy tylko widzę takie porównanie w legacy code, staram się je zmienić na operator identyczności `===`. Operator ten jednak nie nadal się do porównywania obiektów, ponieważ aby obiekty były równe, muszą odnosić się do tego samego obiektu w pamięci.

```php
$value1 = new ValueObject(10);
$value2 = new ValueObject(10);

var_dump($value1 === $value2);
// false

$value1 = new ValueObject(10);
$value2 = $value1;

var_dump($value1 === $value2);
// true
```
Drugą i chyba najbardziej polecaną opcją na porównywanie wartości jest utworzenie dedykowanej metody, która porówna nam obiekty. Metodę taką implementujemy w Value Object i jako argument podajemy obiekt tej samej klasy. Sama metoda w sobie jest bardzo prosta, gdyż za pomocą operatora identyczności porównuje wartość aktualnego obiektu z wartością drugiego obiektu. Ważna uwaga, jeżeli nasz obiekt posiada więcej niż 1 wartość, należy porównać wszystkie wartości.

```php
<?php declare(strict_types=1);

final class ValueObject
{
    private int $value;

    public function __construct(int $value)
    {
        $this->value = $value;
    }

    public function equals(ValueObject $value): bool
    {
        return $this->value === $value->value();
    }

    public function value(): int
    {
        return $this->value;
    }
}
```
Istnieje jeszcze trzecia metoda, o której dobry programista nigdy nie powinien pomyśleć. Tam gdzie chcesz porównać Value Object’y, po prostu porównaj ich wartości if’em w każdym miejscu gdzie tego potrzebujesz. Chyba nie muszę mówić jak to wygląda i czym się ostatecznie zakończy? 🙂

### Koncept Whole Value Pattern
Tak jak wspomniałem we wcześniejszym akapicie, obiekty wartości muszą być zawsze w poprawnym stanie. Stąd wartości powinny spełniać koncept Whole Value Pattern zdefiniowany przez Ward’a Cunningham’a w _[The checks Pattern Language of Information Integrity](http://c2.com/ppr/checks.html)_ w 1994 roku. Aby zapewnić poprawny stan obiektu, podczas jego tworzenia należy sprawdzać wprowadzane wartości. Sprawdzanie to powinno odbywać się wewnątrz Value Object’u. Dlaczego robimy to wewnątrz obiektu? Wyobrażmy sobie sytuację, że nasz obiekt zawiera się w jakimś innym obiekcie, na przykład Encji. Zgodzisz się ze mną, że walidowanie czy Value Object posiada poprawny stan nie jest odpowiedzialnością Encji lub innej klasy. Narusza to przecież zasadę jednej odpowiedzialności (ang. Single Responibility Principle).

## To Value Object or not to Value Object
Mam nadzieję, że po przeczytaniu artykułu również tak jak ja, uważasz, że obiekty wartości są bardzo przydatne w modelowaniu Domeny. Rozwiązują one wiele problemów, przy czym są bardzo łatwe w implementacji oraz utrzymaniu. Dodatkowo w ściśle określonym kontekście każdy ekspert domenowy oraz programista dokładnie wie co oznacza przedstawiana wartość i jakie mamy wobec nich oczekiwania.

Daj znać w komentarzach co sądzisz o Value Object’ach. Jeżeli masz jakieś ciekawe doświadczenia z obiektami wartości, z chęcią wymienię się doświadczeniem w komentarzach.
