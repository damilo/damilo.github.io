---
title: "Python Embedded: Beschaffung, Integration und Anwendung"
---


## Einführung
Python ist eine der bekanntesten und am meisten genutzten Programmiersprachen - dies bestätigen sowohl die <a href='http://pypl.github.io/PYPL.html' target='_blank'> Erstplatzierung im PYPL Index</a> als auch <a href='https://www.tiobe.com/tiobe-index/' target='_blank'>Rang Drei des TIOBE Index</a> (Stand: April 2020). Die Freiheit diese "[...] Software auszuführen, zu studieren, zu ändern und zu verbreiten [...]" resultiert in aktuell über 118.000 Site Packages und Machine Learning Engineers als auch Data Scientists nutzen Python intensiv.

Nichts­des­to­trotz gibt es Systemumgebungen in denen es untersagt ist Python direkt zu installieren aufgrund von IT Sicherheits-Richtlinien. Glücklicherweise bietet die Python Software Foundation eine sogenannte Embedded Distribution ab Version 3.5 an. Diese besteht aus einer minimalen Laufzeitumgebung und ist dafür gedacht in andere Applikationen oder Umgebungen eingebettet zu werden.

Dieser Artikel beschreibt eine Vorgehensweise zur Beschaffung der Python Embedded Distribution, welche Anforderungen eine Systemumgebung erfüllen muss für dessen Ausführung und wie Site Packages hinzugefügt werden.

Die fortan beschriebenen Ausführungen beziehen sich auf Computer mit Windows 7 oder höher als Betriebssystem. Die gezeigten bespielhaften Anwendungen nutzen Windows 10.
{: .notice}


## Vorabklärungen und Anforderungen
Die Python Embedded Distribution ist dafür vorgesehen in eine andere Applikation integriert zu werden und demnach richten sich die Anforderungen für dessen Verwendung hauptsächlich nach den Möglichkeiten der Einbettung, sodass Python mit der Applikation kommunizieren kann. Ein anderer Aspekt betrifft das verwendete Betriebssystem selbst, welches eine bestimmte Runtime enthalten haben muss, damit Python grundsätzlich lauffähig ist.

### Microsoft C Runtime
<a href='https://docs.python.org/3/using/windows.html#the-embeddable-package' target='_blank'>Die Dokumentation des Python Embeddable Package</a> führt eine wichtige Notiz auf:

"The embedded distribution does not include the Microsoft C Runtime and it is the responsibility of the application installer to provide this. The runtime may have already been installed on a user's system previously or automatically via Windows Update, and can be detected by finding ucrtbase.dll in the system directory."
{: .notice}

