---
title: "Sanftes" und "Unsanftes" Beenden des Windows Explorer
---


In RPA verwendete Applikationen m&uuml;ssen stets ordnungsgem&auml;&szlig; initialisiert und beendet werden um einerseits einen stabilen Programmablauf zu gew&auml;hrleisten als auch anschließende Automationen nicht zu behindern. Grunds&auml;tzlich werden in einem RPA-Projekt hierzu eigene Workflows erstellt, welche Initialisierung (Init Application), sanftes Beenden (Close Application) als auch unsanftes Beenden (Kill Application) handhaben.

Sofern in Windows auf Dateiebene gearbeitet und dabei UI Automation verwendet werden muss, stellt sich die Frage wie der Windows Explorer geschlossen werden kann.


## Das Problem
Der Windows Explorer, auch File Explorer genannt, stellt eine graphische Oberfl&auml;che (GUI) bereit um auf das Dateisystem zuzugreifen. Mittels Tastenkombination <Windows> + <E> kann auf einfache Weise ein Fenster &uuml;ber den Explorer ge&ouml;ffnet werden. Grunds&auml;tzlich ist es m&ouml;glich sehr viele unabh&auml;ngige Fenster gleichzeitig offen zu haben und Aktionen auszuf&uuml;hren (z.B. ordner&uuml;bergreifendes Copy-Paste).

{% include figure image_path="/assets/2020-04-22-rpa-close-and-kill-win-explorer/2020-04-22-rpa-close-and-kill-win-explorer-00_169.png" alt="Windows Explorer: drei gleichzeitig ge&ouml;ffnete Fenster" caption="Windows Explorer: drei gleichzeitig ge&ouml;ffnete Fenster" %}

Einzelne Fenster k&ouml;nnen in der UI Automation mittels eindeutig definierten Selektoren auseinander gehalten werden. Die Selektoren zu den im obigen Abbild gezeigten Fenstern lauten:
```
<wnd app='explorer.exe' cls='CabinetWClass' title='This PC' />
<wnd app='explorer.exe' cls='CabinetWClass' title='Desktop' />
<wnd app='explorer.exe' cls='CabinetWClass' title='Downloads' />
```

Wenn aber alle Fenster geschlossen werden sollen im Zuge einer Ausnahmebehandlung, ist es wahrlich umst&auml;ndlich den &Uuml;berblick zu behalten - es m&uuml;sste der Wert des Attributs 'title' je Fenster gespeichert werden. Dieses Vorgehen wird weiterhin zur Qual wenn die Titel der Fenster im Vorfeld unbekannt sind aufgrund der Nutzung von dynamischen Selektoren.


## Die L&ouml;sung
### Der Prozess 'explorer.exe'
Hinter Applikationen stecken Prozesse die f&uuml;r die Ausf&uuml;hrung von Aufgaben zust&auml;ndig sind und demnach ist es &uuml;blich diese ausfindig zu machen, unabh&auml;ngig von der Art und Weise deren Beendigung. Eine schnelle und einfache M&ouml;glichkeit den eigentlichen Prozess des Windows Explorer ausfindig zu machen besteht darin den Task Manager zu &ouml;ffnen und sich die Details der Applikation anzeigen zu lassen.

Der Task Manager l&auml;sst sich mit der Tastenkombination <Strg> + <Shift> + <Esc> &ouml;ffnen.
{: .notice}

{% include figure image_path="/assets/2020-04-22-rpa-close-and-kill-win-explorer/2020-04-22-rpa-close-and-kill-win-explorer-01_proctskmng.png" alt="Windows Explorer Prozess im Task Manager" caption="Windows Explorer Prozess im Task Manager" %}

Hinter allen Fenstern des Windows Explorer steckt genau ein Prozess namens 'explorer.exe', dieser ist ebenso Bestandteil der oben angegebenen Selektoren (Attribut 'app'). Mittels des Namens ist es eigentlich m&ouml;glich den Prozess ordnungsgem&auml;&szlig; zu schlie&szlig;en, doch daf&uuml;r w&uuml;rden in diesem Falle drei separate Prozesse erwartet, einer je Fenster. Wird nun der Prozess 'explorer.exe' &uuml;ber den Task Manager geschlossen, ergibt dies folgendes Resultat.

{% include figure image_path="/assets/2020-04-22-rpa-close-and-kill-win-explorer/2020-04-22-rpa-close-and-kill-win-explorer-02_noxplrdsktp.png" alt="Windows 10 Desktop nach Beendigung des Prozesses 'explorer.exe'" caption="Windows 10 Desktop nach Beendigung des Prozesses 'explorer.exe'" %}

