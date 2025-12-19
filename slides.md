---
theme: default
title: Delegacja, aka Composition over Inheritance
---

# Delegacja

### albo inaczej, _Composition vs (over) Inheritance_ i dlaczego jest _**c o o l**_

---
layout: center
---

Credits: <a href="https://youtube.com/@codeaesthetic?si=hFe86QYcNw-LeQWK" target="_blank">Code Aesthetics</a>

<p>
Wybitny kanał z którego zaczerpnąłem przykład problemu, <br/> który może zostać rozwiązany poprzez wykorzystanie delegacji.
</p>

<iframe width="560" height="315" src="https://www.youtube.com/embed/hxGOiiR9ZKg?si=KDMIQczeKRT6O69o" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---
transition: fade
layout: two-cols
---

# Przykład

Pracujemy nad jakimś systemem, który wymaga niskopoziomowej obsługi obrazków.

<v-click>

- Stworzyliśmy abstrakcyjną klasę `Image`

</v-click>

<v-click>

- Stworzyliśmy klasy `JpegImage` oraz `PngImage` rozszerzające naszą klasę `Image`

</v-click>

<v-click>

- Chcemy pozwolić na zapisywanie obrazków.

</v-click>

<v-click at="4">

Okazuje się jednak, że potrzebujemy dodatkowej funkcjonalności. **Musimy umożliwić rysowanie po obrazku**.

</v-click>

::right::

````md magic-move {at:1}
```java

```

```java
abstract class Image {
    /**
     * Niskopoziomowe operacjie na obrazkach
     * załadowanych do pamięci (np. rysowanie).
     */
}
```

```java
abstract class Image {
    /**
     * Niskopoziomowe operacjie na obrazkach
     * załadowanych do pamięci (np. rysowanie).
     */
}

final class JpegImage extends Image {}
final class PngImage  extends Image {}
```

```java
abstract class Image {
    /**
     * Niskopoziomowe operacjie na obrazkach
     * załadowanych do pamięci (np. rysowanie).
     */

    abstract void save();
    abstract Image load();
}

final class JpegImage extends Image {
    @Override
    void save() {}

    @Override
    Image load() {}
}

final class PngImage  extends Image {
    @Override
    void save() {}

    @Override
    Image load() {}
}
```
````

---
transition: fade
layout: two-cols-header
---

Tworzymy więc nową klasę `DrawableImage`, która rozszerza abstrakcyjną klasę `Image`. _W końcu potrzebujemy wykorzystać niskopoziomowe operacjie pozwalające modyfikować obrazek ..._

<v-click>

**Rozszerzenie klasy `Image` wymusza na nas zaimplementowanie metod `save` i `load`.**

</v-click>

::left::

````md magic-move {at:1}
```java
abstract class Image {
    abstract void save();
    abstract Image load();
}
```

```java
abstract class Image {
    abstract void save();
    abstract Image load();
}
```

```java {0}
abstract class Image {
    abstract void save();
    abstract Image load();
}
```
````

::right::

````md magic-move {at: 1}
```java
final class DrawableImage extends Image {
    /**
     * Logika pozwalająca rysować
     * wykorzystując niskopoziomowe operacje
     * zaimplementowane w abstrakcyjnej
     * klasie Image.
     */
}
```

```java
final class DrawableImage extends Image {
    /**
     * Logika pozwalająca rysować
     * wykorzystując niskopoziomowe operacje
     * zaimplementowane w abstrakcyjnej
     * klasie Image.
     */

    @Override
    void save() {}

    @Override
    Image load() {}
}
```

```java {8-13}
final class DrawableImage extends Image {
    /**
     * Logika pozwalająca rysować
     * wykorzystując niskopoziomowe operacje
     * zaimplementowane w abstrakcyjnej
     * klasie Image.
     */

    @Override
    void save() {}

    @Override
    Image load() {}
}
```
````

<div class="left-bottom">

<v-click>

W taki sposób złamaliśmy jedną z zasad SO<b>L</b>ID.

<span v-mark.underline.red="2">
<b>Liskov substitution principle</b>
</span>

</v-click>

</div>

<style>
.left-bottom {
    position: absolute;
    left: 50px;
    bottom: 50px;
}

.two-cols-header {
  column-gap: 20px; /* Adjust the gap size as needed */
}
</style>

---
transition: fade
layout: two-cols-header
---

Potencjalnie możemy rozwiązać ten problem poprzez wprowadzenie klasy pośredniej `ImageSaver`.

::left::

````md magic-move
```java
abstract class Image {
    abstract void save();
    abstract Image load();
}
```

```java
abstract class Image {}
```
````

<v-click at="1">

````md magic-move
```java
abstract class ImageSaver extends Image {
    abstract void save();
    abstract Image load();
}
```
````

</v-click>

::right::

````md magic-move {at: 1}
```java
final class PngImage extends Image {
    /**
     * Logika pozwalająca rysować
     * wykorzystując niskopoziomowe operacje
     * zaimplementowane w abstrakcyjnej
     * klasie Image.
     */

    @Override
    void save() {}

    @Override
    Image load() {}
}
```