Die Embedded Distribution von Python greift auf die Microsoft C Runtime zurück und daher muss sichergestellt sein, dass diese auch installiert ist auf dem genutzten Betriebssystem. Feststellbar ist das Vorhandensein der Runtime mittels Suchen der Bibliothek namens `ucrtbase.dll` im Verzeichnis `<SysDrive>\Windows\System32\`. Diese Anforderung ist mandatorisch für die Funktionsfähigkeit von Python.

### Integrierungs-Applikation
Die Einbettung hängt sowohl von den Anforderungen der zu integrierenden Applikation _an_ Python als auch von den verwendeten Site Packages zur Erstellung der Software-Lösung _in_ Python ab. An einem Beispiel lässt sich dies anschaulich erklären.

Note: Die Evaluierung und Erstellung der Lösung ist ein Back-and-Forth-Prozess. Sofern ein Weg zu einer Sackgasse führt, müssen Schritte zurück und/oder erneut gegangen werden um zum Ziel zu gelangen.
{: .notice}

Angenommen die Software-Lösung in Python sei die Änderung der Farbtiefe von Bildern (von z.B. 32bit auf 8bit). Die Lösung wird analysiert und anschließend ein Design erstellt mit dem Ergebnis das Site Package _Pillow_ zu verwenden. Laut dem <a href='https://pypi.org/project/Pillow/' target='_blank'>Python Package Index</a> ist die neueste Version 7.1.2 und <a href='https://pillow.readthedocs.io/en/stable/installation.html#notes' target='_blank'>Pillow's Dokumentation</a> besagt, dass mindestens die Version 3.5 von Python benötigt wird. Auf der anderen Seite steht die Integrierungs-Applikation, welche Python nur in den Versionen 3.4 bis 3.6 sowohl in der 32bit- als auch 64bit-Variante unterstützt. Folgende Darstellung zeigt die Abhängigkeiten der zu verwendenden Komponenten.

{% include figure image_path="/assets/2020-05-08-integration-of-python-embedded/2020-05-08-integration-of-python-embedded-00_IntegrationAppComponents.png" alt="verwendete Komponenten für die Integration von Python Embedded" caption="verwendete Komponenten für die Integration von Python Embedded" %}

Die drei Ebenen _Integrierungs-Applikation_, _Python_ und _Site Packages_ enthalten für sich alle möglichen Komponenten. Verbindungslinien von einer Ebene zur anderen zeigen die zueinander kompatiblen Komponenten auf. Die grün hervorgehobenen Verbindungslininen geben schlussendlich die tatsächlich verwendbaren Komponenten an. In diesem Falle kann die 32bit- als auch 64bit-Variante von Python in der Version 3.5 oder 3.6 verwendet werden um den Anforderungen gerecht zu werden.


## Durchführung
Folgend wird die Beschaffung der Python Embedded Distribution als auch die Integration von Site Packages beschrieben.

### Python Embedded beziehen
Die Python Embedded Distribution ist über die Website der <a href='https://www.python.org/downloads/windows/' target='_blank'>Python Releases for Windows</a> zu beziehen. Unter den _Stable Releases_ ist die entsprechende Version zu suchen (z.B. Python 3.6.8 - Dec. 24, 2018) und anschließend das sogenannte _embeddable zip file_ in der 32bit- (x86) oder 64bit-Variante (x86-64) herunterzuladen. Die Zip-Datei kann auf dem Ziel-System in jeden beliebigen Ordner entpackt werden.

{% include figure image_path="/assets/2020-05-08-integration-of-python-embedded/2020-05-08-integration-of-python-embedded-01_PyEmbDl.png" alt="Beziehen und Entpacken von Python Embedded" caption="Beziehen und Entpacken von Python Embedded" %}

Die Ausgabe der Version von Python zeigt folgendes an:
```
C:\Users\Daniel\Desktop\python36\python-3.6.8-embed-amd64>python --version
Python 3.6.8
```

Ein Start der `python.exe` gibt die gewohnte Programmierumgebung über die Kommmandozeile:
```
C:\Users\Daniel\Desktop\python36\python-3.6.8-embed-amd64>python
Python 3.6.8 (tags/v3.6.8:3c6b436a57, Dec 24 2018, 00:16:47) [MSC v.1916 64 bit (AMD64)] on win32
>>> import sys
>>> sys.exit ()
```

### Site Packages installieren
Die Integration der Site Packages in die Python Embedded Distribution gestaltet sich umfangreicher aufgrund der Tatsache, dass eine Referenz-Umgebung erstellt werden muss und beinhaltet
- die Installation von Python
- die Installation des Package Installer for Python (pip)
- die Installation der notwendigen Site Packages, bedingt durch die angestrebte Software-Lösung.

Anschließend werden die zuvor installierten Site Packages in das Verzeichnis der entpackten Python Embedded Distribution kopiert und der Umgebung bekannt gemacht.

Die Referenz-Umgebung kann auf jedem beliebigen System erstellt werden - d.h. es muss nicht auf dem Ziel-System für Python Embedded geschehen.
{: .notice}

Folgend wird das Ausfindigmachen und Integrieren der Site Packages beschrieben. Eine Installation der notwendigen Komponenten wird vorausgesetzt.

Mittels Befehl `pip show -f [package-name]` wird Information zu sowohl den Anforderungen des Site Package selbst als auch dessen notwendige Dateien anzeigt. Nachfolgend ist beispielhaft die Ausgabe des Site Package _Pillow_ angegeben.
```
(pyemb) C:\_pyenv\pyemb>pip show -f Pillow
Name: Pillow
Version: 7.1.2
Summary: Python Imaging Library (Fork)
Home-page: https://python-pillow.org
Author: Alex Clark (PIL Fork Author)
Author-email: aclark@python-pillow.org
License: HPND
Location: c:\_pyenv\pyemb\lib\site-packages
Requires:
Required-by:
Files:
  PIL\BdfFontFile.py
  PIL\BlpImagePlugin.py
  PIL\BmpImagePlugin.py
  ...
  Pillow-7.1.2.dist-info\WHEEL
  Pillow-7.1.2.dist-info\top_level.txt
  Pillow-7.1.2.dist-info\zip-safe
