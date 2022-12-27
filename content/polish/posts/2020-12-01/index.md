---
title: "Persystencja Value Object w PHP"
date: 2020-12-01
draft: false
categories: ['Doctrine', 'Embeddable', 'value object']
language: pl
---
Cześć. W tym artykule skupimy się na zapisywaniu Value Object’ów w bazie danych. Jeżeli nie wiesz czym jest Obiekt Wartości, to zapraszam Cię do mojego poprzedniego artykułu: [Value Object – Podstawowy element Domain Driven Design]({{< ref "../2020-09-26" >}} "Value Object – Podstawowy element Domain Driven Design"). Tymczasem przejdźmy do meritum.

Na początek należy uzupełnić poprzedni artykuł, iż Value Object należy do Encji i nigdy nie jest zapisywany w bazie danych samemu sobie. Zazwyczaj zapisujemy je poprzez Agregat, o którym napiszę w kolejnym artykule. Istnieje kilka sposobów na zapis obiektu wartości w bazie danych.

W świecie PHP istnieje wiele narzędzi ORM, które pomagają nam w codziennej pracy z bazami danych. Nie mniej jednak, w tym artykule skupię się jedynie na _Doctrine_, ponieważ znam go najlepiej oraz własnej implementacji ORM na podstawie Doctrine DBAL.

## Obiekt wartości we własnym ORM
Jeżeli implementujemy własne narzędzie ORM, musimy sami zadbać o wszystkie szczegóły, które pozwolą nam współpracować z bazami danych. Na przykładzie relacyjnej bazy danych MySQL, musimy utworzyć schemat tabeli Encji, którą będziemy zapisywać. Nasza przykładowa Encja to **Person** zawierająca Imię i Nazwisko jako Value Object, datę urodzenia oraz PESEL jako identyfikator.

```php
<?php declare(strict_types=1);

final class Person
{
    private string $id;
    private Name $name;
    private DateTimeImmutable $birthDate;

    public function __construct(string $id, Name $name, DateTimeImmutable $birthDate)
    {
        $this->id = $id;
        $this->name = $name;
        $this->birthDate = $birthDate;
    }
}

final class Name
{
    private string $first;
    private string $last;

    public function __construct(string $first, string $last)
    {
        $this->first = $first;
        $this->last = $last;
    }

    public function first(): string
    {
        return $this->firstName;
    }

    public function last(): string
    {
        return $this->lastName;
    }
}
```
Dobrą praktyką jest utrzymywanie identyfikatora również jako Value Object, ale dla uproszczenia pominiemy ten aspekt. Obsługę Value Object’u jako identyfikator Encji omówimy w innym artykule.

Na początek musimy utworzyć odpowiedni schemat:

```sql
create table person
(
    id char(11) not null,
    name_first varchar(255) not null,
    name_last varchar(255) not null,
    date_birth date not null
);

create unique index person_id_uindex
    on person (id);
```
Analizując powyższą kwerendę SQL, możesz zauważyć, że Encja posiada 3 pola, a tabela w bazie danych 4. Wynika to z faktu, że nasz Value Object posiada w sobie 2 pola, które zostały oznaczone odpowiednio prefiksem `name_` w tabeli bazy danych. Aby zapisać utworzoną encję w bazie, utwórzmy przykładowe repozytorium, które wykona zapytanie _INSERT_ do bazy.

```php
<?php declare(strict_types=1);

final class PersonDbalRepository extends DbalRepository implements PersonRepository
{
    public function save(Person $person): void
    {
        $sql = 'INSERT INTO person (id, name_first, name_last, birth_date) VALUES (?, ?, ?, ?);';
        $stmt = $this->connection()->prepare($sql);
        $stmt->bindValue(1, $person->getId());
        $stmt->bindValue(2, $person->getName()->first());
        $stmt->bindValue(3, $person->getName()->last());
        $stmt->bindValue(4, $person->getBirthDate()->format('Y-m-d'));
        $stmt->execute();
    }
}
```
To wszystko, nasza encja została zapisana w bazie danych. Teraz tylko pozostało nam odczytać wiersz z bazy danych i zmaapować go na obiekt Encji oraz Value Objectu

```php
<?php declare(strict_types=1);

final class PersonDbalRepository extends DbalRepository implements PersonRepository
{
    // ...

    public function find(string $id): Person
    {
        $sql = 'SELECT * FROM person WHERE id = ?';
        $stmt = $this->connection()->prepare($sql);
        $stmt->bindValue(1, $id);
        $result = $stmt->execute();
        
        // ...

        return new Person(
            $row['id'],
            new Name($row['name_first'], $row['name_last']),
            DateTimeImmutable::createFromFormat('Y-m-d', $row['birth_date'])
        );
    }
}
```
Proste, prawda? Niestety niesie to za sobą kilka konsekwencji. Przy każdym odczycie danych z bazy, wywołany będzie konstruktor obiektów, który bardzo często będzie zawierał logikę walidacji. Spowoduje to wydłużenie czasu tworzenia obiektów. To jednak nie jest jeszcze takie złe. Wyobraźmy sobie, że w konstruktorze Encji którą tworzymy, wysyłamy **zdarzenie domenowe**: _Osoba została utworzona_ (Tak wiem, dziwnie to brzmi :D). Za każdym raziem, gdy będziemy odczytywać obiekt z bazy danych, zdarzenie zostanie wysłane i nie jest to poprawne zachowanie. Dlatego warto korzystać z gotowych i sprawdzonych narzędzi takich jak _Doctrine_, który tworzy nowy obiekt bezwykorzystania konstruktora.

