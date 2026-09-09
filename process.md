# Blueprint: Deploying Linux Services Privately and Securely

The fundamental architecture for securing and running production backend software on Linux remains **identical across almost all languages and frameworks**. Whether you are deploying Node.js, Python, Java, or Go, you follow the same core methodology under **systemd**.

---

## 🏗️ The Universal 5-Step Process

Regardless of the technology stack, deploying a secure background service always follows these exact steps:

1. **Isolate the Identity:** Create a dedicated, locked-down system user (`/bin/false`) with no login privileges.
2. **Restrict the Directory:** Grant that user ownership *only* over the application's specific folder.
3. **Define the Service:** Write a systemd unit configuration file (`/etc/systemd/system/your-app.service`).
4. **Declare the Runtime:** Tell systemd which `User` runs it, the `WorkingDirectory`, and the specific execution command.
5. **Enforce Background Execution:** Start and enable the service to survive system reboots.

---

## 🔄 Stack-Specific Modifications (The `ExecStart` Config)

While the system setup steps remain identical, only two specific lines inside the **systemd service file** (`WorkingDirectory` and `ExecStart`) change based on the language or framework you are using:

### 1. Node.js
Uses the Node interpreter to execute a JavaScript entry file.
```ini
WorkingDirectory=/var/www/my-node-app
ExecStart=/usr/bin/node server.js
```

### 2. Python (e.g., FastAPI / Flask)
Points to the virtual environment's specific Python or application server executable.
```ini
WorkingDirectory=/var/www/my-python-app
ExecStart=/var/www/my-python-app/venv/bin/gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app
```

### 3. Java (e.g., Spring Boot)
Executes a compiled Java Archive (`.jar`) using the Java Runtime Environment.
```ini
WorkingDirectory=/var/www/my-java-app
ExecStart=/usr/bin/java -jar my-application.jar
```

### 4. Go (Golang) or Rust
Compiles directly into native machine code binaries. Because there is no interpreter runtime required, you execute the binary directly.
```ini
WorkingDirectory=/var/www/my-go-app
ExecStart=/var/www/my-go-app/server-binary
```

---

## 📦 The Modern Container Alternative (Docker)

In modern containerized environments, you may bypass systemd entirely. However, **the underlying security concept remains identical**. You drop root administrative privileges and run the app under a non-human user directly inside the container using a `Dockerfile`:

```dockerfile
# 1. Start with the software runtime base image
FROM node:20

# 2. Setup the application directory
WORKDIR /app
COPY . .

# 3. Drop root privileges and switch to a built-in, unprivileged system user
USER node

# 4. Start the application process
CMD ["node", "server.js"]
```
