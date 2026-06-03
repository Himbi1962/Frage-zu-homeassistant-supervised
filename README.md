# Energie-Management für Viessmann Wärmepumpe

## 🏠 Projekt-Übersicht
Kostenoptimales Energiemanagement (PEMS) für ein Einfamilienhaus mit:
- Viessmann Vitocal 250-A Wärmepumpe
- Brunner Kachelofeneinsatz mit Wassertasche
- Sailer Pufferspeicher
- PV-Anlage 9,5 kWp
- Viessmann Vitocharge Vx3 15kWh
- UVR610 (Technische Alternative Regler)
- RPi5 mit Home Assistant
- Synology 1520+ (Modbus-Slave + MySQL)
- AtomS3 Lite (Stromzähler & Batterie-Daten einer Averiy P210)

## Projekt-Beschreibung
Ausgehend von der Definition, in welchen Nutzugszeitabschnitten Räume im Haus welche Nutztemperaturen haben sollen, soll eine Logik auf der Basis von Wetterprognosen wie Temperatur und Wind, sowie die
Sonneneinstrahlung auf der Basis von PV-Ertragsprognosen vorausberechnen, welche Wärmeenergie auf welchem Temperaturniveau der Heizkreise in den nächsten 24-Stunden benötigt wird.
Dabei soll die Logik mit der Zeit selbst lernen, wie sich das Gebäude verhält, also welche Wärmeverbräuche entstehen bei den Ist-Klimadaten. Es soll auch lernen, welches Temperatur-
niveau der Heizkreise erforderlich ist, um die Nutztemperatur bei gegebenen Klimadaten zu erhalten; so sollen Grundlastkurven für die Räume entwickelt werden. Wären z.B. alle Stellventile der
Wärmeübertrager in einem Heizkreis geschlossen, wäre die gewählte Heizkreistemperatur bei den aktuellen Klimadaten zu hoch und die Grundlastkurve sollte geeignet reduziert werden. 
Kann die Nutztemperatur in einem Raum nicht erreicht/gehalten werden, muss die Grundlastkurve erhöht werden.
Mit der Zeit soll die Logik auch lernen, wie sich die Räume bei Auskühlung bzw. Aufheizung verhalten. Trivial ist, dass im Fall der Auskühlung der Wärmeverbrauch auf Null zurückgeht.
Weniger trivial ist, wie schnell der einzelne Raum bei den gegebenen Klimadaten auskühlt. Aufwändiger hingegen wird die Betrachtung der Aufheizung. Um einen Raum auf die Nutztemperatur
zu erhöhen, braucht es einen Zuschlag zur Grundlastkurve, um den jeweiligen Raum bis zum Beginn des nächsten Nutzungszeitabschnittes die erforderliche Raumtemperatur erreicht zu haben. Dies gilt für
Fußbodenheizungen gleichermaßen, wie für Radiatorenheizungen. Es gilt: Je kürzer die Aufheizzeit ist, desto höher muss der Zuschlag zur Grundlastkurve sein, was erhebliche Auswirkungen auf die
Effiziens der Wärmepumpe hat. Es gilt die Faustformel, dass je Grad Temperaturerhöhung 1,5% Effiziensverlust entstehen.
Hier beginnt eine spannende Aufgabe für die Logik, die selbsttätig berechnen soll, ob eine Absenkung der Raumtemperaturen unterhalb der Nutztemperatur wirtschaftlich sinnvoll ist, wie lange die
Absenkphase dauern darf, also wann mit der Aufheizung mit welchen Temperaturen begonnen werden soll. Möglicherweise wird das Ergebnis so sein, dass außerhalb von Urlaubsphasen nur gering oder gar nicht
abgesenkt werden sollte, da die Energieeinsparung durch geringere Wärmeverluste aufgrund geringerer Raumtemperatur so klein ist, dass durch den Effiziensverlust der Wärmepumpe eher eine Kostensteigerung
durch den erhöhten Stromverbrauch zu erwarten ist. Langfristige Absenkungsphasen z.B. im Urlaub können hiervon abweichen, aber genau da ist es wichtig, dass die Heizung sogar mehrere Tage vor der Rückkehr
selbsttätig starten muss, um dann die Räume wieder auf Nutztemperatur zu bekommen.
Eine wichtige Rolle bei der Vorausberechnung spielen dabei die Maschinenkennwerte der Wärmepumpe laut Viessmann, Wetterprognosen von OpenWeather, PV-Ertragsprognosen von Solstat, sowie die Strompreise für
Haushalt und Wärmepumpe, möglicherweise mit dynamischen Strompreisen wie Tibber (die allerdings nur für höchstens 36 Stunden im Voraus verfügbar sind, kurz vor dem Abruf der neuen Tibber-Daten sogar nur für
10 Stunden - für eigene weitergehende Prognosezeiträume müssten dann auch eine Abschätzungen der Strompreisentwicklungen innerhalb der weitergehenden Prognosezeiträume) Im Regelfall sollte die Logik z.B.
im Stundenabstand prognostizieren, welche Wärme- und Stromverbräuche je Stunde das Haus in den kommenden 24 Stunden hat. Nun muss festgelegt werden, wann die Wärmepumpe unter Berücksichtigung der Effizienzen
der Speicher die benötigte Wärmemenge erzeugt und die Speicher auflädt. Für die Wärmeverbräuche steht der Sailer-Schichtspeicher mit 1200 Litern - davon 800 Liter für die Heizung -  bzw. das Gebäude mit
einer möglichen Temperaturüberhöhung von 1-2°C zur Verfügung. Die Warmwassererzeugung funktioniert über eine Frischwasserstation, die sich aus der oberen Heizungswasservorlage (400 Liter) aus dem Sailerpuffer
bedient. Sollte eine Heizungspuffertemperatur erforderlich sein, die höher als die Warmwasservorlage ist, wird immer zunächst die Warmwasservorlage zunächst überhöht, bevor der Heizungsbereich aufgeladen
wird. Also wird ein für den Schichtpuffer ein geeignetes Puffermanagement erfoderlich werden, das eine Simulation der Pufferladung und -entladung ermöglicht. Dies gilt ebenso für den "Wärmepuffer Haus". Hier
besteht die Möglichkeit in pekuniär günstigen Phasen die Aufheizung vorzeitig bzw. leicht überhöht durchzuführen, was den Wärmeverlust zwar etwas erhöht, aber dafür in günstigere Phasen verschiebt. Eine große
Rolle spielen dabei auch die Maschinenkennwerte der Wärmepumpe. Zwar gibt der Wärmepumpenhersteller mehrere Kurven für die Wärmeerzeugung und den Stromverbrauch an in Abhängigkeit der Rücklauf- und der Außen-
temperatur. Es fehlt jedoch die Abhängigkeit von der abgeforderten Leistung, sprich die an die Wärmepumpe übermittelte Vorlauf-Solltemperatur. Auch diese Zusammenhänge muss die Logik über die Zeit evtl. 
als Funktionen f(TA-Ist, RLT-Ist, VLT-Soll) für die Bereiche unter- und oberhalb der Unstetigkeitsstelle der Invertermaschine.
Nun sind nicht alle Wärmeübertrager in einem Heizkreis gleichwertig, wenn die Voreinstellungen der Ventile nicht stimmig sind. Von Zeit zu Zeit (Monatsweise in der Heizzeit) sollten die Wärmeübertrager daraufhin
ausgewertet werden, welche Stellventile in der Beharrungsphase am längsten und am kürzesten voll geöffnet waren. Die am längsten geöffneten Ventile müssen in den Voreinstellungen eher reduziert, das am kürzesten
muss voll geöffnet werden. Für die dazwischen liegenden müssen so angepasst werden, dass die Öffnungszeiten denen des voll geöffneten nahe kommen. Dabei wird sich vermutlich mit der Zeit auch eine leichte 
Veränderung der Grundlastkurve ergeben.
Für all das müssen eine Menge Daten in der relationalen Datenbank erfasst werden, z.B. Viertelstundenweise Prognosen und Istwerte, aus denen die Logik mit der Zeit die Vorgaben für die wirtschaftlich optimale 
Wärmeerzeugung entwickeln kann. Es wird auch Zeiten geben, in denen die Wärmepumpe nicht die erforderliche Wärmeleistung alleine erbringen kann, dann müssen weitere externe Wärmeerzeuger (vorbeugend wie den
Kachelofen) oder die Heizstäbe der Wärmepumpe freigegeben werden. 
Die Logik soll auch die Stromverbräuche, die Batterie-(ent-)ladungen, die Stromerzeugung über PV beurteilen. Das wird je nach Strompreismodell statisch, dynamisch oder evtl. auch gemischt, den Wärmeerzeugungszeitraum wesent-
lich beeinflussen.
Für die Zukunft werden möglicherweise weitere kleinere Batteriesysteme für Steckergeräte ergänzt, deren Ladung von der Logik ebenfalls regeln soll. Außerdem soll der RPi mal durch einen leistungsfähigen
Industrie-PC ersetzt werden.

