---
title: Würdevolles und Unwürdevolles Beenden des Windows Explorer
---


In RPA verwendete Applikationen müssen stets ordnungsgemäß initialisiert und beendet werden um einerseits einen stabilen Programmablauf zu gewährleisten als auch anschließende Automationen nicht zu behindern. Grundsätzlich werden in einem RPA-Projekt hierzu eigene Workflows erstellt, welche Initialisierung (Init Application), sanftes Beenden (Close Application) als auch unsanftes Beenden (Kill Application) handhaben.

Sofern in Windows auf Dateiebene gearbeitet und dabei UI Automation verwendet werden muss, stellt sich die Frage wie der Windows Explorer geschlossen werden kann.


## Das Problem
Der Windows Explorer, auch File Explorer genannt, stellt eine graphische Oberfläche (GUI) bereit um auf das Dateisystem zuzugreifen. Mittels Tastenkombination `<Windows> + <E>` kann auf einfache Weise ein Fenster über den Explorer geöffnet werden. Grundsätzlich ist es möglich sehr viele unabhängige Fenster gleichzeitig offen zu haben und Aktionen auszuführen (z.B. ordnerübergreifendes Copy-Paste).

{% include figure image_path="/assets/2020-04-22-rpa-close-and-kill-win-explorer/2020-04-22-rpa-close-and-kill-win-explorer-00_169.png" alt="Windows Explorer: drei gleichzeitig geöffnete Fenster" caption="Windows Explorer: drei gleichzeitig geöffnete Fenster" %}

Einzelne Fenster können in der UI Automation mittels eindeutig definierten Selektoren auseinander gehalten werden. Die Selektoren zu den im obigen Abbild gezeigten Fenstern lauten:
```
<wnd app='explorer.exe' cls='CabinetWClass' title='This PC' />
<wnd app='explorer.exe' cls='CabinetWClass' title='Desktop' />
<wnd app='explorer.exe' cls='CabinetWClass' title='Downloads' />
```

Wenn aber alle Fenster geschlossen werden sollen im Zuge einer Ausnahmebehandlung, ist es wahrlich umständlich den Überblick zu behalten - es müsste der Wert des Attributs 'title' je Fenster gespeichert werden. Dieses Vorgehen wird weiterhin zur Qual wenn die Titel der Fenster im Vorfeld unbekannt sind aufgrund der Nutzung von dynamischen Selektoren.


## Die Lösung
### Der Prozess 'explorer.exe'
Hinter Applikationen stecken Prozesse die für die Ausführung von Aufgaben zuständig sind und demnach ist es üblich diese ausfindig zu machen, unabhängig von der Art und Weise deren Beendigung. Eine schnelle und einfache Möglichkeit den eigentlichen Prozess des Windows Explorer ausfindig zu machen besteht darin den Task Manager zu öffnen und sich die Details der Applikation anzeigen zu lassen.

Der Task Manager lässt sich mit der Tastenkombination `<Strg> + <Shift> + <Esc>` öffnen.
{: .notice}

{% include figure image_path="/assets/2020-04-22-rpa-close-and-kill-win-explorer/2020-04-22-rpa-close-and-kill-win-explorer-01_proctskmng.png" alt="Windows Explorer Prozess im Task Manager" caption="Windows Explorer Prozess im Task Manager" %}

Hinter allen Fenstern des Windows Explorer steckt genau ein Prozess namens 'explorer.exe', dieser ist ebenso Bestandteil der oben angegebenen Selektoren (Attribut 'app'). Mittels des Namens ist es eigentlich möglich den Prozess ordnungsgemäß zu schließen, doch dafür würden in diesem Falle drei separate Prozesse erwartet, einer je Fenster. Wird nun der Prozess 'explorer.exe' über den Task Manager geschlossen, ergibt dies folgendes Resultat.

{% include figure image_path="/assets/2020-04-22-rpa-close-and-kill-win-explorer/2020-04-22-rpa-close-and-kill-win-explorer-02_noxplrdsktp.png" alt="Windows 10 Desktop nach Beendigung des Prozesses 'explorer.exe'" caption="Windows 10 Desktop nach Beendigung des Prozesses 'explorer.exe'" %}

