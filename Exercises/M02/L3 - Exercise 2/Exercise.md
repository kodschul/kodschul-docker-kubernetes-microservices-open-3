
# Übung: Pizza-Bestellsystem mit Messaging-System

## Ziel
Erstelle ein Microservice-basiertes Pizza-Bestellsystem, das RabbitMQ für die asynchrone Kommunikation nutzt. Das System besteht aus drei Services:
1. **Order Service**: Nimmt Pizza-Bestellungen entgegen.
2. **Kitchen Service**: Bereitet die Pizza zu.
3. **Delivery Service**: Liefert die Pizza aus.

Diese Übung zeigt, wie Messaging-Systeme zur Kommunikation zwischen Diensten in einer Microservices-Architektur verwendet werden können.

---

### Voraussetzungen
1. **RabbitMQ**: Installiere RabbitMQ und stelle sicher, dass es läuft. Die Management-Konsole sollte unter [http://localhost:15672](http://localhost:15672) erreichbar sein.
2. **Python**: Stelle sicher, dass Python auf deinem Rechner installiert ist und installiere die benötigten Pakete:
   ```bash
   pip install flask pika
   ```

---

### Anleitung

#### Schritt 1: RabbitMQ-Server starten
Stelle sicher, dass RabbitMQ lokal läuft.

#### Schritt 2: Implementiere den Order Service
- Erstelle eine Python-Datei mit dem Namen `order_service.py`.
- Verwende Flask, um einen Endpunkt `/order` zu erstellen, der POST-Anfragen mit einer Pizzasorte akzeptiert.
- Bei einer Anfrage soll eine Nachricht mit den Bestelldetails an die RabbitMQ-Warteschlange `pizza_orders` gesendet werden.

#### Schritt 3: Implementiere den Kitchen Service
- Erstelle eine Python-Datei mit dem Namen `kitchen_service.py`.
- Setze einen Listener auf die Warteschlange `pizza_orders`.
- Wenn eine Bestellung eingeht, simuliere die Pizzazubereitung, indem die Bestellung ausgegeben und 5 Sekunden gewartet wird.
- Sende anschließend eine Nachricht an die Warteschlange `pizza_ready` in RabbitMQ.

#### Schritt 4: Implementiere den Delivery Service
- Erstelle eine Python-Datei mit dem Namen `delivery_service.py`.
- Setze einen Listener auf die Warteschlange `pizza_ready`.
- Wenn eine „fertige Pizza“-Nachricht empfangen wird, simuliere die Auslieferung der Pizza, indem die Nachricht ausgegeben und 3 Sekunden gewartet wird.

---

### Testen
1. Starte jeden Service in einem eigenen Terminal.
2. Verwende `curl` oder einen REST-Client, um eine POST-Anfrage an den Order Service zu senden:
   ```bash
   curl -X POST -H "Content-Type: application/json" -d '{"pizza_type": "Pepperoni"}' http://localhost:5000/order
   ```
3. Beobachte die Konsolenausgaben in den verschiedenen Terminals, um den Bestellablauf zu verfolgen:
   - Der Order Service sollte die Bestellung empfangen und weiterleiten.
   - Der Kitchen Service sollte die Bestellung zubereiten und als fertig markieren.
   - Der Delivery Service sollte die Pizza ausliefern.
