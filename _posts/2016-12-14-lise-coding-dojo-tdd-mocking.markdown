---
layout: post
title:  "Test Driven Development und Mocking Dojo"
date:   2016-12-14 16:00:00 +0100
categories: dojo tdd mocking 
tags: test github nunit fluentassertions mock moq csharp
---
Am Freitag, den 2.12.2016, wurde bei der [lise][lise] das Developer Meeting in Form eines Coding Dojos veranstaltet. Im Dojo wurden gleich zwei Themen rund um das Testen von Software behandelt. Zum Einen wurde das Test Driven Development als Methode geübt, zum Anderen das isolierte Testen einer Komponente durch Mocking vermittelt. Die eigens dafür erstellten Übungsaufgaben stehen frei [zur Verfügung][GitHub] und können selbstständig durchgeführt werden. Dieser Blogpost stellt das Dojo und die Übungen vor.
<!--more-->

## Coding Dojo

![Photo from the coding Dojo]({{ site.url }}/assets/lise-coding-dojo.jpg)

Dojo? Dojo ist eine Übungshalle in den japanischen Kampfkünsten, in denen Katas (Übungen) durchgeführt werden. Als Dojo wird auch die Gemeinschaft der Teilnehmer bezeichnet. In einem Dojo gilt folgendes Mindset:
* Sicherer Platz außerhalb der Arbeit
* Wir sind hier um zu lernen
* Bedürfnis nach Entschleunigung, keine Hektik
* Fokus auf "Mache es richtig!"
* Gemeinschaftliches Spiel

Auf Basis dieses Mindsets wurden somit die beiden Aufgaben angegangen.

## Test Driven Development (Aufgabe 1)

Für das Kennenlernen von Test Driven Development haben wir eine Aufgabe vorbereitet, in der es darum geht, für einen Shop anhand von unterschiedlichen Währungen der Einkäufer Preisaufschläge zu berechnen. Hier liegen konkrete fachliche Anforderungen vor, die in Code übersetzt werden sollen. Dabei gibt es verschiedene Fälle zu betrachten, der Normalfall - Verkauf in Euro-Währung - verursacht keine Aufschläge für den Kunden. Ansonsten wird grundsätzlich für Nicht-Euro-Länder ein Aufschlag von 7%, jedoch mindestens 10 € berechnet. Zusätzlich gibt es ein paar Sonderfälle, die ebenfalls berücksichtigt werden müssen. Jeder dieser Fälle beschreibt eine konkrete Anforderung, die getestet werden kann.

### Wie funktioniert Test Driven Development?

Code wird immer auf irgendeine Art getestet. Häufig nachdem der produktive Code geschrieben wurde und im Zweifel lediglich manuell. Nach dem Prinzip "Test First" wird dieses Vorgehen umgekehrt. Es wird zuerst der automatisierte Test geschrieben. [Test Driven Development][TDD] sorgt dabei dafür, dass der produktive Code anhand des Test-Codes formuliert wird, der Test also den Produktiv-Code vorantreibt.
Test Driven Development umfasst dabei drei Zyklen, die unter "Red-Green-Refactor" bekannt sind.

![Diagram Red Green Refactor]({{ site.url }}/assets/red-green-refactor.png) 

1. Red: Schreibe einen fehlschlagenden Test. Ein fehlschlagender Test wird soweit vorbereitet, dass er kompiliert, aber fehlschlägt. Er verwendet also Methoden im Produktiv-Code, die zwar erzeugt werden, jedoch noch keinen Inhalt aufweisen.
2. Green: Schreibe so viel Produktiv-Code, bis der Test erfolgreich ist, also mit minimalstem Aufwand ([YAGNI][YAGNI]). Vorherige Tests bleiben dabei ebenfalls erfolgreich.
3. Refactor: Räume den Code auf, die Tests bleiben dabei immer grün. Sie bieten ein Sicherheitsnetz, dass durch Aufräumarbeiten keine Funktionalität verloren geht.

### Testen mit NUnit und FluentAssertions

