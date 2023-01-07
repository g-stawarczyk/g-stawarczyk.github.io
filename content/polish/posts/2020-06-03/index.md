---
title: "Wzorzec potoków i filtrów (pipe & filters)"
date: 2020-06-03
draft: false
categories: ['Wzorce architektoniczne']
tags: ['wzorce', 'wzorzec architektoniczny']
language: pl
slug: wzorzec-potokow-i-filtrow-pipe-filters
---
Cześć.

W moim pierwszym wpisie, chciałbym Wam przestawić wzorzec architektoniczny **pipe & filters**. Jest to sposób na rozbicie dużego zadania na mniejsze części w bardzo przejrzysty sposób. Zaletami takiego rozwiązania jest reużywalność klas (**filtrów**), możliwość szybkiej zmiany kroków wykonywania danego algorytmu oraz zrównoleglenia procesu. Niewątpliwą zaletą jest również testowalność. Dzięki wyodrębnieniu poszczególnych modułów możemy, każdy z kroków możemy testować w izolacji, na przykład testami jednostkowymi.

Jeśli się nie mylę, historia tego rozwiązania sięga pamięcią do 1973 roku, kiedy to jeden z pomysłów Douglasa McIlroya został zaimplementowany w systemach UNIX. Douglas zauważył, że dane wyjściowe jednego programu mogą stanowić dane wejściowe do drugiego programu. W ten sposób powstały potoki, wywołania programów oddzielonych pionową linią „|” (pipe).
Przykład wywołania potoku w systemach UNIX:  
`tail -f /var/log/nginx/access.log | grep 500`

W architekturze aplikacyjnej, przykładowym problemem do rozwiązania przez wzorzec **pipe & filters** może być obróbka pliku. Załóżmy, że mamy spakowany plik **CSV**, z którego musimy wyciągnąć jakieś dane, a następnie na podstawie wybranych danych wykonać zadanie. Na pierwszy rzut oka możemy zauważyć 3 pod zadania: rozpakowanie pliku, przetworzenie pliku CSV oraz wykonanie zadania.

Rozbicie całego procesu na tak małe zadania pozwala nam na użycie ich w innych **pipeline’ach** lub po prostu wykorzystanie w dowolnej klasie, które będzie potrzebowała danej funkcji.

Implementacja wzorca jest bardzo prosta. Klasa **Pipeline** musi przyjąć kolejne kroki algorytmu, które następnie musi wykonać. Można to zrobić za jednym razem lub zbudować przy pomocy metody. Przykładowa implementacja wzorca w języku PHP przedstawia się następująco:

{{< highlight php >}}
<?php

class Pipeline implements PipelineInterface
{
    /** @var callable[] */
    private $stages = [];

    public function __construct(callable ...$stages)
    {
        $this->stages = $stages;
    }

    public function pipe(callable $stage): PipelineInterface
    {
        $pipeline = clone $this;
        $pipeline->stages[] = $stage;

        return $pipeline;
    }

    public function process($payload)
    {
        foreach ($this->stages as $stage) {
            $payload = $stage($payload);
        }

        return $payload;
    }
}
{{< /highlight >}}
Samo wykorzystanie również jest trywialne. Musimy utworzyć nową instancję potoku, przekazać filtry (zadania) w odpowiedniej kolejności i uruchomić proces:

{{< highlight php >}}
<?php

class Foo
{
    public function __invoke(): SomeClass
    {
        $data = ...;

        return (new Pipeline)
            ->pipe(new FilterA)
            ->pipe(new FilterB)
            ->pipe(new FilterC)
            ->process(new SomeClass($data));
    }
}
{{< /highlight >}}
{{< highlight php >}}
<?php

class FilterA
{
    public method __invoke(SomeClass $obj): SomeClass
    {
        // Do the magic here :).

        return $obj;
    }
}
{{< /highlight >}}
Prawda, że proste?

Dajcie znać co myślicie o wzorcu **pipe & filters** oraz jakie widzicie jego zastosowanie
