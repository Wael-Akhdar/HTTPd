# HTTPd - C Web Server

> **Privacy Note:** The source code of this project is kept private to comply with EPITA's anti-plagiarism regulations. The code is structured, commented, and available upon request for recruiters during an interview. This repository serves as a technical showcase and provides a testable compiled version.

## 📖 Project Context

**HTTPd** is a core EPITA system programming project aimed at building a fully functional HTTP server from scratch using standard C POSIX libraries and the Socket API. The goal is to understand the OSI model's application and transport layers (TCP/IP), manipulate low-level network streams, and manage background processes (daemons).

## ✨ Implemented Features

* **Network Communication:** Native implementation of TCP sockets using `socket`, `bind`, `listen`, and `accept` to handle incoming client connections.
* **HTTP Protocol Handling:** Parses incoming HTTP requests and correctly formats HTTP responses (handling headers, status codes, and payloads). Fully supports `GET` and `HEAD` methods.
* **Daemonization:** The server can run as a background daemon process, detaching from the terminal to run autonomously.
* **Configuration Parsing:** Dynamically loads settings from a `.conf` file. Supports defining log files, PID files, and setting up Virtual Hosts (`[[vhosts]]`) with distinct server names, ports, and root directories.
* **Activity Logging:** Features a comprehensive logging module to trace incoming requests, server errors, and operational statuses.

## 🏗️ Technical Architecture

1.  **Server Lifecycle & Daemon:** The `main` entry point parses CLI arguments and the configuration file. If daemon mode is triggered, it forks the process to detach it from the terminal.
2.  **Socket & Event Loop:** The `server` module continuously listens on the configured ports. Upon accepting a connection, the payload is handed over to the HTTP module.
3.  **HTTP Engine:** * `request.c`: Tokenizes and validates the incoming HTTP raw string.
    * `get.c`: Resolves the requested resource mapped to the server's root directory.
    * `response.c`: Generates the proper HTTP payload (Headers + Body) to send back through the socket.

## 🧠 Technical Challenges Solved

* **String Manipulation:** HTTP headers are plain text and highly variable. Developed a custom string utility module (`src/utils/string`) to safely split, trim, and parse incoming buffers without memory leaks or buffer overflows.
* **Zombie Processes & Signal Handling:** Properly managing the daemon lifecycle requires precise handling of `fork` and UNIX signals (`sigaction`) to gracefully shut down the server and release ports without leaving orphaned processes.

## 🎯 How to test the project?

A pre-compiled version of the server and its configuration wrapper are available for testing. Note that the C binary does not parse `.conf` files directly; it relies on a provided Bash wrapper script.

1. Go to the **[Releases](../../releases)** section and download the `HTTPd_provided_files.tar.gz` archive.
2. Extract the provided files:
```bash
tar -xvf HTTPd_provided_files.tar.gz
```
3. **Terminal 1 - Setup:** Create a simple text file to serve.

```bash
echo "Hello from the first terminal" > hello.txt
```

4.  Terminal 1 - Start the daemon: Use bash to execute the wrapper and start the server in the background.

```bash
chmod +x httpd
bash ./tests/config_reader.sh --path-bin ./httpd --path-config server.conf --daemon start
```
*Expected behavior: The terminal hands back control immediately. A test.pid file is created containing the child process PID.*

5. Terminal 1 - Verify the process: Check that the daemon is actively running.

```bash
cat test.pid
ps -p $(cat test.pid)
```
   
6. Terminal 2 - Test the request: Open a second terminal and fetch the file you created.

```bash
curl -v http://127.0.0.1:8080/hello.txt
```
*Expected behavior: HTTP/1.1 200 OK response displaying "Hello from the first terminal". The connection closes cleanly.*

7. Terminal 1 - Check the logs: Verify that the server successfully recorded the startup and the incoming HTTP request.

```bash
cat test.log
```
*Expected behavior: The output shows timestamped logs for the server initialization and the incoming GET request.*

8. Terminal 1 - Stop the daemon: Go back to the first terminal and shut down the server.

```bash
bash ./tests/config_reader.sh --path-bin ./httpd --path-config server.conf --daemon stop
```

9. Terminal 1 - Verify termination: Ensure the process is completely killed and the PID file is cleaned up.

```bash
cat test.pid
# Expected: cat: test.pid: No such file or directory
```
