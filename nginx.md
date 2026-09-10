# NGINX Notes

## 1. What is NGINX?

NGINX is a web server and reverse proxy software.

It can perform multiple roles:

- Web server
- Reverse proxy
- Load balancer
- SSL/TLS termination
- Static file server
- HTTP cache

---

# 2. NGINX as a Web Server

NGINX can directly serve static files such as:

- HTML
- CSS
- JavaScript
- Images
- Videos

Example:

/var/www/html/

    index.html
    style.css
    app.js

Request:

Browser
   |
   | GET /index.html
   ↓
 NGINX
   |
   ↓
index.html


NGINX reads the file and sends it back to the browser.

### Important

NGINX itself is not the frontend application.

It can SERVE the frontend application files.

---

# 3. NGINX as a Reverse Proxy

NGINX can sit in front of backend servers.

Client
   |
   ↓
 NGINX
   |
   ↓
Backend Server


Example:

Browser requests:

https://example.com/api/users

NGINX receives the request.

NGINX forwards it to:

Backend Server
10.0.0.20:8080

The backend response comes back:

Backend
   ↓
 NGINX
   ↓
Client


## Why is it called Reverse Proxy?

Because the proxy sits in front of the servers and represents/protects the backend servers.

Forward Proxy:

Client → Forward Proxy → Internet

Reverse Proxy:

Internet → Reverse Proxy → Backend Servers


### Easy memory

Forward Proxy
→ represents CLIENT

Reverse Proxy
→ represents SERVER

---

# 4. Backend Server Hiding

The client normally communicates with NGINX instead of directly communicating with the backend.

Example:

Client
   |
   ↓
NGINX
   |
   ↓
Backend Server
10.0.0.20:8080


The backend server can be kept private.

The client only needs to know:

example.com

It does not need to know:

10.0.0.20:8080


### Important

"Hiding the backend server" is a benefit of a reverse proxy.

The main definition is:

NGINX receives client requests and forwards them to backend servers.

---

# 5. NGINX as Frontend Server

If NGINX serves frontend files directly:

Browser
   |
   ↓
NGINX
   |
   ↓
HTML/CSS/JS


Example:

/var/www/html/index.html

NGINX can serve:

- index.html
- style.css
- app.js
- images


You can say:

"NGINX is serving the frontend application."

Do NOT say NGINX itself is the frontend application.

---

# 6. NGINX Serving Frontend + Reverse Proxy

NGINX can do both at the same time.

Example:

                  NGINX
                 /     \
                /       \
               ↓         ↓
        Frontend       Backend API
        HTML/CSS/JS    Node/Java/Python


Request:

https://example.com/

        ↓

NGINX

        ↓

index.html


API request:

https://example.com/api/users

        ↓

NGINX

        ↓

Backend application


This is very common in production.

---

# 7. NGINX as Load Balancer

NGINX can distribute traffic across multiple backend servers.

Client
   |
   ↓
 NGINX
   |
   +------→ Backend 1
   |
   +------→ Backend 2
   |
   +------→ Backend 3


Example:

upstream backend {
    server 10.0.0.11;
    server 10.0.0.12;
    server 10.0.0.13;
}


NGINX can distribute requests among these servers.

---

# 8. NGINX Load-Balancing Methods

## Round Robin

Default method.

Requests are distributed sequentially.

Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
Request 5 → Server 2


## Least Connections

Sends the request to the server with the fewest active connections.

Example:

Server 1 → 10 connections
Server 2 → 3 connections
Server 3 → 7 connections

New request → Server 2


Configuration:

upstream backend {
    least_conn;

    server 10.0.0.11;
    server 10.0.0.12;
    server 10.0.0.13;
}


## IP Hash

Uses the client's IP address to determine the backend server.

Useful when requests from the same client should generally go to the same backend.

Configuration:

upstream backend {
    ip_hash;

    server 10.0.0.11;
    server 10.0.0.12;
}


---

# 9. NGINX Configuration

Main configuration:

/etc/nginx/nginx.conf


Common configuration structure:

http {

    server {

        location / {

        }

    }

}


## server block

Defines a virtual server.

Example:

server {
    listen 80;
    server_name example.com;

    location / {
        root /var/www/html;
        index index.html;
    }
}


## location block

Defines how NGINX handles a particular URL path.

Example:

location /api/ {
    proxy_pass http://backend;
}


---

# 10. Reverse Proxy Configuration

Example:

upstream backend {
    server 10.0.0.11:8080;
    server 10.0.0.12:8080;
}

server {

    listen 80;

    server_name example.com;

    location /api/ {

        proxy_pass http://backend;

    }
}


Flow:

Client
   ↓
example.com/api/users
   ↓
NGINX
   ↓
Backend 1 or Backend 2

---

# 11. Static Frontend + Backend API

