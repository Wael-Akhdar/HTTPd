# HTTPd - C Web Server

> **Privacy Note:** The source code of this project is kept private to comply with EPITA's anti-plagiarism regulations. The code is structured, commented, and available upon request for recruiters during an interview. This repository serves as a technical showcase and provides a testable compiled version.

## 📖 Project Context

**HTTPd** is a core EPITA system programming project aimed at building a fully functional HTTP server from scratch using standard C POSIX libraries and the Socket API. The goal is to understand the OSI model's application and transport layers (TCP/IP), manipulate low-level network streams, and manage background processes (daemons).

## ✨ Implemented Features

* **Network Communication:** Native implementation of TCP sockets using `socket`, `bind`, `listen`, and `accept` to handle incoming client connections.
* **HTTP Protocol Handling:** Parses incoming HTTP requests and correctly formats HTTP responses (handling headers, status codes, and payloads). Currently supports `GET` methods.
* **Daemonization:** The server can run as a background daemon process. It supports standard control commands via the CLI to `start`, `stop`, and `restart` the daemon smoothly.
* **Configuration Parsing:** Dynamically loads settings from a configuration file. Supports defining log files, PID files, and setting up multiple Virtual Hosts (`[[vhosts]]`) with distinct server names and ports.
* **Activity Logging:** Features a comprehensive logging module to trace incoming requests, server errors, and operational statuses.

## 🏗️ Technical Architecture

1.  **Server Lifecycle & Daemon:** The `main` entry point parses CLI arguments and the configuration file. If requested, it forks the process to detach it from the terminal (daemon mode).
2.  **Socket & Event Loop:** The `server` module continuously listens on the configured ports. Upon accepting a connection, the payload is handed over to the HTTP module.
3.  **HTTP Engine:** * `request.c`: Tokenizes and validates the incoming HTTP raw string.
    * `get.c`: Resolves the requested resource mapped to the server's root directory.
    * `response.c`: Generates the proper HTTP payload (Headers + Body) to send back through the socket.

## 🧠 Technical Challenges Solved

* **String Manipulation:** HTTP headers are plain text and highly variable. Developed a custom string utility module (`src/utils/string`) to safely split, trim, and parse incoming buffers without memory leaks or buffer overflows.
* **Zombie Processes & Signal Handling:** Properly managing the daemon lifecycle requires precise handling of `fork` and UNIX signals (`sigaction`) to gracefully shut down the server and release ports without leaving orphaned processes.

## 🎯 How to test the project?

A pre-compiled version of the server is available for testing.

1.  Go to the **[Releases](../../releases)** section.
2.  Download the archive containing the `httpd` executable and an example configuration file.
3.  Open a terminal and start the server:

```bash
chmod +x httpd
# Start the server in daemon mode
./httpd --start -c server.conf

# Test it via curl or your browser
curl http://localhost:4242/

# Stop the server
./httpd --stop