Zur Formulierung von Tests liegen für die Sprache C# unterschiedliche Test-Treiber vor. Einer davon ist [NUnit][NUnit], der in unserem Beispiel verwendet wird. Zur Erstellung eines Tests müssen sowohl eine Klasse als Testklasse mittels `[TestFixture]` annotiert werden, als auch jede einzelne Test-Methode mit `[Test]`. Der typische Aufbau eines Tests berücksichtigt folgende Struktur: Arrange, Act, Assert. Zunächst wird das Test-Szenario aufgebaut (Arrange), dann wird die zu testende Funktionalität aufgerufen (Act), zuletzt wird das Ergebnis des Tests gegenüber dem erwarteten Ergebnis verglichen (Assert). Zur Formulierung der Assertions ist die Bibliothek [FluentAssertions][FluentAssertions] eingebunden, die die Überprüfung leserlicher formuliert. Die Dokumentationen beider Bibliotheken geben Aufschluss darüber, wie diese einzusetzen sind. Desweiteren liegen im bereitgestellten Code Muster-Lösungen vor (siehe unten).

## Mocking (Aufgabe 2)

Im zweiten Teil des Dojos ging es darum, für den entwickelten PriceCalculator zusätzlich die Möglichkeit zu schaffen, die Preise in die Zielwährung zu konvertieren. Da die Wechselkurse sich dauerhaft ändern und nicht selbstständig berechnet werden können, bedienen wir uns bei einem Web-Service, der die benötigten Wechselkurse bereitstellt.

### Problemstellung bei Abhängigkeiten

Warum ist es ein Problem, Tests mit einer Abhängigkeit (Dependency) zu implementieren? Mit Abhängigkeiten wird kollaboriert. Problematisch ist es dann, wenn die Abhängigkeiten für Unbequemlichkeit in der Kollaboration sorgen. Zwei Beispiele für Unbequemlichkeiten sind Zeit und Zufälle.
Beim Unit Testing wollen wir lange Testlaufzeiten vermeiden, das Aufrufen eines Webservice dauert aber länger als uns lieb ist. Und was wäre, wenn der Web-Service gar nicht verfügbar ist? Außerdem, wie formuliere ich einen zuverlässigen Test (habe ich den richtigen Wechselkurs abgefragt?), wenn die Wechselkurse für Währungen sich ständig ändern? Letztlich haben wir auch keinen Einfluss auf die Funktionalität des Web-Services, unsere Tests sollen dagegen sicherstellen, dass unser Code richtig funktioniert und nicht der Web-Service, denn wir schreiben einen Unit-Test. Wenn es also schmerzt, einen sauberen Test zu formulieren, dann liegen Unbequemlichkeiten vor, die beseitigt werden wollen.

### Abhängigkeiten durch Mock ersetzen

Ein Mock ist eine Art Fake einer Abhängigkeit. Neben Mocks gibt es weitere Arten von Fakes. Ein Mock zeichnet sich dadurch aus, dass er nicht nur Aufrufe dokumentiert und vorgegebene Antworten liefert, sondern außerdem das Überprüfen von Behauptungen (Assertions) ermöglicht.
Durch Mocking können wir uns also der Unbequemlichkeiten im Test-Szenario entledigen und zuverlässige Tests schreiben.

![Diagram Mock Dependency]({{ site.url }}/assets/mock-diagram.png)
 
Die vorhandene Abhängigkeit wird im Test durch einen Mock (Fake) ersetzt, während im Produktionsbetrieb die tatsächliche Abhängigkeit verwendet wird. Siehe dazu auch das Diagramm, dass die mögliche Implementierung anhand eines übergeordneten Interfaces darstellt. Hätten wir keinen Mock, sondern die tatsächliche Implementierung, könnten wir nur sehr unscharf prüfen, ob der Wechselkurs stimmt: Er soll positiv sein - es gibt keine negativen Wechselkurse - und nicht 1, denn wir erwarten, dass zwei unterschiedliche Währungen nicht den gleichen Wert besitzen. Aber ist nicht durchaus irgendwann mal möglich, dass der Schweizer Franken den gleichen Kurs besitzt wie der Euro? Dann würde unser Test fehlschlagen.
In dieser Aufgabe verwenden wir die Mocking-Library [Moq][Moq]. Mit einem Mock können wir einerseits die Rückgabewerte bestimmen und andererseits den Aufruf der richtigen Methoden im Mock überprüfen. Wir können also bezogen auf unsere Übung einen definierten Wechselkurs zurückgeben (ihn faken) und überprüfen, ob die Abfrage des Wechselkurses mit den richtigen Parametern geschieht und der definierte Wechselkurs Anwendung findet.