```
Aus dieser Information ist zu entnehmen, dass
- das Site Package im Verzeichnis `c:\_pyenv\pyemb\lib\site-packages` installiert ist
- keine Abhängigkeiten zu weiteren Site Packages bestehen (`Requires: [...]`)
- die notwendigen Dateien in den Verzeichnissen `PIL` und `Pillow-7.1.2.dist-info` liegen.

Um dieses Site Package in der Python Embedded Distribution nutzbar zu machen, müssen die besagten Verzeichnisse aus `c:\_pyenv\pyemb\lib\site-packages` in eine eigens erstellte Verzeichnis-Struktur kopiert werden. __Wichtig__: Dieser Vorgang muss ebenso für alle zusätzlich geforderten Site Packages durchgeführt werden (in diesem Falle existieren keine weiteren Abhängigkeiten). Folgend wird eine mögliche Verzeichnis-Struktur aufgezeigt:
```
+-- Python Embedded (Oberverzeichnis)
  +-- python-3.6.8-embed-amd64 (Python Embedded Distribution)
  +-- scripts (Verzeichnis für Skripte)
  +-- lib
    +-- site-packages (Verzeichnis für Site Packages)
      +-- PIL
      +-- Pillow-7.1.2.dist-info
```

Schlussendlich werden der Python Embedded Distribution die Pfade zu den Site Packages und Skripten bekannt gemacht, indem diese der Datei `python36._pth` aus Verzeichnis `python-3.6.8-embed-amd64` hinzugefügt werden:
```
[file: python36._pth]

python36.zip
.

# Uncomment to run site.main() automatically
#import site

../lib/site-packages
../scripts
```

Die resultierende Python Embedded Distribution ist nun fähig auf die Funktionalitäten des Site Package zuzugreifen und bereit in die Integrierungs-Applikation eingebettet zu werden.


## Anwendungsbeispiele
Quellen der Anwendungen:
- <a href='/assets/2020-05-08-integration-of-python-embedded/pyemb36_Pillow.zip'>Anwendung 1: Bildbearbeitung, Pillow</a>
- <a href='/assets/2020-05-08-integration-of-python-embedded/pyemb36_spamest.zip'>Anwendung 2: RPA, UiPath Studio, Machine Learning, scikit-learn</a>
{: .notice}

### Anwendung 1: Bildbearbeitung, Pillow
Das folgend beschriebene Beispiel zeigt die Verwendung des Site Package _Pillow_ und bezieht sich auf die in der Durchführung beschriebenen Erläuterungen. Sämtliche Schritte zur Bereitstellung der Python Embedded Distribution inklusive des Site Package sind dieser zu entnehmen.

Ein Python-Skript soll erstellt werden, welches mehrfahrbige in Schwarz-Weiß-Bilder umwandelt. Mittels _Pillow_ ist dies folgendermaßen zu bewerkstelligen:
```
[file: pillow_example.py]

import sys
import os
from PIL import Image

def main ():
    if len (sys.argv) != 2:
        print ("Please provide image file as argument.\nProgram will exit now...")
        return
    
    imgFilePath = sys.argv[1]
    with Image.open (imgFilePath) as img:
        img = img.convert ("L")
        fileName, fileExt = os.path.splitext (imgFilePath)
        img.save (fileName + "_bw" + fileExt)
    
    return


if __name__ == "__main__":
    main ()
