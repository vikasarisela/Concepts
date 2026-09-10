# NGINX, HTTP Status Codes & Methods: Comprehensive Troubleshooting Guide

This guide details how to interpret HTTP status codes, use HTTP methods for context, and follow a structured step-by-step framework to diagnose and fix backend application failures running behind an NGINX reverse proxy.

---

## 1. HTTP Status Codes Cheat Sheet

When troubleshooting an active incident, the HTTP status code tells you exactly *where* the request pipeline broke down.

### 4xx Client Error Series (Usually Request or Configuration Issues)
*   **`400 Bad Request`**
    *   **Meaning:** The backend rejected the payload structural format.
    *   **What to check:** Look for syntax errors in JSON payloads, mismatched data types, or missing required fields in the request body.
*   **`401 Unauthorized`**
    *   **Meaning:** The request lacks valid authentication credentials.
    *   **What to check:** Missing, expired, or malformed JWT tokens, Bearer strings, or API keys in the request headers.
*   **`403 Forbidden`**
    *   **Meaning:** The user is authenticated but lacks permission to view the resource, OR NGINX has a file permission issue.
    *   **What to check:** Check user role routing logic. If serving static files, ensure the `nginx` user has OS-level read permissions (`chmod`/`chown`) to the web root folder.
*   **`404 Not Found`**
    *   **Meaning:** The requested URL path does not exist on the server.
    *   **What to check:** Typos in the URL string, incorrect NGINX `root` configuration paths, or missing dynamic endpoints inside the backend code routing tables.
*   **`429 Too Many Requests`**
    *   **Meaning:** The client has breached a rate-limiting threshold.
    *   **What to check:** Check NGINX `limit_req` configurations or application middleware designed to throttle rapid API requests.

### 5xx Server Error Series (Infrastructure or Code Failures)
*   **`500 Internal Server Error`**
    *   **Meaning:** The application server is running, but it crashed while trying to process the code execution.
    *   **What to check:** Look for code exceptions, unhandled promise rejections, null pointers, or broken database queries inside your application logs.
*   **`502 Bad Gateway`**
    *   **Meaning:** NGINX is online, but the backend application process is dead, turned off, or listening on the wrong port entirely.
    *   **What to check:** Application manager statuses (Systemd, PM2, Docker). Ensure the `proxy_pass` port matches your application’s port.
*   **`503 Service Unavailable`**
    *   **Meaning:** The server is alive but temporarily unable to handle the workload.
    *   **What to check:** High server CPU usage or saturated request queues where the backend application cannot keep up with traffic spikes.
*   **`504 Gateway Timeout`**
    *   **Meaning:** NGINX successfully connected to the backend, but the backend application took too long to return a response.
    *   **What to check:** Long-running database queries without proper indexing, frozen background processes, or NGINX `proxy_read_timeout` limits (defaults to 60 seconds).

---

## 2. Troubleshooting Context via HTTP Methods

The HTTP method (verb) dictates what operations the backend application is running. Monitoring the method helps narrow down specific components.

| HTTP Method | Core Function | Primary Troubleshooting Focus |
| :--- | :--- | :--- |
| **`GET`** | Fetch data / load assets. | Watch out for **504 timeouts** caused by heavy database lookups lacking optimized indexes. |
| **`POST`** | Create a new resource. | Watch out for **400 errors** (malformed payload arrays) or NGINX **413 Request Entity Too Large** (uploading files larger than allowed `client_max_body_size`). |
| **`PUT` / `PATCH`** | Update an existing resource. | Watch out for **404 errors** (attempting to alter an ID that does not exist) or **403 errors** (permission errors modifying another user's data). |
| **`DELETE`** | Remove a resource. | Watch out for **500 errors** triggered by database foreign key constraints blocking row deletions. |
| **`OPTIONS`** | Security pre-flight request. | Watch out for **CORS (Cross-Origin Resource Sharing)** blocks. Ensure your headers allow the specific origin in NGINX or app middleware. |

---

## 3. Step-by-Step Server Troubleshooting Blueprint

When an application fails behind NGINX, execute this diagnostic sequence to isolate the point of failure:

[ Client App ] ---> [ NGINX Web Server ] ---> [ Application Server ] ---> [ Database Server ]

### Step 1: Isolate the Response Code
Use your web browser's Developer Tools network tab or run a direct `curl` statement to observe the raw status code.
```bash
curl -I https://yourdomain.com
```
* If it is a **4xx** code: Re-examine client-side payloads and auth headers.
* If it is a **5xx** code: Move immediately to checking infrastructure logs.

### Step 2: Read NGINX Error Logs
NGINX is your front gate. Its logs immediately identify whether the issue is infrastructural or code-based.
```bash
sudo tail -f /var/log/nginx/error.log
```
* Look for **`connect() failed (111: Connection refused)`**: This explicitly means your backend application process is offline (**502 Bad Gateway**). Skip to **Step 4**.
* Look for **`upstream timed out`**: This means your backend is alive but frozen (**504 Gateway Timeout**). Skip to **Step 5**.

### Step 3: Validate NGINX Configuration Changes
If you recently edited a configuration block, test the system syntax for errors before applying updates.
```bash
sudo nginx -t
```
If errors are displayed, fix the designated line numbers and reload NGINX gracefully:
```bash
sudo systemctl reload nginx
```

### Step 4: Verify Backend Process Status
If NGINX logs a "Connection Refused" error, your application process manager needs to be inspected. Check the active status of your runtime layer:
```bash
# For Linux Systemd Services (e.g., Gunicorn, Puma, Go apps)
sudo systemctl status my-backend-service

# For Node.js applications managed via PM2
pm2 status

# For containerized microservices
docker ps
```
If the status shows `inactive`, `failed`, or `errored`, trigger a manual system restart:
```bash
sudo systemctl restart my-backend-service
# OR
pm2 restart all
```

### Step 5: Read Application System Logs
If the backend process is running but passing **500 or 504 errors** back to NGINX, review the deep application log files for exception stack traces.
```bash
# View Linux system journal logs for a service
sudo journalctl -u my-backend-service.service -n 50 --no-pager

# View PM2 live log streaming
pm2 logs

# View Docker container logs
docker logs --tail 50 my-container-name
```
Look for database network exception timeouts, unhandled code breaks, or missing file environment variables (`.env`).

### Step 6: Verify Database Connectivity
If your application stack trace explicitly complains about an inability to reach the persistence layer, verify the host database status:
```bash
# For PostgreSQL databases
sudo systemctl status postgresql

# For MySQL / MariaDB databases
sudo systemctl status mysql

# For MongoDB instances
sudo systemctl status mongod
```
If the database service has crashed due to high memory consumption or storage limitations, clear system resources and restart the database service safely.