### Tipps beim Testen mit unbequemen Abhängigkeiten

Gerade bei der Arbeit mit Abhängigkeiten macht es sich bezahlt, insbesondere das ["Dependency Inversion Principle"][DIP] (eines der [SOLID][SOLID]-Prinzipen) zu berücksichtigen, damit wir testbaren Code auch für das Mocking erreichen. Das Prinzip besagt, dass Abhängigkeiten möglichst nicht in der Klasse selbst erzeugt werden ("new is glue"), sondern stattdessen von außen in die Klasse gegeben werden. Dies geschieht gängigerweise z.B. über eine Constructor-Injection. Das bedeutet, die Abhängigkeit wird dem Konstruktor übergeben und nicht im Konstruktor selbst ein neues Objekt der Abhängigkeit erzeugt.
Außerdem ist es wertvoll, das ["Single Responsibility Principle"][SRP] (ebenfalls ein [SOLID][SOLID]-Prinzip) zu beherzigen, denn eine Klasse, die nur eine Aufgabe erledigt, braucht naturgemäß weniger Abhängigkeiten als eine Klasse, die eine Fülle an unterschiedlichsten Aufgaben durchführt. Dies erhöht die Kohäsion der Klasse.
Das Programmieren gegen Interfaces (wie im obigen Diagramm angedeutet) erleichtert zudem, eine Implementierung einer Abhängigkeit gegen eine andere auszutauschen und so für losere Kopplung zu sorgen.

## Erkenntnisse

Test Driven Development hilft dabei, überhaupt erst testbaren Code zu schreiben. Die Anwendung von TDD ist im konkreten Fall zu bewerten. Eine vollkommene Testabdeckung ist nicht zu erreichen und auch überhaupt nicht notwendig. Jedoch hilft jeder automatisierte Test, egal ob auf dem Wege von TDD oder anderweitig geschrieben, spätere Regressionstests durchzuführen, was ohne Automatisierung in den meisten Projekten vernachlässigt wird.
Mocks helfen dabei, Unit-Tests zu isolieren, zu schärfen und ihre Ausführungsdauer zu verkürzen. Die [Testpyramide][Testpyramide] besagt ein breites Fundament an Unit-Tests, in denen typischerweise Mocks zum Einsatz kommen können. In darüber liegenden Schichten (Integrations- oder Systemtests) findet Mocking jedoch selten Anwendung, da hier bewusst das Zusammenspiel der Komponenten geprüft wird.

## Wie führe ich die Übung selber durch?

Die Aufgabenbeschreibung und der dafür notwendige Code ist auf [GitHub][GitHub] verfügbar. Alles Nötige für die Durchführung ist hier erklärt. Außerdem liegen zwei Beispiel-Lösungen von meinem Kollegen [Martin Dieblich][MartinSolution] und [mir][SteveSolution] als eigene Branches im Repository vor. Viel Spaß beim Üben!

*Dieser Blogpost wurde ebenfalls auf [Softwareentwicklung Köln Blog][se-koeln-blog] veröffentlicht.*

[TDD]: https://en.wikipedia.org/wiki/Test-driven_development
[YAGNI]: https://de.wikipedia.org/wiki/YAGNI
[NUnit]: https://github.com/nunit/docs/wiki
[FluentAssertions]: http://www.fluentassertions.com/
[Moq]: https://github.com/Moq/moq4/wiki/Quickstart
[DIP]: https://en.wikipedia.org/wiki/Dependency_inversion_principle
[SOLID]: https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)
[SRP]: https://en.wikipedia.org/wiki/Single_responsibility_principle
[Testpyramide]: https://watirmelon.blog/2012/01/31/introducing-the-software-testing-ice-cream-cone/
[GitHub]: https://github.com/skorzinetzki/tdd-mocking-dojo
[SteveSolution]: https://github.com/skorzinetzki/tdd-mocking-dojo/tree/steve_develop
[MartinSolution]: https://github.com/skorzinetzki/tdd-mocking-dojo/tree/martin_develop
[lise]: https://www.lise.de
[se-koeln-blog]: http://www.softwareentwicklung-koeln.de/test-driven-development-und-mocking-dojo/