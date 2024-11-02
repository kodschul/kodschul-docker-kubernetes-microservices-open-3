
# Übung: Filmreservierungssystem mit REST-API

## Ziel
Erstelle ein Microservice-basiertes System zur Filmreservierung, das REST-APIs zur Kommunikation zwischen den Diensten nutzt. Das System besteht aus drei Services:
1. **Reservation Service**: Nimmt Filmreservierungen entgegen.
2. **Inventory Service**: Verfolgt die Verfügbarkeit von Sitzplätzen für jeden Film.
3. **Notification Service**: Sendet eine Bestätigungsnachricht an den Benutzer nach einer erfolgreichen Reservierung.

---

### Voraussetzungen
1. **Python**: Stelle sicher, dass Python auf deinem Rechner installiert ist. Installiere die benötigten Pakete:
   ```bash
   pip install flask requests
   ```

---

### Anleitung

#### Schritt 1: Implementiere den Inventory Service
- Erstelle eine Datei `inventory_service.py`.
- Implementiere eine API, die verfügbare Sitzplätze für jeden Film anzeigt und Plätze bei einer Reservierung reduziert.

#### Schritt 2: Implementiere den Reservation Service
- Erstelle eine Datei `reservation_service.py`.
- Implementiere eine API zur Annahme von Reservierungsanfragen. Verwende den Inventory Service, um die Platzverfügbarkeit zu prüfen und zu aktualisieren.

#### Schritt 3: Implementiere den Notification Service
- Erstelle eine Datei `notification_service.py`.
- Implementiere eine API, die nach erfolgreicher Reservierung eine Bestätigungsnachricht ausgibt.

#### Schritt 4: Verbinde den Reservation Service mit dem Notification Service
- Aktualisiere den Reservation Service so, dass der Notification Service nach erfolgreicher Reservierung eine Nachricht sendet.

---

### System testen
1. **Verfügbare Plätze prüfen**: Rufe die API des Inventory Service auf, um die verfügbaren Plätze für einen Film zu prüfen.
   ```bash
   curl -X GET http://localhost:5001/inventory/movie_1
   ```

2. **Reservierung aufgeben**: Sende eine POST-Anfrage an den Reservation Service, um einen Sitzplatz zu reservieren.
   ```bash
   curl -X POST -H "Content-Type: application/json" -d '{"movie_id": "movie_1"}' http://localhost:5002/reserve
   ```

3. **Bestätigung prüfen**: Wenn die Reservierung erfolgreich ist, zeigt der Notification Service die Bestätigungsmeldung in der Konsole an.
