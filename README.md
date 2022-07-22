# ParkingSystem

**Funktion**: (elektrische) Parkplätze von Garagen, aus einer Datei herausgelesen oder selbst hinzugefügt, verwalten. 
Das Kennzeichen der Fahrzeuge wird in den Garagen abgespeichert und von Bildern abgelesen. 

## Installation
#### Google Colab/Jupyter Notebooks
    ParkingSysten.ipynb und alle Dateien (Bilder, Dataset,... inklusive eigene Bilder) hochladen. data.zip entzippen im Notebook.
    Es ist empfohlen den GPU Laufzeittyp zu verwenden

## Hauptprogramm
Das Hauptprogramm hat 6 Funktionen:  
**Ablauf**: Autos in Garagen reinfahren und rausfahren lassen, Informationen über freie Parkplätze. Wenn ein Auto rausfährt, wird eine Rechnung (in Form einer Textdatei) geschickt  
**Garage hinzufügen**: Selbsterklärend, Name und Anzahl der Parkplätze auswählen. Elektrische Parkplätze können auch hinzugefügt werden  
**Garage entfernen**: Selbsterklärend  
**bestimmtes Auto rausfahren lassen**: Garage auswählen und dann ein Fahrzeug (mithilfe von Kennzeichen) rausfahren lassen. Eine Rechnung wird verschickt  
**Daten abspeichern**: Falls Garagen hinzugefügt oder gelöscht wurden, wird das in der garages.txt Textdatei aktualisiert. Diese Datei wrid beim Programmstart gelesen  
**Beenden**: Selbsterklärend  
 
### Classes:

#### Car
    hat ein Kennzeichen und ein Bild davon. (Kennzeichen wird zufällig generiert falls nicht vorhanden)
    leaveGarage(garage): verlassen einer Garage, dabei wird eine Rechnung (Zeit + Kosten) geschickt

#### Garage
    Name, Anzahl der Parkplätze und freie Parkplätze, Liste aus Autos (Car)
	percentageFreeParking(): Freie Parkplätze in Prozent
	info(): gibt Name, Anzahl der freien Parkplätze, Prozent der freien Parkplätze an
	changeCars(difference, eletric = False): Ändert die Anzahl der freien Parkplätze, postive difference = Autos kommen dazu, negative difference = Autos verlassen (Rechnung verschickt) parameter electric ist nur für EGarage relevant
	
#### EGarage
    Abgleitet von Garage, einziger Unterschied ist, dass es elektrische Parkplätze hat. Alles andere bleibt gleich

#### ParkGuidance
    verwalten von Garagen (gespeichert in einer Liste)
	addGarage(garage): selbsterklärend
	removeGarage(garage): selbsterklärend
	sortGarages(): sortiert Garagen nach freien Plätzen

#### GarageFileReader
    liest die Daten aus garages.txt und erstellt einen ParkGuidance und fügt die Garagen hinzu
	writeData: statische Methode, aktualisiert garages.txt, falls Garagen entfernt oder hinzugefügt worden sind
	
## Kennzeichen aus Bild herauslesen
Zuerst wird das Bild auf 400x400 angepasst. Dann wird nach das Kennzeichenschild gesucht und rausgeschnitten, damit das Bild nur das Kennzeichen hat.  
Dieses Bild wird dann in Schwarz und Weiß verarbeitet, damit die Zahlen und Buchstaben deutlich erkennbar sind.  
Dann wird es in mehrere Bildern geteilt: Aus jeder Zahl und Buchstabe wird ein Bild gemacht.  
Das trainierte Tensorflow Model schaut sich jedes Bild an und gibt es als Text zurück.  
Beachte, dass das Model zuerst trainiert werden muss, bevor es verwendet werden kann.  

### Anwendung:
Zuerst musst das Model für die Zahl und Buchstaben Erkennung kompiliert und trainiert werden. 30-40 Epochen sind ausreichend.   
Die Größe des Bildes muss zuerst angepasst werden, falls nötig. Die Empfohlene Auflösung ist zwischen 300x300 und 500x500.   
Das neue Bild kommt dann in die detect_plate(img, text='') function. Der Output muss einer Variable zugewiesen werden. Beachte, dass 2 Bilder zurückgeliefert werden.   Das Erste ist das Originalbild mit den Kennzeichen markiert und das Zweite ist nur dass Kennzeichen.   
Danach kommt das Kennzeichen Bild in die segment_characters(image) function. Der Output ist eine Liste von allen Buchstaben und Zahlen gefunden als Bild.  
Wenn das Model schon trainiert ist, können wir schon die show_results() function aufrufen. Als Output bekommen wir das Kennzeichen als string geliefert.  

### Functions:
    detect_plate(img, text=''): sucht nach dem Kennzeichen im Bild (img) und gibt das Bild mit den markierten Kennzeichen und ausgeschnittenen Kennzeichen zurück
    display(img, text=''): Bild (img) anzeigen lassen, Text kann hinzugefügt werden
    find_contours(dimensions, img): Markiert alle Zahlen und Buchstaben im Bild mit einen blauen Rechteck
    segment_characters(image): macht das Bild Schwarz-Weiß und teilt es in mehrere und gibt es zurück als eine Liste
    f1score(y, y_pred): Model Perfomance Werte während des Training
    fix_dimensions(img): Bild anpassen, dass es 28x28 groß ist
    show_results(): nimmt jedes Bild in der Liste (von segment_characters()) und schreibt die Zahl/Buchstabe vom Bild

### Classes:
    stop_training_callback(tf.keras.callbacks.Callback): diese Klasse wird beim Model Training benutzt.
    Es wird solange trainiert bis es einen bestimmten f1score (0.999) hat.
