---
title: "CQRS - it's simple!"
date: 2021-09-07T23:33:05+02:00
draft: false
categories: ['Development']
image_webp: "images/blog/blog-02.jpg"
cover:
    image: "images/blog/blog-02.jpg"
language: en
---

# CQRS jest prosty
CQRS (**Command Query Responsibility Segregation**) to ewolucja CQS (ang. **Command Query Separation** ).
*CQS* charakteryzuje się rozdzieleniem klasy na metody zapisujące oraz odczytujące.
Metody zapisujące nazywamy **Command**, a odczytujące **Query**.
*Command* zmieniają stan, zmieniają dane, ale nie zwracają żadnych danych.
*Query*, w przeciwieństwie do metod zapisujących, odczytują dane bez ich modyfikacji. Ilekroć wywołamy metodą odczytu, zawsze otrzymamy takie same dane w odpowiedzi.

Jak jest róznica między **CQS** a **CQRS**? 

> CQRS is simply the creation of two objects where there was previously only one.
>
> -- <cite>Greg Young</cite>

Proste prawda?

Sprawdźmy to w takim razie na przykładzie.

<!-- {{< gist g-stawarczyk 08cb5516744d335043ab63f8b76dd26a "01-cqs-example.php" >}} -->

{{< highlight php >}}
<?php

interface SomeClass
{
    public function enable(int $id): void;
    public function modify(int $id, array $data): void;
    public function remove(int $id): void;

    public function get(int $id): SomeObject;
    public function getNewest(): array;
}
{{< /highlight >}}

Co możemy zrobić z powyższym kodem, tak żeby był napisany w duchu CRQS?
Posłuchajmy Grega i zróbmy z jednego obiektu dwa.

{{< highlight php >}}
<?php

interface SomeCommand
{
    public function enable(int $id): void;
    public function modify(int $id, array $data): void;
    public function remove(int $id): void;
}

interface SomeQuery
{
    public function get(int $id): SomeObject;
    public function getNewest(): array;
}
{{< /highlight >}}

### High Five!
![Alt text for my gif](https://media.giphy.com/media/26ufgSwMRqauQWqL6/giphy.gif#center)
