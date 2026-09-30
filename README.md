# E016 Fixer

Aplikacja w jednym pliku HTML, która naprawia pliki EPUB wysyłane przez **Send to Kindle** z błędem **E016**. Działa całkowicie offline.

## Problem

Niektóre EPUB-y wysłane na Kindle docierają jako dokument z „original layout”: mają ikonę PDF, nie da się zmienić wielkości tekstu, a na maila przychodzi wiadomość:

> Your document was delivered to your Kindle in its original layout
> […] Get troubleshoot help for file formatting issue (see error E016).

Z testów wynika, że błąd wywołuje połączenie dwóch rzeczy:

1. książka jest w języku, którego Send to Kindle oficjalnie nie obsługuje (np. polski, czeski),
2. element `<dc:language>` w pliku `content.opf` ma dodatkowe atrybuty, np.:

```xml
<dc:language id="language1">pl</dc:language>
<dc:language xsi:type="dcterms:RFC3066">PL</dc:language>
```

Pomaga uproszczenie deklaracji do postaci:

```xml
<dc:language>pl</dc:language>
```

Jednocześnie w `<metadata>` powinny być pełne deklaracje przestrzeni nazw:

```xml
<metadata xmlns:dc="http://purl.org/dc/elements/1.1/" xmlns:opf="http://www.idpf.org/2007/opf">
```

## Użycie

1. Pobierz plik [`e016-fixer.html`](e016-fixer.html) i otwórz go w przeglądarce. Nie trzeba niczego instalować ani mieć dostępu do internetu.
2. Przeciągnij na stronę jeden lub kilka plików EPUB albo kliknij pole, żeby je wybrać.
3. Pobierz poprawione pliki i wyślij je na Kindle.

Pliki są przetwarzane wyłącznie lokalnie, w przeglądarce. Nic nie jest nigdzie wysyłane.

### Opcje

- **Skróć kod języka do głównego podtagu:** np. `pl-PL` → `pl`.
- **Wymuś język:** nadpisuje język zadeklarowany w pliku. Jeśli książka nie ma żadnej deklaracji języka, zostanie dodana ta wartość.

## Co jest zmieniane

W każdym pliku OPF wskazanym w `META-INF/container.xml`:

- `<dc:language>` zostaje uproszczone: bez atrybutów, ze znormalizowaną wielkością liter (`PL` → `pl`), a trzyliterowe kody typu `pol` są zamieniane na dwuliterowe,
- elementy `<meta refines="#…">` i `<link refines="#…">`, które wskazują na usunięte `id`, zostają usunięte, żeby plik pozostał poprawny,
- `<language>` bez prefiksu (np. po edycji w Sigilu) zostaje zamienione na `<dc:language>`,
- do `<metadata>` zostają dopisane brakujące `xmlns:dc` i `xmlns:opf`,
- plik OPF zapisany w UTF-16 zostaje przekodowany do UTF-8.

W archiwum:

- plik `mimetype` jest ustawiany jako pierwszy i nieskompresowany, zgodnie ze specyfikacją OCF,
- pozostałe pliki są kopiowane bajt w bajt, bez ponownej kompresji.

Jeśli w książce nie ma nic do poprawienia, aplikacja to zgłasza i nie proponuje pobierania. Przy każdym pliku widać listę zmian, deklarację języka przed i po oraz ostrzeżenia, np. o potencjalnym DRM albo o pozostałych odwołaniach do usuniętego `id`.

## Walidacja

Zmiany nie wprowadzają nowych błędów EPUBChecka, ale też nie naprawiają błędów, które plik miał wcześniej. Jeśli oryginał przechodził walidację, poprawiony plik również powinien ją przejść.

## Wymagania

Aktualna przeglądarka z obsługą `CompressionStream` / `DecompressionStream` (`deflate-raw`):

- Chrome / Edge 103+
- Firefox 113+
- Safari 16.4+

## Ograniczenia

- Archiwa ZIP64 i zaszyfrowane wpisy ZIP nie są obsługiwane.
- Pliki z DRM nie zostaną naprawione, bo Send to Kindle i tak ich nie przyjmie.
- Alternatywą dla tej aplikacji jest konwersja EPUB → EPUB w programie [Calibre](https://calibre-ebook.com/), która również usuwa problem, ale zmienia w pliku znacznie więcej.