```

Um dieses Skript in der Umgebung von Python Embedded laufen zu lassen ist nachstehender Befehl in einer Kommandozeile auszuführen (es wird beispielhaft das existierende Bild 'P1030459.JPG' verwendet):
```
C:\Users\Daniel\Desktop\python36>python-3.6.8-embed-amd64\python.exe pillow_example.py P1030459.JPG
```

{% include figure image_path="/assets/2020-05-08-integration-of-python-embedded/2020-05-08-integration-of-python-embedded-02_ExamplePillow.png" alt="Beispiel: Änderung der Farbtiefe unter Verwendung des Site Package _Pillow_" caption="Beispiel: Änderung der Farbtiefe unter Verwendung des Site Package _Pillow_" %}

Das Resultat ist ein neues Bild namens 'P1030459_bw.JPG' in schwarz-weiß.

### Anwendung 2: RPA, UiPath Studio, Machine Learning, scikit-learn
Eine fortgeschrittene Anwendung bildet die Kombination von RPA und Python. Die Software-Lösung soll den Inhalt von E-Mails aus einem Postfach lesen und Spam klassifizieren mittels einem in Python erstellten Machine Learning Algorithmus.

Die Integrations-Applikation soll UiPath Studio v2019.10 sein, eine Entwicklungsumgebung zur Erstellung von UI-Automationen, welche das Lesen und Bereitstellen des Inhalts der E-Mails übernimmt. UiPath stellt ein <a href='https://gallery.uipath.com/packages/UiPath.Python.Activities/' target='_blank'>Activity Package zur Kommunikation mit Python</a> zur Verfügung, dessen aktuelle Version 1.0.7346.28183 ist. Die Activity `Python Scope` gibt Aufschluss über die verwendbaren Versionen von Python. Die Eigenschaft _Version_ der Activity stellt folgende Auswahl bereit: ['Auto', 'Python_27', 'Python_33', 'Python_34', 'Python_35', 'Python_36']. Außerdem kann sowohl die 32bit- als auch 64bit-Variante von Python verwendet werden.

Die Anforderungen an Python werden weder in der <a href='https://gallery.uipath.com/packages/UiPath.Python.Activities/' target='_blank'>Package Gallery</a> noch in dem <a href='https://docs.uipath.com/activities/docs/about-the-python-activities-pack' target='_blank'>Python Activity Guide</a> angegeben, sodass keine andere Wahl bleibt als das Package herunterzuladen, zu einem UiPath-Projekt hinzuzufügen und die Activities selbst zu evaluieren.
{: .notice}

Auf der Ebene der Python Site Packages wird _scikit-learn_ für die Implementierung der Machine Learning Komponente verwendet. Laut dem <a href='https://pypi.org/project/scikit-learn/' target='_blank'>Python Package Index</a> ist die neueste Version 0.22.2 und benötigt mindestens die Version 3.5 in der <a href='https://scikit-learn.org/stable/install.html#installing-the-latest-release' target='_blank'>64bit-Variante</a> von Python sowie die Site Packages Numpy >= 1.11.0, SciPy >= 0.17.0 und joblib >= 0.11.

Das Vorgehen zur beschriebenen Durchführung unterscheidet sich lediglich derart, dass
- mehr Python Site Packages inkludiert werden müssen aufgrund von Abhängigkeiten
- die Kommunikation zwischen Integrierungs-Applikation und Python mittels einem API geschieht.

#### Python Site Packages
Die Package Information zu _scikit-learn_ gibt folgendes aus:
```
(pyemb) C:\_pyenv\pyemb>pip show -f scikit-learn
Name: scikit-learn
Version: 0.22.2
Summary: A set of python modules for machine learning and data mining
Home-page: http://scikit-learn.org
Author: None
Author-email: None
License: new BSD
Location: c:\_pyenv\pyemb\lib\site-packages
Requires: numpy, joblib, scipy
Required-by:
Files:
  scikit_learn-0.22.2.dist-info\COPYING
  scikit_learn-0.22.2.dist-info\INSTALLER
  scikit_learn-0.22.2.dist-info\METADATA
  ...
  sklearn\utils\tests\test_validation.py
  sklearn\utils\validation.py
  sklearn\utils\weight_vector.py
```
Aus dieser Information ist zu entnehmen, dass
- das Site Package im Verzeichnis `c:\_pyenv\pyemb\lib\site-packages` installiert ist
- Abhängigkeiten zu weiteren Site Packages bestehen (`numpy, joblib, scipy`)
- die notwendigen Dateien in den Verzeichnissen `scikit_learn-0.22.2.dist-info` und `sklearn` liegen.

Neben den Dateien für _scikit-learn_ müssen auch die Information jedes einzelnen abhängigen Site Package _numpy, joblib_ sowie _scipy_ evaluiert und deren Dateien kopiert werden. Dies wiederholt sich solange, bis keine weiteren Abhängigkeiten vorhanden sind. Beispielhaft wird die Information von _numpy_ ausgegeben:
```
(pyemb) C:\_pyenv\pyemb>pip show -f numpy
Name: numpy
Version: 1.18.4
Summary: NumPy is the fundamental package for array computing with Python.
Home-page: https://www.numpy.org
Author: Travis E. Oliphant et al.
Author-email: None
License: BSD
Location: c:\_pyenv\pyemb\lib\site-packages
Requires:
Required-by: scipy, scikit-learn
Files:
  ..\..\Scripts\f2py.exe
  numpy-1.18.4.dist-info\INSTALLER
  numpy-1.18.4.dist-info\LICENSE.txt
  numpy-1.18.4.dist-info\LICENSES_bundled.txt
  ...
  numpy\tests\test_scripts.py
  numpy\tests\test_warnings.py
  numpy\version.py