```java
final class PngImage extends ImageSaver {
    /**
     * Logika pozwalająca rysować
     * wykorzystując niskopoziomowe operacje
     * zaimplementowane w abstrakcyjnej
     * klasie Image.
     */

    @Override
    void save() {}

    @Override
    Image load() {}
}
```
````

<div class="left-bottom">

<v-click>

To rozwiązanie jednak prowadzi do potencjalnie <span style="font-size: 28px; font-weight: bold; font-family: 'Comic Sans MS'"> Bardzo Dużego </span> refactoru. W każdym miejscu, gdzie wykorzystaliśmy metody `save` i `load` typu `Image`, musimy dokonać zmiany na typ `ImageSaver`.

</v-click>

</div>

<style>
.left-bottom {
    position: absolute;
    left: 50px;
    bottom: 50px;
}

.two-cols-header {
  column-gap: 20px; /* Adjust the gap size as needed */
}
</style>

---
transition: fade
layout: two-cols
---

## Jak kompozycja rozwiązuje problem?

Zamiast rozszerzać klasy `PngImage` i `JpegImage` o klasę `Image`, możemy **_po prostu użyć_** instancji klasy `Image`.

<v-click at="2">

Klasy `PngImage` i `JpegImage` odpowiadały tylko za obsługę konkretnego formatu pliku. Nie muszą **być** obrazkiem. Kontrakt obsługi zapisu i odczytu obrazków może zostać ujęty przez interface `ImageSaver`.

</v-click>

::right::

````md magic-move {at:1}
```java
abstract class Image {
    /* ... */
    abstract void save();
    abstract Image load();
}

final class JpegImage extends Image {
    @Override
    public void save() {}

    @Override
    public Image load() {}
}

final class PngImage  extends Image {
    @Override
    public void save() {}

    @Override
    public Image load() {}
}
```

```java
abstract class Image {
    /* ... */
}

final class JpegImage {
    public void save(Image image) {}
    public Image load() {}
}

final class PngImage {
    public void save(Image image) {}
    public Image load() {}
}
```

```java
abstract class Image {
    /* ... */
}

interface ImageSaver {
    void save() {}
    Image load() {}
}

final class JpegImage implements ImageSaver {
    public void save(Image image) {}
    public Image load() {}
}

final class PngImage implements ImageSaver {
    public void save(Image image) {}
    public Image load() {}
}
```
````

<style>
.left-bottom {
    position: absolute;
    left: 50px;
    bottom: 50px;
}

.two-cols-header {
  column-gap: 50px; /* Adjust the gap size as needed */
}
</style>

---
transition: fade
---

<div class="center">
    <img src="/orchestra-man.webp" alt="">
    <img src="/orchestra-delegate.webp" alt="">
</div>

<style>
.center {
    display: flex;
    justify-content: center;
    align-items: center;
}
</style>

---
transition: fade
---

# Kiedy używać delegacji?

<v-clicks>

- Chcemy przenieść odpowiedzialność za funkcjonalność na inną klasę bez wykorzystywania dziedziczenia.
- Potrzebujemy zamieniać implementacje klasy do której delegujemy zadania (Dependency Injection).
- Łamiemy Liskov substitution principle.

</v-clicks>

<v-click>

Delegacja oddziela klasę bazową od klasy potomnej. **Klasa bazowa zostaje klasą pomocniczą klasy potomnej**.

</v-click>

---
transition: fade
layout: two-cols
---

### Zalety

<v-clicks>

- Nasza klasa nie udostępnia w publicznym API metod klasy bazowej niezwiązanych z jej funkcjonalnością.
- Zawarcie kontraktu poprzez interface umożliwia wprowadzenie wzorców takich jak Dependency Injection / Strategy.
- Dużo większa swoboda. Klasy są mniej powiązane (decoupled / coupling). Ustalenie kontraktu poprzez interface pozwala na swobodne podmienianie funkcjonalności.

</v-clicks>

::right::

### Potencjalne Wady

<v-clicks>

- Dodatkowa warstwa "niebezpośredniości" (layer of indirection).
- ??

</v-clicks>

---

## Fantastyczne Materiały

1. <a href="https://youtube.com/@codeaesthetic?si=hFe86QYcNw-LeQWK" target="_blank">Code Aesthetics</a> - Kanał z wytłumaczonymi podstawowymi zasadami pisania dobrego kodu na niebanalnych przykładach.
2. <a href="https://www.youtube.com/watch?v=QM1iUe6IofM">_"Object-Oriented Programming is Bad"_</a> - Bardzo dobry film poszerzający horyzonty na temat OOP i "smaku" podczas pisania kodu.
3. <a href="https://refactoring.guru/design-patterns">Design Patterns</a> - Strona z wytłumaczeniem wielu wzorców projektowych.
4. <a href="https://python-patterns.guide/gang-of-four/composition-over-inheritance/">The Composition Over Inheritance Principle</a>

---

<img height="100%" src="/composition.png">

---
transition: fade
layout: center
---

Live Coding
