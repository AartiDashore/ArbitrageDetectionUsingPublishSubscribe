# Detecting Arbitrage Opportunities Using Published Quotes

## Overview

In this lab, we will build a system that listens to real-time foreign exchange (forex) rates from a price feed and detects arbitrage opportunities. Arbitrage refers to the process of making a profit from price differences in different markets. The system will subscribe to a forex price feed and, whenever a profitable arbitrage opportunity is available, it will print a message indicating the opportunity.

## Key Concepts

### Foreign Exchange Markets
Foreign exchange markets (Forex) facilitate the exchange of one currency for another. For example, you can trade USD for GBP, and then trade GBP for JPY, and so on. In an efficient market, such exchange opportunities should not exist, as the prices should equilibrate quickly. However, for this lab, we are tasked with detecting arbitrage, which happens when currency conversion results in a profit.

### Price Feed Format
The forex prices are published in real-time as UDP/IP datagrams. The message format contains the following fields:
- **Currency 1, Currency 2**: ISO 3-character currency codes (e.g., USD, GBP, EUR, etc.)
- **Exchange Rate**: The number of units of Currency 2 per 1 unit of Currency 1.
- **Timestamp**: The timestamp in microseconds since the Unix epoch.

Each message may contain multiple quotes, with new quotes superseding older ones if they are more recent.

### Arbitrage Detection
Arbitrage opportunities are detected by modeling the currencies and forex markets as a graph:
- **Nodes** represent currencies.
- **Edges** represent forex market exchanges, weighted by the **negative logarithms** of the exchange rates.

Using the **Bellman-Ford algorithm**, we can identify **negative-weight cycles**, which indicate arbitrage opportunities.

### Steps to Build the System

1. **Subscribe to the Forex Price Feed**: 
   - The system subscribes to the forex price feed via a UDP socket, providing its IP address and port.
   - The subscription lasts for 10 minutes or until the system hasn't received any messages for over 1 minute.

2. **Process Published Quotes**:
   - Upon receiving a forex quote, the system updates its graph with the exchange rate data.
   - The graph edges are weighted with the negative logarithm of the exchange rate.

3. **Detect Arbitrage**:
   - Use the Bellman-Ford algorithm to check for negative cycles in the graph.
   - Report the arbitrage opportunity by showing the series of trades that lead to a profit.

4. **Handle Out-of-Order Messages**: 
   - Since quotes may arrive out of order, the system should only use the most recent quotes (within a 1.5-second window).

## Project Structure

### Files
- `fxp_bytes_subscriber.py`: The subscriber-side version of the functions for constructing and decoding quotes.
- `lab3.py`: The main class and driver that runs the process of subscribing, receiving, processing quotes, and detecting arbitrage opportunities.

### Dependencies
- **Bellman-Ford Algorithm**: A class implementing the Bellman-Ford algorithm to detect negative cycles. The professor's implementation `bellman_ford.py` should be used.
- **Forex Provider**: You can run a local version of the forex provider using the provided `forex_provider.py`.

### Example Log

Here’s a sample log of running the subscriber:

```
2019-10-14 23:58:30.188639 AUD USD 0.75035
2019-10-14 23:58:30.188639 USD CHF 1.0016
2019-10-14 23:58:30.188639 USD JPY 100.04957
2019-10-14 23:58:30.188639 EUR USD 1.1002
2019-10-14 23:58:30.188639 GBP USD 1.2516
2019-10-14 23:58:31.194879 AUD CAD 0.30038324044194714
2019-10-14 23:58:31.194879 CAD GBP 1.2015329617677886
ARBITRAGE:
    Start with USD 100
    Exchange USD for GBP at 0.7989773090444231 --> GBP 79.89773090444231
    Exchange GBP for CAD at 0.8322701347524601 --> CAD 66.496495266256
    Exchange CAD for AUD at 3.3290805390098406 --> AUD 221.37218830325284
    Exchange AUD for USD at 0.75035 --> USD 166.10662149334576
```

## Implementation Details

### 1. Subscribing to the Forex Provider

To start receiving forex price quotes, the subscriber needs to send its IP and port to the Forex provider:

```python
import socket

def subscribe_to_forex_feed(ip, port):
    server_address = (ip, port)
    with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as sock:
        sock.bind(server_address)
        print(f"Subscribed to Forex provider at {ip}:{port}")
        while True:
            data = sock.recv(4096)
            print(f"Received data: {data}")
```

### 2. Processing Received Quotes

The forex quotes are decoded into their components (currencies, exchange rate, and timestamp) and stored in a graph structure.

```python
import struct
import math
from collections import defaultdict

class ForexSubscriber:
    def __init__(self):
        self.graph = defaultdict(list)  # Stores the graph as an adjacency list
        self.latest_timestamps = {}  # Stores the latest timestamp for each market

    def process_quote(self, data):
        currency1, currency2, rate, timestamp = decode_quote(data)
        if self.is_valid_quote(currency1, currency2, timestamp):
            self.update_graph(currency1, currency2, rate)
    
    def update_graph(self, currency1, currency2, rate):
        # Add both the direct and reciprocal exchange rate
        log_rate = -math.log(rate)
        self.graph[currency1].append((currency2, log_rate))
        self.graph[currency2].append((currency1, -log_rate))
        
    def is_valid_quote(self, currency1, currency2, timestamp):
        # Check if the quote is recent enough and valid
        pass
```

### 3. Detecting Arbitrage

Using the Bellman-Ford algorithm, we search for negative-weight cycles, indicating arbitrage opportunities.

```python
def detect_arbitrage(graph):
    # Bellman-Ford algorithm implementation
    pass
```

### 4. Reporting Arbitrage Opportunities

When a negative cycle is detected, the system should print the sequence of trades involved in the arbitrage.

## Conclusion

This project will allow you to simulate a real-time forex market, detect arbitrage opportunities using a graph-based approach, and report those opportunities as they arise. By using the Bellman-Ford algorithm, we can efficiently detect negative-weight cycles that represent arbitrage profits.