## 📋 Komponenten
- **UVR610** (TA): Wärmepumpen- und Speicherregelung als Fallback-Ebene
- **RPi5**: Home Assistant (aktuell), Berechnung der Betriebsvorgaben der Wäremepumpe und der Batteriespeicher, zur Übergabe an den UVR610, später nur noch Datenerfassung/Dashboard
- **Synology 1520+**: MySQL-Datenbank + Modbus-Slave, ggf. Oberfläche für Datenerfassung des PEMS
- **AtomS3 Lite #1**: Stromzählerdaten (zwei Leseköpfe)
- **AtomS3 Lite #2**: Averiy P210 
- **Sonoff-Steckdosen**: Nachladung der P210, div. Stromverbräuche von Steckergeräten (ZigBee-Netzwerk)
- **Sonoff-TRV**: Raumtemperatur-Regelung in Räumen mit Radiatoren (ZigBee-Netzwerk)
- **Sonoff-SNZB-02D**: Raumklimaerfassung in allen beheizten Räumen (ZigBee-Netzwerk)
- **Sonoff-SNZB-04P**: Tür- bzw. Fensterkontakte (ZigBee-Netzwerk)
- **Sonoff-MINI-ZB2GS**: Relais zur Raumtemperatur-Regelung in Räumen mit Fußbodenheizung (ZigBee-Netzwerk)
- **Waveshare 2-CAN-TO-ETH**: Vitocharge Vx3 CAN-Bus-Daten
- **Wärmezähler (noch unbekannt)**: Zur Ermittlung der 2 Energieeingänge Wärmepumpe und Kachelofen sowie der 2 Energieausgänge Heizkreis und Frischwasserstation, Effiziensberechnung Sailer-Puffer

## 🎯 Ziel
Langfristig HA ablösen durch Python-Automationen für prädiktives Energiemanagement. Dann soll evtl. der RPi durch einen Industrie-PC ersetzt werden

## 📁 Projektstruktur
- `python-automations/` - Python-Logik für Automationen
- `docs/` - Dokumentation
- `config/` - Konfigurationsdateien
- `docker/` - Docker-Container für Deployment

