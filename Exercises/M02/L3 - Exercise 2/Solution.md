
# Lösung für das Pizza-Bestellsystem mit Messaging-System

## Voraussetzungen
1. **RabbitMQ**: Installiere RabbitMQ lokal auf deinem Rechner und stelle sicher, dass es läuft. Die Management-Konsole ist unter [http://localhost:15672](http://localhost:15672) erreichbar.
2. **Python**: Stelle sicher, dass Python installiert ist, und installiere die benötigten Bibliotheken mit:
   ```bash
   pip install flask pika
   ```

---

### Schritt 1: RabbitMQ-Server starten
Stelle sicher, dass RabbitMQ lokal läuft. Falls nicht, starte es manuell über den Dienstmanager oder die Befehlszeile deines Systems.

---

### Schritt 2: Implementiere den Order Service
Der Order Service nimmt Bestellungen entgegen und sendet eine Nachricht an die Warteschlange `pizza_orders` in RabbitMQ.

**Code** (order_service.py):
```python
import pika
from flask import Flask, request, jsonify

app = Flask(__name__)

def send_order_message(order):
    connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
    channel = connection.channel()
    channel.queue_declare(queue='pizza_orders')
    channel.basic_publish(exchange='', routing_key='pizza_orders', body=order)
    connection.close()

@app.route('/order', methods=['POST'])
def create_order():
    data = request.get_json()
    pizza_type = data.get('pizza_type', 'Margherita')
    order = f"Order for {pizza_type} pizza"
    send_order_message(order)
    return jsonify({'status': 'Order received!', 'order': order}), 201

if __name__ == '__main__':
    app.run(port=5000, debug=True)
```

Starte den Order Service mit:
```bash
python order_service.py
```

---

### Schritt 3: Implementiere den Kitchen Service
Der Kitchen Service empfängt Bestellnachrichten aus der Warteschlange `pizza_orders`, bereitet die Pizza vor und sendet eine Nachricht an die Warteschlange `pizza_ready`, sobald die Pizza fertig ist.

**Code** (kitchen_service.py):
```python
import pika
import time

def prepare_pizza(ch, method, properties, body):
    print(f"Preparing: {body.decode()}")
    time.sleep(5)
    print(f"{body.decode()} is ready!")

    connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
    channel = connection.channel()
    channel.queue_declare(queue='pizza_ready')
    channel.basic_publish(exchange='', routing_key='pizza_ready', body=body)
    connection.close()

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='pizza_orders')
channel.basic_consume(queue='pizza_orders', on_message_callback=prepare_pizza, auto_ack=True)

print('Waiting for orders...')
channel.start_consuming()
```

Starte den Kitchen Service mit:
```bash
python kitchen_service.py
```

---

### Schritt 4: Implementiere den Delivery Service
Der Delivery Service hört auf der Warteschlange `pizza_ready` und simuliert die Auslieferung der Pizza, wenn eine Nachricht empfangen wird.

**Code** (delivery_service.py):
```python
import pika
import time

def deliver_pizza(ch, method, properties, body):
    print(f"Delivering: {body.decode()}")
    time.sleep(3)
    print(f"{body.decode()} delivered to customer!")

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='pizza_ready')
channel.basic_consume(queue='pizza_ready', on_message_callback=deliver_pizza, auto_ack=True)

print('Waiting for ready pizzas...')
channel.start_consuming()
```

Starte den Delivery Service mit:
```bash
python delivery_service.py
```

---

### System testen
Um das System zu testen:
1. Sende eine POST-Anfrage, um eine Pizza-Bestellung aufzugeben:
   ```bash
   curl -X POST -H "Content-Type: application/json" -d '{"pizza_type": "Pepperoni"}' http://localhost:5000/order
   ```
2. Überprüfe die Konsolenausgabe der einzelnen Services, um den Bestellablauf zu beobachten:
   - Der Order Service nimmt die Bestellung entgegen und leitet sie weiter.
   - Der Kitchen Service bereitet die Pizza zu und markiert sie als fertig.
   - Der Delivery Service erhält die fertige Pizza und liefert sie aus.