```
Das Site Package hat keinerlei weitere Abhängigkeiten und die notwendigen Dateien bestehen aus den Verzeichnissen `numpy-1.18.4.dist-info` und `numpy` _sowie_ der ausführbaren Datei `f2py.exe` unter Scripts.

Eine mögliche resultierende Verzeichnis-Struktur der Python Embedded Distribution nach Kopieren sämtlicher Site Packages ist folgend angegeben:
```
+-- Python Embedded (Oberverzeichnis)
  +-- python-3.6.8-embed-amd64 (Python Embedded Distribution)
  +-- scripts (Verzeichnis für Skripte)
    |-- f2py.exe
  +-- lib
    +-- site-packages (Verzeichnis für Site Packages)
      +-- numpy
      +-- numpy-1.18.4.dist-info
      +-- joblib
      +-- joblib-0.14.1.dist-info
      +-- scipy
      +-- scipy-1.4.1.dist-info
      +-- sklearn
      +-- scikit_learn-0.22.2.dist-info
```

Das Bekanntmachen der Pfade für Python geschieht wie in der Durchführung beschrieben.

#### Python API der Integrierungs-Applikation
Hauptaugenmerk für diese Komponente der Software-Lösung liegt auf der Art und Weise der Verwendung der Python Activities. Aufgrund der Tatsache, dass UiPath Studio eine visuelle Programmierumgebung bietet, lässt sich die Implementierung bildlich darstellen. Die Kommentare geben Aufschluss über die gesetzten Activity Properties.

{% include figure image_path="/assets/2020-05-08-integration-of-python-embedded/2020-05-08-integration-of-python-embedded-03_ExamplePythonScope-Python368x64.jpg" alt="Beispiel: Python Activities der Integrierungs-Applikation UiPath Studio" caption="Beispiel: Python Activities der Integrierungs-Applikation UiPath Studio" %}

Die Activity `Invoke Python Method` ruft eine Python-Funktion namens `load_predict` auf. Die Definition dieser Funktion sieht folgendermaßen aus (Info: es handelt sich um einen Auszug aus dem Python-Skript mit Helferfunktionen):
```
[file: spamest_consumer.py (Auszug)]

import sys
from sklearn.neural_network import MLPClassifier

try:
    import cPickle as pickle
except ModuleNotFoundError:
    import pickle


def load_vectorizer (vecFilePath):
    with open (vecFilePath, 'rb') as fileObj:
        vec = pickle.load (fileObj)
    return vec


def load_estimator (mlmFilePath):
    with open (mlmFilePath, 'rb') as fileObj:
        est = pickle.load (fileObj)
    return est


def predict (est, X):
    y_pred = est.predict (X)
    return y_pred


def load_predict (mlmFilePath, vecFilePath, sample):
    vec = load_vectorizer (vecFilePath)
    X = vec.transform ([sample]).toarray ().astype ('f4')
    return predict (load_estimator (mlmFilePath), X)
```
Der Funktion `load_predict` werden drei Parameter übergeben (zwei Dateipfade zu serialisierten Python-Objekten sowie das auszuwertende Sample). Wichtig zu erkennen ist, dass die Parameter sich exakt in dem Listen-Objekt `inputParamList` in UiPath Studio widerspiegeln.


## Zusammenfassung und Ausblick
In wenigen Schritten lässt sich ein Windows-System mit einer vollwertigen Python-Laufzeitumgebung ausstatten - (bis auf das Referenz-System) gänzlich ohne Installationen. Die Laufzeitumgebung lässt sich grundsätzlich überall ablegen und ausführen, sei es lokal oder in einem Netzwerk-Verzeichnis.

Die Anwendungsbeispiele zeigen nur einen Ausschnitt der Kommunikationswege auf. Eine weitere Einbettung ist möglich mittels Aufrufen der Python Executable mit Parametern und anschließendem Auslesen des Standard Outputs (stdout). Hierdurch werden sogar bestehende Abhängigkeiten zwischen Site Packages und Integrierungs-Applikation aufgelöst, sodass jede Version der Python Embedded Distribution verwendbar ist.