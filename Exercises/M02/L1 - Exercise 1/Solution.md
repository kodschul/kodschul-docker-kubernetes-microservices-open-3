
# Lösung für das Filmreservierungs-Microservice-System mit REST-API

## Voraussetzungen
1. **Python**: Stellen Sie sicher, dass Python auf Ihrem Rechner installiert ist und die erforderlichen Bibliotheken verfügbar sind. Installieren Sie die Pakete mit:
   ```bash
   pip install flask requests
   ```

---

### Schritt 1: Implementiere den Inventory Service
Der Inventory Service verwaltet die Anzahl verfügbarer Plätze für verschiedene Filme und bietet REST-API-Endpunkte zur Überprüfung und Aktualisierung der Sitzplatzverfügbarkeit.

**Code** (inventory_service.py):
```python
from flask import Flask, jsonify, request

app = Flask(__name__)

# Beispiel-Datenbank für Filme und verfügbare Plätze
movies = {
    "movie_1": {"title": "Inception", "available_seats": 5},
    "movie_2": {"title": "Interstellar", "available_seats": 3},
    "movie_3": {"title": "Dunkirk", "available_seats": 8},
}

@app.route('/inventory/<movie_id>', methods=['GET'])
def get_movie_inventory(movie_id):
    movie = movies.get(movie_id)
    if movie:
        return jsonify(movie), 200
    else:
        return jsonify({"error": "Movie not found"}), 404

@app.route('/inventory/<movie_id>/reserve', methods=['POST'])
def reserve_seat(movie_id):
    movie = movies.get(movie_id)
    if movie and movie["available_seats"] > 0:
        movie["available_seats"] -= 1
        return jsonify({"message": "Seat reserved", "remaining_seats": movie["available_seats"]}), 200
    elif movie:
        return jsonify({"error": "No seats available"}), 400
    else:
        return jsonify({"error": "Movie not found"}), 404

if __name__ == '__main__':
    app.run(port=5001, debug=True)
```

Starten Sie den Inventory Service:
```bash
python inventory_service.py
```

---

### Schritt 2: Implementiere den Reservation Service
Der Reservation Service bietet eine API zur Reservierung und verwendet den Inventory Service, um die Verfügbarkeit zu überprüfen.

**Code** (reservation_service.py):
```python
import requests
from flask import Flask, jsonify, request

app = Flask(__name__)

INVENTORY_SERVICE_URL = "http://localhost:5001"

@app.route('/reserve', methods=['POST'])
def create_reservation():
    data = request.get_json()
    movie_id = data.get("movie_id")

    # Überprüfe die Platzverfügbarkeit
    response = requests.post(f"{INVENTORY_SERVICE_URL}/inventory/{movie_id}/reserve")

    if response.status_code == 200:
        return jsonify({"message": "Reservation successful", "movie_id": movie_id}), 201
    elif response.status_code == 400:
        return jsonify({"error": "No seats available"}), 400
    else:
        return jsonify({"error": "Movie not found"}), 404

if __name__ == '__main__':
    app.run(port=5002, debug=True)
```

Starten Sie den Reservation Service:
```bash
python reservation_service.py
```

---

### Schritt 3: Implementiere den Notification Service
Der Notification Service empfängt eine Anfrage vom Reservation Service bei erfolgreicher Reservierung und sendet eine Bestätigungsmeldung.

**Code** (notification_service.py):
```python
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route('/notify', methods=['POST'])
def send_notification():
    data = request.get_json()
    movie_id = data.get("movie_id")
    message = f"Your reservation for movie ID {movie_id} has been confirmed!"
    print(message)
    return jsonify({"message": message}), 200

if __name__ == '__main__':
    app.run(port=5003, debug=True)
```

Starten Sie den Notification Service:
```bash
python notification_service.py
```

---

### Schritt 4: Verbinde den Reservation Service mit dem Notification Service
Aktualisieren Sie `reservation_service.py`, sodass der Notification Service nach erfolgreicher Reservierung aufgerufen wird.

```python
import requests
from flask import Flask, jsonify, request

app = Flask(__name__)

INVENTORY_SERVICE_URL = "http://localhost:5001"
NOTIFICATION_SERVICE_URL = "http://localhost:5003"

@app.route('/reserve', methods=['POST'])
def create_reservation():
    data = request.get_json()
    movie_id = data.get("movie_id")

    response = requests.post(f"{INVENTORY_SERVICE_URL}/inventory/{movie_id}/reserve")

    if response.status_code == 200:
        requests.post(f"{NOTIFICATION_SERVICE_URL}/notify", json={"movie_id": movie_id})
        return jsonify({"message": "Reservation successful", "movie_id": movie_id}), 201
    elif response.status_code == 400:
        return jsonify({"error": "No seats available"}), 400
    else:
        return jsonify({"error": "Movie not found"}), 404

if __name__ == '__main__':
    app.run(port=5002, debug=True)
```

---

### System testen
1. **Verfügbare Plätze prüfen**:
   ```bash
   curl -X GET http://localhost:5001/inventory/movie_1
   ```

2. **Reservierung aufgeben**:
   ```bash
   curl -X POST -H "Content-Type: application/json" -d '{"movie_id": "movie_1"}' http://localhost:5002/reserve
   ```

3. **Bestätigung prüfen**: Wenn die Reservierung erfolgreich ist, zeigt der Notification Service die Bestätigung in der Konsole an.