## Value Object w Doctrine
Zapisywanie Obiektu wartości za pomocą _Doctrine_ można wykonać na 3 sposoby. Pierwszym z nich i chyba najłatwiejszym jest ręczne tworzenie **Obiektu Wartości** w metodzie **get** lub odpowiednie przypisanie do pól encji podczas wykonania metody **set** czy w **konstruktorze**.

```php
<?php declare(strict_types=1);

final class Name
{
    private string $first;
    private string $last;

    public function __construct(string $first, string $last)
    {
        $this->first = $first;
        $this->last = $last;
    }

    public function first(): string
    {
        return $this->first;
    }

    public function last(): string
    {
        return $this->last;
    }
}

/** @Entity */
final class Person
{
    /**
     * @Id
     * @Column(name="id", type="string", length=11)
     * @GeneratedValue(strategy="NONE")
     */
    private string $id;
    /** @ORM\Column(type="string") */
    private string $nameFirst;
    /** @ORM\Column(type="string") */
    private string $nameLast;
    /** @ORM\Column(type="string") */
    private DateTimeImmutable $birthDate;

    public function __construct(string $id, Name $name, DateTimeImmutable $birthDate)
    {
        $this->id = $id;
        $this->name_first = $name->first();
        $this->name_last = $name->last();
        $this->birthDate = $birthDate;
    }

    public function getName(): Name
    {
        return new Name($this->name_first, $this->name_last);
    }
}
```
Proste i szybkie rozwiązanie, jednak mamy odrobinę więcej kodu do utrzymania.

###  Embeddables
Drugim sposobem jest wykorzystanie mechanizmu **Embedded**. Na początek wyrzućmy pola `$name_first` oraz `$name_last` na rzecz jednego pola `$name` oraz dodajmy mapping encji. Dla uproszczenia jako adnotacje:

```php
<?php declare(strict_types=1);

/** @Embeddable */
final class Name
{
    /** @ORM\Column(type="string") */
    private string $first;
    /** @ORM\Column(type="string") */
    private string $last;

    public function __construct(string $first, string $last)
    {
        $this->first = $first;
        $this->last = $last;
    }

    public function first(): string
    {
        return $this->first;
    }

    public function last(): string
    {
        return $this->last;
    }
}

/** @Entity */
final class Person
{
    /**
     * @Id
     * @Column(name="id", type="string", length=11)
     * @GeneratedValue(strategy="NONE")
     */
    private string $id;
    /** @Embedded(class="Name") */
    private Name $name;
    /** @ORM\Column(type="string") */
    private DateTimeImmutable $birthDate;

    public function __construct(string $id, Name $name, DateTimeImmutable $birthDate)
    {
        $this->id = $id;
        $this->name = $name;
        $this->birthDate = $birthDate;
    }

    public function getName(): Name
    {
        return $this->name;
    }
}
```
Spójrz proszę na linię 3 oraz 37. W linii 3 oznaczyliśmy naszą klasę jako _Embeddable_, dzięki czemu _Doctrine_ wie, że jest to Value Object, który będzie wykorzystywany w Encji. Tak właśnie zrobiliśmy, w linii 37 oznaczyliśmy pole `$name` adnotację _Embedded_, która mówi, że to pole jest Value Object’em o podanej klasie. _Doctrine_ z tak skonfigurowaną Encją podczas odczytywania, przy wykorzystaniu swojej implementacji _EntityRepository_, utworzy nam poprawny obiekt **Person** z polem `$name` jako **Value Object**.

Proste, prawda? Niestety to rozwiązanie ma swoje minusy. W wersji Doctrine/ORM 2.x pole **_Embedded_** nie może być **null’em**. Prawdopodobnie Embbeded będzie mogło być nullem w wersja 3.x, która na dzień pisania postu jest jeszcze w fazie developmentu.

Jak poradzić sobie z tym problemem? Albo wybieramy sposób pierwszy opisany wyżej, albo wybieramy sposób trzeci. **Spoiler ALERT!** Jednak już na początku muszę Cię uprzedzić, że i to rozwiązanie ma swój minus. Niestety w naszym opisywanym przypadku, niestety nie pozwala on na implementację obiektu **Name** jako **Value Object** w naszej Encji.

Ten sposób to **Własny Typ** (ang. **Custom Mapping Type**). Sama implementacja jest również dość prosta i co ważne, pozwala na użycie opcji **nullable**.

```php
<?php declare(strict_types=1);

use Doctrine\DBAL\Types\Type;
use Doctrine\DBAL\Platforms\AbstractPlatform;

class MyType extends Type
{
    const MYTYPE = 'mytype'; // modify to match your type name

    public function getSQLDeclaration(array $fieldDeclaration, AbstractPlatform $platform)
    {
        // return the SQL used to create your column type. To create a portable column type, use the $platform.
    }

    public function convertToPHPValue($value, AbstractPlatform $platform)
    {
        // This is executed when the value is read from the database. Make your conversions here, optionally using the $platform.
    }

    public function convertToDatabaseValue($value, AbstractPlatform $platform)
    {
        // This is executed when the value is written to the database. Make your conversions here, optionally using the $platform.
    }

    public function getName()
    {
        return self::MYTYPE; // modify to match your constant name
    }
}
```
Tak wygląda szablon implementacji własnego typu w dokumentacji _[Doctrine](https://www.doctrine-project.org/projects/doctrine-orm/en/2.7/cookbook/custom-mapping-types.html)_.

Jak poprawnie zaimplementować custom mapping type? Tym zajmiemy się w kolejnym artykule, w którym opiszemy jak użyć Value Object jako identyfikator. Z tego miejsca zapraszam Cię do śledzenia mojego bloga.
