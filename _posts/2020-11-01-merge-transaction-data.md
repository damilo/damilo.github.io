---
title: Aktualisieren von Transaktions-Informationen mittels Zusammenführen von Datentabellen
---

Datentabellen (DataTables) sind ein oft genutztes Vehikel in der RPA, um eine Vielzahl verschiedenartiger Information zu speichern. Hierunter fallen ebenso Daten für die Verarbeitung von Transaktionen.

## Das Problem
Ein RPA-Prozess soll als sogenannter Performer dienen und gegebene Transaktionen verarbeiten. Alle Transaktionen, ungeachtet von deren Bearbeitungs-Status, sind in einer Excel-Queue namens `TransactionQueue.xlsx` hinterlegt. Ferner sollen nur Transaktionen verarbeitet werden, deren Bearbeitungs-Status 'neu' oder 'fehlgeschlagen' ist. Abschließend sollen die erfolgreich verarbeiteten Transaktionen in der Excel-Queue aktualisiert werden.

## Die Lösung
Quellen der Anwendung:
- <a href='/assets/2020-11-01-merge-transaction-data/DataTable.UpdateTransactions.xaml'>DataTable.UpdateTransactions.xaml (UiPath Studio)</a>

Die Lösung besteht aus den zwei Teilen
- Einholung der zu verarbeitenden Transaktionen und
- Aktualisierung der (erfolgreich) verarbeiteten Transaktionen.

Es muss zudem eine Voraussetzung erfüllt sein um eine funktionierende Lösung zu erhalten: Jede Transaktion hat eine eindeutige Referenz. Beispiele für eindeutige Referenzen sind Bestellnummer, Vorgangsnummer, aber auch eigens gewählte Identifizierungen wie Datum einer E-Mail oder ein Globally Unique Identifier (GUID). Daher ist es ratsam ein entsprechendes Format der Excel-Queue vorab zu definieren.

### Einholung der zu verarbeitenden Transaktionen
Die Excel-Queue wird eingelesen und in eine DataTable `QueueData` überführt. Anschließend wird die DataTable gefiltert, um alle Transaktionen mit Bearbeitungs-Status 'neu' oder 'fehlgeschlagen' zu erhalten. Schlussendlich werden die zu bearbeitenden Transaktionen in eine neue DataTable `TransactionData` kopiert.

Bereits in diesem Prozessschritt wird wichtige Vorarbeit geleistet für die abschließende Aktualisierung der verarbeiteten Transaktionen: Die neue DataTable `TransactionData` erhält die gleiche Struktur wie die DataTable `QueueData`. Mittels Instanz-Methode <a href='https://docs.microsoft.com/en-us/dotnet/api/system.data.datatable.clone?view=netframework-4.6' target='_blank'>*DataTable.Clone*</a> lässt sich dies bewerkstelligen und vereinfacht das Zusammenführen von zwei DataTables.

Die DataTable `TransactionData` ist für die Verarbeitung der Transaktionen verwendbar.

### Aktualisierung der verarbeiteten Transaktionen
Eine Zusammenführung der verarbeiteten Transaktionen mit der Excel-Queue wird unter Verwendung der Instanz-Methode <a href='https://docs.microsoft.com/en-us/dotnet/api/system.data.datatable.merge?view=netframework-4.6' target='_blank'>*DataTable.Merge*</a> durchgeführt. Eine Überladung dieser Methode erwartet drei Parameter, die nachfolgend erklärt sind:

`table`: eine zu übertragende Quell-DataTable - in diesem Falle die DataTable `TransactionData`.

`preserveChanges`: ein Schalter zur Bestimmung ob Änderungen in der Ziel-DataTable beibehalten werden sollen - ist auf `false` zu setzen, da eine Änderung der Excel-Queue während der Verarbeitung ausgeschlossen ist.

`missingSchemaAction`: eine Angabe über Was zu tun ist im Falle von unterschiedlichen Strukturen der zusammenzuführenden DataTables - Die Einstellung dieses Parameters hängt davon ab, ob sich die DataTable `TransactionData` während der Verarbeitung ändern kann (beispielsweise durch Hinzufügen einer neuen Spalte mit zusätzlicher Information der Transaktionen). Es ist sinnvoll den Wert auf `System.Data.MissingSchemaAction.Add` zu setzen.

Die Zusammenführung wird eingeleitet durch das erneute Einlesen und Speichern der Excel-Queue in eine Ziel-DataTable (z.B. namens `QueueData`). Nachfolgend ist ein sogenannter *Primary Key* in dieser DataTable zu setzen über Instanz-Eigenschaft <a href='https://docs.microsoft.com/en-us/dotnet/api/system.data.datatable.primarykey?view=netframework-4.6' target='_blank'>*DataTable.PrimaryKey*</a>. Hier wird die Spalte mit der eindeutigen Referenz einer Transaktion angegeben. Die Zusammenführung der Ziel- und Quell-DataTable ist mittels Instanz-Methode *DataTable.Merge* unter Verwendung der obig genannten Parameter auszuführen. Schlussendlich wird die Ziel-DataTable zurück in die Excel-Queue geschrieben.