Eine g&auml;hnende Leere ist zu betrachten, da alle GUI-Elemente verschwunden sind, sodass nur noch ein Hintergrundbild zu sehen ist. Der Windows Explorer beinhaltet n&auml;mlich nicht nur die GUI f&uuml;r das Navigieren im Dateisystem, sondern auch die Desktop Icons, das Startmen&uuml;, die Taskbar sowie die Systemsteuerung - zusammenfassend bekannt unter der Windows Shell.

### Entkopplung der Dateisytem-Fenster
Ein erneutes Starten des Prozesses 'explorer.exe' &uuml;ber den Task Manager w&auml;re eine L&ouml;sung des Problems, da anschließend alle Fenster des Dateisystems geschlossen sind. Aber ist es m&ouml;glich die Fenster von der Windows Shell zu trennen, um gezielt nur diese zu Beenden? In Windows gibt es hierzu eine Einstellung, die es uns gestattet die Dateisystem-Fenster in einem separaten Prozess starten zu lassen.

{% include figure image_path="/assets/2020-04-22-rpa-close-and-kill-win-explorer/2020-04-22-rpa-close-and-kill-win-explorer-03_xplrsepproc.png" alt="Einstellung: Dateisystem in einem separaten Prozess ausf&uuml;hren lassen" caption="Einstellung: Dateisystem in einem separaten Prozess ausf&uuml;hren lassen" %}

Ist diese Einstellung gesetzt, so werden alle Fenster in einem separaten Prozess gestartet. Aber Vorsicht ist geboten, denn dieser Prozess hei&szlig;t ebenso 'explorer.exe'.

### "Sanftes" Beenden der Windows Explorer Fenster - eins nach dem anderen?
Aufgrund der Trennung der Fenster von der Windows Shell sollte es nun m&ouml;glich sein diese eindeutig zu identifizieren und damit zu beenden. Aber auch hier wird uns ein Strich durch die Rechnung gemacht, wie folgende beispielhafte Auflistung der laufenden Prozesse namens 'explorer.exe' zeigt.

|PID |Prozess-Name |Fenster-Titel |
|-|-|-|
|14520 |explorer |Downloads |
|16412 |explorer | |

Die Windows Shell ist nun separat aufgef&uuml;hrt, erkenntlich an dem fehlenden Fenster-Titel, aber dennoch wird nur ein Fenster angezeigt, und zwar das aktuell aktive. Eine L&ouml;sung w&auml;re eine Schleife zu bauen, die alle Fenster schlie&szlig;t solange ein Prozess 'explorer.exe' mit einem Fenster-Titel vorhanden ist. Dies resultiert aber in einer (mir pers&ouml;nlich) unsch&ouml;nen Programmierung.

### "Unsanftes" statt "Sanftes" Beenden
Grunds&auml;tzlich sei gesagt, dass eine ordnungsgem&auml;&szlig;e Beendigung vorzuziehen ist, aber um alle Fenster in einem Wisch zu beenden erschlie&szlig;t sich aktuell kein anderer Weg als den Prozess f&uuml;r ein Fenster zu killen, wobei auch alle anderen Fenster geschlossen werden. Laut meiner Kenntnis entstehen hierbei auch keine Nebenwirkungen, z.B. den Verlust von Daten (sofern aktuell kein Vorgang l&auml;ft). Somit ist es ausreichend alle laufenden Prozesse einzuholen, nach einem Prozess 'explorer.exe' mit einem Fenster-Titel zu suchen und diesen dann zu killen. Nachfolgend ist hierzu der UiPath RPA-Programmausschnitt gezeigt.

{% include figure image_path="/assets/2020-04-22-rpa-close-and-kill-win-explorer/2020-04-22-rpa-close-and-kill-win-explorer-04_rpaimpl.png" alt="UiPath RPA Implementierung zum Beenden aller Windows Fenster" caption="UiPath RPA Implementierung zum Beenden aller Windows Fenster" %}

## Zusammenfassung
Die einzelnen Fenster des Windows Explorer lassen sich nur umst&auml;ndlich greifen und damit wird programmatisch ein ordentliches Beenden erschwert. Abhilfe hierbei bringt ein Kill des richtigen 'explorer'-Prozesses. Hierf&uuml;r ist aber die Einstellung das Dateisystem in einem separaten Prozess ausf&uuml;hren zu lassen unabdingbar.