Eine gähnende Leere ist zu betrachten, da alle GUI-Elemente verschwunden sind, sodass nur noch ein Hintergrundbild zu sehen ist. Der Windows Explorer beinhaltet nämlich nicht nur die GUI für das Navigieren im Dateisystem, sondern auch die Desktop Icons, das Startmenü, die Taskbar sowie die Systemsteuerung - zusammenfassend bekannt unter der Windows Shell.

### Entkopplung der Dateisytem-Fenster
Ein erneutes Starten des Prozesses 'explorer.exe' über den Task Manager wäre eine Lösung des Problems, da anschließend alle Fenster des Dateisystems geschlossen sind. Aber ist es möglich die Fenster von der Windows Shell zu trennen, um gezielt nur diese zu Beenden? In Windows gibt es hierzu eine Einstellung, die es uns gestattet die Dateisystem-Fenster in einem separaten Prozess starten zu lassen.

{% include figure image_path="/assets/2020-04-22-rpa-close-and-kill-win-explorer/2020-04-22-rpa-close-and-kill-win-explorer-03_xplrsepproc.png" alt="Einstellung: Dateisystem in einem separaten Prozess ausführen lassen" caption="Einstellung: Dateisystem in einem separaten Prozess ausführen lassen" %}

Ist diese Einstellung gesetzt, so werden alle Fenster in einem separaten Prozess gestartet. Aber Vorsicht ist geboten, denn dieser Prozess heißt ebenso 'explorer.exe'.

### Würdevolles Beenden der Windows Explorer Fenster - eins nach dem anderen?
Aufgrund der Trennung der Fenster von der Windows Shell sollte es nun möglich sein diese eindeutig zu identifizieren und damit zu beenden. Aber auch hier wird uns ein Strich durch die Rechnung gemacht, wie folgende beispielhafte Auflistung der laufenden Prozesse namens 'explorer.exe' zeigt.

|PID |Prozess-Name |Fenster-Titel |
|-|-|-|
|14520 |explorer |Downloads |
|16412 |explorer | |

Die Windows Shell ist nun separat aufgeführt, erkenntlich an dem fehlenden Fenster-Titel, aber dennoch wird nur ein Fenster angezeigt, und zwar das aktuell aktive. Eine Lösung wäre eine Schleife zu bauen, die alle Fenster schließt solange ein Prozess 'explorer.exe' mit einem Fenster-Titel vorhanden ist. Dies resultiert aber in einer (mir persönlich) unschönen Programmierung.

### Unwürdevolles statt Würdevolles Beenden
Grundsätzlich sei gesagt, dass eine ordnungsgemäße Beendigung vorzuziehen ist, aber um alle Fenster in einem Wisch zu beenden erschließt sich aktuell kein anderer Weg als den Prozess für ein Fenster zu killen, wobei auch alle anderen Fenster geschlossen werden. Laut meiner Kenntnis entstehen hierbei auch keine Nebenwirkungen, z.B. den Verlust von Daten (sofern aktuell kein Vorgang läuft). Somit ist es ausreichend alle laufenden Prozesse einzuholen, nach einem Prozess 'explorer.exe' mit einem Fenster-Titel zu suchen und diesen dann zu killen. Nachfolgend ist hierzu der UiPath RPA-Programmausschnitt gezeigt.

{% include figure image_path="/assets/2020-04-22-rpa-close-and-kill-win-explorer/2020-04-22-rpa-close-and-kill-win-explorer-04_rpaimpl.png" alt="UiPath RPA Implementierung zum Beenden aller Windows Fenster" caption="UiPath RPA Implementierung zum Beenden aller Windows Fenster" %}


## Zusammenfassung
Die einzelnen Fenster des Windows Explorer lassen sich nur umständlich greifen und damit wird programmatisch ein ordentliches Beenden erschwert. Abhilfe hierbei bringt ein Kill des richtigen 'explorer'-Prozesses. Hierfür ist aber die Einstellung das Dateisystem in einem separaten Prozess ausführen zu lassen unabdingbar.