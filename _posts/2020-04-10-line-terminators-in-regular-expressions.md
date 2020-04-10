---
title: Zeilenumbrüche in regulären Ausdrücken
---

Eine Regular Expression, ins Deutsche übersetzt 'regulärer Ausdruck', hilft dabei eine Zeichenkette zu filtern. Programmatisch ist es so zu verstehen, dass eine Zeichenkette mit einem bestimmten Muster abgeglichen wird und der übereinstimmende Teil der Zeichenkette zurückgegeben wird.

**Beispiel:**
Eine Zeichenkette lautet `Heute ist Freitag der 10.04.2020. Die Sonne scheint.` Um nach dem Datum zu filtern, wird die Zeichenkette mit dem regulären Ausdruck `\d\d\.\d\d\.\d\d\d\d` abgeglichen und erhält als Rückgabe '20.04.2020'.
{: .notice}

## Das Problem
Die Aufgabe bestand darin den Text einer E-Mail nach IDs zu durchsuchen und es wurde festgelegt, dass pro Zeile genau eine ID vorkommt. Des Weiteren beginnt eine ID immer mit einer Zahl und kann zwischen 9 und 20 alphanumerische Zeichen enthalten - damit wäre `311A593BC2` eine ID und `A4036K4V` wäre keine ID.
"Ein einfaches Muster" dachte ich mir und began sogleich den regulären Ausdruck zu erstellen.

Der initiale Ausdruck lautete `(\d\w{8,19})` und ein Trockentest führte zum Erfolg. Doch als das Muster auf einen E-Mail-Text angewandt wurde, begannen die Probleme. Der Text hatte ja noch mehr Zeichen, bedingt durch Signatur und Disclaimer. Außerdem sind durch Copy-Paste der IDs auch Leerzeichen am Anfang und Ende möglich.

"Dann also in die zweite Runde" dachte ich mir und fügte dem initialen Ausdruck noch hinzu: die Möglichkeit von Leerzeichen, zusammenhängende alphanumerische Zeichen je Zeile. Der erweiterte Ausdruck lautete somit `^\ *(\d\w{8,19})\ *$`, doch das Ergebnis eines Tests war anders als erwartet - es wurden genau 0 (Null) Treffer gefunden.

## Die Lösung
Anfangs konnte die Ursache nicht genau identifiziert werden, da ein Test via <a href='https://regex101.com/r/1OAiXd/2/' target='_blank'>regular expressions 101</a> genau die erwarteten Ergebnisse ausgab. Doch dann wurde ich aufmerksam auf die Behandlung von Zeilenumrüchen (Line Terminators) in regulären Ausdrücken.

Grundsätzlich sind zwei Steuerzeichen für einen Zeilenumruch reserviert, aber je nach Betriebssystem wird dieser durch eine andere Escape-Sequenz dargestellt.

|Betriebssystem |Steuerzeichen |Escape-Sequenz |
|-|-|-|
|Unix / Linux |LF |\n |
|Windows |CR LF |\r\n |
|Mac OS |CR |\r |

Verdeutlichen soll dies folgendes Beispiel:

Eine Zeichenkette lautet
```
Heute ist Freitag der 10.04.2020.
Die Sonne scheint.
```
Wird dieser Text in einem Programm eingelesen, so sieht er folgendermaßen aus
```
Heute ist Freitag der 10.04.2020.\r\nDie Sonne scheint.
```

Der reguläre Ausdruck zur Filterung der IDs sollte die Zeilenumrüche doch schon erkennen aufgrund von `$`, oder nicht?

Tatsächlich ist dem nicht so, denn laut <a href='https://docs.microsoft.com/en-us/dotnet/standard/base-types/anchors-in-regular-expressions#end-of-string-or-line-' target='_blank'>Microsoft's Dokumentation</a> wird damit eine Übereinstimmung zu `\n` erkannt, aber `\r\n` bleibt außen vor. Die Lösung wird in der Dokumentation sogleich mitgegeben: Zusätzlich muss `\r?` mit in den regulären Ausdruck aufgenommen werden.

Der finale reguläre Ausdruck zur Findung der IDs aus dem E-Mail-Text lautete demnach `^\ *(\d[0-9a-zA-Z]{8,19})\ *\r?$`. Es sei darauf hingewiesen, dass `\w` ersetzt wurde mit `[0-9a-zA-Z]`, da sonst auch IDs mit `_` (z.B. `311A593_BC2`) zulässig wären.