Example:

server {

    listen 80;

    server_name example.com;


    # Frontend
    location / {

        root /var/www/html;

        index index.html;

    }


    # Backend API
    location /api/ {

        proxy_pass http://backend;

    }

}


Flow:

/ 
 ↓
NGINX
 ↓
Frontend files


/api/
 ↓
NGINX
 ↓
Backend application

---

# 12. NGINX and SSL/TLS

NGINX can terminate HTTPS connections.

Client
   |
   | HTTPS
   ↓
NGINX
   |
   | HTTP/HTTPS
   ↓
Backend


This is called:

SSL/TLS Termination


The client establishes the HTTPS connection with NGINX.

NGINX can then communicate with the backend.

---

# 13. NGINX Architecture

Typical production architecture:

                    Internet
                       |
                       ↓
                    NGINX
                 Reverse Proxy
                 Load Balancer
                 SSL Termination
                  /          \
                 ↓            ↓
           Frontend       Backend
           Static files    Application
                              |
                              ↓
                           Database


---

# 14. NGINX vs Application Server

NGINX:

- Web server
- Reverse proxy
- Load balancer
- Static file server

Application server:

- Runs application code
- Business logic
- API processing

Examples of application servers/frameworks:

- Node.js
- Java/Spring Boot
- Python
- .NET


Example:

Client
  ↓
NGINX
  ↓
Node.js application
  ↓
Database


---

# 15. Important NGINX Commands

Check NGINX version:

nginx -v


Test configuration:

nginx -t


Reload configuration:

systemctl reload nginx


Restart NGINX:

systemctl restart nginx


Start NGINX:

systemctl start nginx


Stop NGINX:

systemctl stop nginx


Check status:

systemctl status nginx


Enable at boot:

systemctl enable nginx


---

# 16. Important Ports

HTTP:

80


HTTPS:

443


Example:

Client → NGINX:443 → Backend:8080


---

# 17. NGINX Logs

Common log locations:

/var/log/nginx/access.log

/var/log/nginx/error.log


## access.log

Records client requests.

Example:

GET /index.html


## error.log

Records NGINX errors and problems.

Useful when troubleshooting:

- 502 Bad Gateway
- 504 Gateway Timeout
- Configuration errors
- Connection problems

---

# 18. Common HTTP Errors

## 404 Not Found

Requested resource does not exist.

Example:

Client → NGINX → File not found

Usually check:

- URL
- root directory
- file name
- location configuration


## 502 Bad Gateway

NGINX cannot get a valid response from the backend.

Possible reasons:

- Backend application is down
- Wrong backend IP/port
- Firewall issue
- Backend not listening


## 504 Gateway Timeout

NGINX waited for the backend but did not receive a response within the timeout.

Possible reasons:

- Backend is slow
- Backend is overloaded
- Network problem
- Timeout configuration


---

# 19. NGINX Request Flow

When a client sends a request:

1. Client sends request
2. Request reaches NGINX
3. NGINX checks server configuration
4. NGINX matches the URL/location
5. NGINX decides what to do
6. It may:
   - Serve a static file
   - Forward request to backend
   - Return an error
   - Load-balance to a backend
7. Response is sent back to client


---

# 20. Forward Proxy vs Reverse Proxy

## Forward Proxy

Client
  ↓
Forward Proxy
  ↓
Internet


Proxy represents the client.

Example:

Company employees → Corporate Proxy → Internet


## Reverse Proxy

Internet
  ↓
Reverse Proxy
  ↓
Backend Servers


Proxy represents/protects the servers.

Example:

Users → NGINX → Application Servers


---

# 21. Most Important Interview Points

### What is NGINX?

NGINX is a web server and reverse proxy that can also perform load balancing and serve static content.

### What is a reverse proxy?

A reverse proxy sits in front of backend servers, receives client requests, and forwards them to the appropriate backend.

### Why use NGINX?

- Reverse proxy
- Load balancing
- Static content serving
- SSL termination
- Security
- Caching
- Routing

### Can NGINX serve HTML?

Yes.

NGINX can directly serve static HTML, CSS, JavaScript, images, etc.

### Can NGINX act as a load balancer?

Yes.

It supports load-balancing methods such as:

- Round Robin
- Least Connections
- IP Hash

### Can NGINX serve frontend and proxy backend?

Yes.

NGINX can serve frontend static files directly and forward API requests to backend application servers.

---

# One-Line Memory

NGINX = Web Server + Reverse Proxy + Load Balancer

Forward Proxy → represents CLIENT

Reverse Proxy → represents SERVER

NGINX can serve frontend files directly.

NGINX can forward API requests to backend servers.

NGINX can distribute traffic across multiple backend servers.

NGINX is software; Reverse Proxy and Load Balancer are roles/functions it can perform.