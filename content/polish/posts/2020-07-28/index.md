---
title: "DDD – Domain Driven Design"
date: 2020-07-28
draft: false
categories: ['DDD']
tags: ['domain driven design', 'DDD', 'Ubiquitous language']
language: pl
description: Domain Driven Design jest to koncept służący do projektowania systemów, który polega na tym, aby oprogramowanie jak najbardziej odzwierciedlało rzeczywisty system lub proces. Kod, który napiszemy, powinien być odzwierciedleniem biznesu, a wszechobecny język (ang. ubiquitous language) użyty w projekcie powinien być spójny i zrozumiały z „biznesem”.
---
Cześć. Jeżeli tutaj trafiłeś, to znaczy, że szukasz informacji na temat DDD – Domain Driven Design. Chciałbym Cię zaprosić do serii artykułów poświęconych metodologii DDD. Postaram się Tobie w zrozumiały sposób przybliżyć czym jest DDD oraz kiedy warto stosować to podejście.

W trakcie całej serii pojawią się przykłady w języku PHP, które dopełnią całość, a Tobie czytelniku, dostarczą praktycznej wiedzy w zakresie wykorzystania **Domain Driven Design**.

## DDD – co to takiego?
Domain Driven Design jest to koncept służący do **projektowania systemów, który polega na tym, aby oprogramowanie jak najbardziej odzwierciedlało rzeczywisty system lub proces**. Kod, który napiszemy, powinien być odzwierciedleniem biznesu, a **wszechobecny język** (ang. ubiquitous language) użyty w projekcie powinien być spójny i zrozumiały z „biznesem”.

Innymi słowy, DDD to zestaw narzędzi, struktur oraz terminologii, których używamy podczas projektowania i implementacji modeli biznesowych. Jedną z najważniejszych założeń konceptu DDD jest głębokie nastawienie na współpracę między osobami z „biznesu”, a zespołem technicznym odpowiedzialnym za aplikację.

Kim są **„osoby z biznesu”**? Są to osoby, często nietechniczne, które mają bardzo dużą wiedzę na temat dziedziny biznesu klienta i procesów jakie tam zachodzą, a jakie aplikacja ma rozwiązać. Takie osoby zwane są **ekspertami domenowymi**. Nie martwcie się, nie są to osoby, które tworzą oprogramowanie. Eksperci domenowi pomagają zrozumieć **architektom i programistom** dziedzinę, wspierając ich wiedzą biznesową.

## Domena
W najprostszych słowach, domena to obszar biznesu, z którym pracujesz i problemy jakie musi rozwiązać. Model, który zostanie zaprojektowany jest rozwiązaniem problemu biznesowego. Domena nie skupia się na technicznych aspektach takich jak na przykład framework, czy baza danych. Jeżeli Twoim zadaniem jest zaprojektowanie aplikacji z ogłoszeniami, Domeną Twojej aplikacji będzie wszystko co jest związane bezpośrednio z ogłoszeniami i ma być zaimplementowane w aplikacji (np. reguły biznesowe, czy problemy).

Jedną z najważniejszych rzeczy w DDD jest wyznaczenie języka wszechobecnego (ang. **Ubiquitous language**), którym będzie podążał model Domenowy.

## Wszechobecny język
**Ubiquitous language** to definicja zrozumiałego języka dla ekspertów domenowych oraz programistów implementujących zaprojektowany system. Zdefiniowany język musi mieć swoje odzwierciedlenie w **Domenie** aplikacji. Wyznaczenie wspólnego, uniwersalnego języka jest kluczem do rozwiązania niejasności, niezrozumienia i barier w komunikacji pomiędzy zespołem programistycznym a biznesem.

Wszechobecny język rozwija się wraz z rosnącym biznesem.

> Domain experts should object to terms or structures that are awkward or inadequate to convey domain understanding; developers should watch for ambiguity or inconsistency that will trip up design.
> 
> -- <cite>Eric Evan</cite>

Mam nadzieję, że wyjaśniłem Ci w zrozumiały sposób czym jest Domain Driven Design. W kolejnych artykułach zagłębimy się w koncept DDD, w którym poznamy nieco przykładów ze świata DDD.
