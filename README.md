# NATS WebUI

A web interface for monitoring and interacting with NATS messaging system.

## Setup Guide (Windows)

### Prerequisites
- Docker installed
- Git repository cloned

### Step 1: Run NATS Server
```powershell
docker run --name nats-server -p 4222:4222 -p 8222:8222 nats:latest
```

### Step 2: Build NATS WebUI Docker Image
```powershell
docker build -t nats-webui:latest .
```

### Step 3: Create Data Directory
```powershell
mkdir nats-webui
```

### Step 4: Run NATS WebUI Container
```powershell
docker run --rm -p 8080:8600 -v "${PWD}/nats-webui:/data" nats-webui
```

The WebUI will now be accessible at http://localhost:8080

## Connection Tips

To get the Docker container IP address (useful for connecting to NATS server):
```powershell
docker inspect -f '{{.Name}} - {{.NetworkSettings.IPAddress }}' $(docker ps -aq)
```

For monitoring NATS connections:
```powershell
Invoke-RestMethod -Uri "http://localhost:8222/connz?json=true" | ConvertTo-Json
```

## Troubleshooting

### Missing Host Field Error

When encountering:
```
Failed to fetch varz: reqwest::Error { kind: Decode, source: Error("missing field `host`") }
```

The issue was resolved by updating the `ServerVarz` struct in `datatypes.rs` to properly match the actual JSON structure returned by the NATS server monitoring endpoint.

Check varz
```powershell
Invoke-RestMethod -Uri "http://localhost:8222/varz?json=true" | ConvertTo-Json
```

### Note About WebUI Clients

WebUI client connections may not appear in standard `connz` monitoring endpoints, which is expected behavior (not sure what is the fix).

## Publishing Test Messages

To publish messages to subjects that WebUI clients are subscribed to:
```
nats pub [subject] [message]
```


# Original README follows...

---

<p align="center">
  <img src="/screenshots/Logo.png" alt="NATS WebUI Logo"/>
</p>

NATS-WebUI
==========
NATS-WebUI is a web app for monitoring messages on [NATS](https://nats.io/) publications as well as [NATS Server](https://nats.io/) diagnostics. This project was built to explore building web-backends in Rust and uses the following stack:

- HTTP Server and WebSockets in Rust via [Warp Web Framework](https://github.com/seanmonstar/warp) 
- SQLite DB via [rusqlite](https://github.com/jgallagher/rusqlite)
- VueJS
- HTTP Requests via [reqwest](https://github.com/seanmonstar/reqwest)
- [Rants](https://github.com/davidMcneil/rants) Async NATS client

## Screenshots
![Screenshot 4](/screenshots/screenshot4.png) ![Screenshot 3](/screenshots/screenshot3.png)

## Installation
```docker run -d -p 80:80 sphqxe/nats-webui:latest```

## Usage
- Add a server by entering its hostname, port, and monitoring port. The monitoring endpoint is called server-side, so the NATS server host must be resolvable and reachable from the server hosting the WebUI.
- In order to subscribe and receive messages from publications, the subjects must be added to the subject hierarchy on the server dashboard. The hierarchy is represented as a subject tree where each node in the tree contains a single subject token. The editor takes input as a tab-spaced tree. For example, to represent the following subjects:
    ````
    time.us
    time.us.east
    time.us.east.atlanta
    time.eu.east
    time.eu.warsaw
    ````
    Input the subject tree as such:
    ````
    time
      us
        east
          atlanta
      eu
        east
        warsaw
    ````
- Create a client to monitor publications. Once the subjects have been entered as previously stated, they should show up on the right side of the client screen. Select the subjects to subscribe to and click the "link" icon to start receiving messages.

## License
MIT

## Authors
Theodore Lee (@sphqxe)
