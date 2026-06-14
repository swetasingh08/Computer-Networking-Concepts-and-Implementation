

---

# Image Transfer Over TCP Sockets

A Python client-server application that demonstrates reliable binary file transfer over TCP sockets by transmitting an image from a client to a server using a length-prefixed protocol.

---

## Overview

This project implements a simple but robust image transfer mechanism using TCP stream sockets. Unlike simple text-based protocols, transferring binary data like images requires careful handling of message boundaries. This implementation uses a length-prefix approach: the client first sends the size of the image as a 4-byte integer, followed by the raw image bytes. The server uses this size information to know exactly how much data to receive before processing the complete image.

---

## Quick Start

### Prerequisites

- Python 3.7+
- Pillow (PIL) library for image handling

### Installation

```bash
# Clone the repository
git clone https://github.com/your-username/image-transfer-tcp.git
cd image-transfer-tcp

# Install dependencies
pip install Pillow
```

### Running the Simulation

```bash
# Start the server in one terminal
python server.py

# In another terminal, run the client
python client.py
```

The client will read a local image file, transmit it to the server, and the server will display and save the received image.

---

## How It Works

### Protocol Design

TCP is a stream-oriented protocol, meaning data arrives as a continuous stream of bytes with no built-in message boundaries. A single send() on the client side may correspond to multiple recv() calls on the server side, and vice versa. This creates a challenge: how does the server know when it has received the complete image?

This implementation solves the problem using a length-prefix protocol:

```
+------------------+------------------------------------+
|  4-byte size (I) |  Image data (size bytes)            |
+------------------+------------------------------------+
```

The struct module packs the image size as a network byte order (big-endian) unsigned integer using the format '!I', ensuring cross-platform compatibility.

### Client (client.py)

The client performs the following steps:

1. Opens and reads the image file in binary mode
2. Creates a TCP socket and connects to the server at localhost:4000
3. Packs the length of the image data into a 4-byte big-endian integer using struct.pack('!I', len(img_data))
4. Sends the 4-byte size header first using sendall() to ensure complete delivery
5. Sends the raw image bytes using sendall() for reliable transmission
6. Closes the connection

Key design decisions:
- sendall() is used instead of send() to guarantee all data is transmitted, handling partial writes internally
- The file path is absolute; modify this to match your system or use a relative path for portability

### Server (server.py)

The server performs the following steps:

1. Creates a TCP socket, binds to localhost:4000, and listens for incoming connections
2. Accepts a client connection
3. Receives exactly 4 bytes for the size header
4. Unpacks the size using struct.unpack('!I', size_data)[0]
5. Loops to receive data in 4096-byte chunks until the total received matches the expected size
6. Wraps the accumulated bytes in a BytesIO object
7. Opens and displays the image using PIL/Pillow
8. Saves the received image to disk
9. Closes the client connection and the server socket

Key design decisions:
- The server receives data in a loop, accumulating chunks until the total matches the expected size — this handles TCP fragmentation correctly
- BytesIO allows treating the raw bytes as a file-like object that PIL can read directly
- The received image is both displayed on screen and saved to disk

---

## Data Flow Diagram

```
Client                                    Server
  |                                         |
  |-- Read image file (binary)              |
  |                                         |
  |-- TCP Connect (localhost:4000) -------> |
  |                                         |
  |-- Send 4-byte size header ------------> |-- Receive size
  |   (struct.pack: 245,832 bytes)          |   (struct.unpack: expect 245832 bytes)
  |                                         |
  |-- Send image bytes ------------------> |-- Receive in 4KB chunks
  |   (sendall: 245,832 bytes)              |   (loop until 245832 bytes collected)
  |                                         |
  |                                         |-- Wrap in BytesIO
  |                                         |-- Open with PIL
  |                                         |-- Display image
  |                                         |-- Save to disk
  |                                         |
  |-- TCP Disconnect ---------------------> |-- Close connection
```

---

## Technical Details

### Handling TCP's Stream Nature

TCP does not preserve message boundaries. A naive implementation that simply calls recv() once would likely receive only a partial image. The length-prefix pattern solves this by:

1. Establishing a fixed-size header that the server can reliably read first (always 4 bytes)
2. Using the extracted size to determine when the complete payload has been received
3. Looping with recv() until the accumulated data matches the expected size

### Binary Mode and Encoding

Unlike text-based protocols, image files contain arbitrary bytes that cannot be decoded as UTF-8 or ASCII. The file is opened with 'rb' (read binary) mode, and all socket operations use raw bytes without encoding/decoding.

### Why struct.pack with '!I'?

- '!' specifies network byte order (big-endian), ensuring consistent interpretation regardless of the platform's native byte order
- 'I' specifies a 4-byte unsigned integer
- This allows file sizes up to approximately 4 GB (2^32 - 1 bytes)

---

## Error Handling Considerations

The current implementation is minimal and suitable for demonstration purposes. For production use, consider adding:

- Try-except blocks around socket operations to handle connection failures
- Timeout handling for stalled connections
- Validation that the received data is a valid image format
- Support for multiple concurrent clients using threading
- Graceful handling of disk full errors when saving the image
- Checksum verification to detect data corruption during transfer

---

## Sample Output

Server console:

```
Server waiting for image...
Client connected: ('127.0.0.1', 54321)
Image Size: 240 KB
```

The received image will open in your default image viewer, and a copy will be saved to the specified path.

Client console:

```
Client is running...
Sending image to server...
Image sent to server.
```

---


