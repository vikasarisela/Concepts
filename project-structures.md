# Blueprint: Cross-Platform Project Structures & Build Artifacts

As a DevOps engineer, you do not need to write application code, but you **must** know where dependency files live, where build artifacts are generated, and what files to exclude from production images to build efficient CI/CD pipelines.

---

## 🟢 1. Node.js (JavaScript / TypeScript)

Node.js applications are interpreted (or compiled via TypeScript) and rely heavily on the `node_modules` directory for dependencies.

### Project Layout
```text
my-node-app/
├── src/                      # Source code (.js or .ts files)
├── dist/ or build/           # 📦 OUTPUT: Compiled JS files (TypeScript only)
├── package.json              # 📋 CRITICAL: Project metadata and script definitions
├── package-lock.json         # 🔒 CRITICAL: Exact dependency version lockfile
├── node_modules/             # ⚠️ EXCLUDE: Heavy local dependencies directory
└── Dockerfile                # CI/CD deployment blueprint
```

### 🛠️ DevOps Pipeline Cheat Sheet
* **Dependency Installation:** `npm ci --only=production` (Use `ci` instead of `install` in pipelines for deterministic, faster builds based strictly on the lockfile).
* **Build Command (TypeScript):** `npm run build` (Outputs clean JavaScript to `/dist`).
* **What to Deploy:** Only `/dist`, `package.json`, `package-lock.json`, and **production** `node_modules`. **Never** package the `src/` folder or development `node_modules` into a production server or container.

---

## ☕ 2. Java (Spring Boot / Maven or Gradle)

Java applications are compiled into a single, highly portable compressed archive file (JAR or WAR) that runs inside the Java Virtual Machine (JVM).

### Project Layout (Maven Example)
```text
my-java-app/
├── src/
│   ├── main/java/            # Source code (.java)
│   └── main/resources/       # App properties and configuration files
├── pom.xml                   # 📋 CRITICAL: Maven dependency & build configuration
├── build.gradle              # 📋 CRITICAL: Alternative if using Gradle
├── target/                   # 📂 BUILD OUTPUT FOLDER (Maven default)
│   ├── classes/              # Compiled bytecode (.class)
│   └── my-app-1.0.0.jar      # 📦 THE GOLDEN ARTIFACT: The runnable package
└── target/ or build/libs/    # Output folder if using Gradle
```

### 🛠️ DevOps Pipeline Cheat Sheet
* **Build Command (Maven):** `mvn clean package -DskipTests`
* **Build Command (Gradle):** `./gradlew build -x test`
* **What to Deploy:** **Only the `.jar` file** found inside the `target/` or `build/libs/` directory. The entire source tree (`src/`, `pom.xml`, etc.) can be completely discarded after the build stage finishes.
* **Execution:** `java -jar my-app-1.0.0.jar`

---

## 🔵 3. .NET (C# / Core)

Modern .NET applications use a compilation framework that outputs cross-platform binaries optimized for specific environments (like Linux containers).

### Project Layout
```text
my-dotnet-app/
├── Program.cs                # Application entry point
├── App.csproj                # 📋 CRITICAL: Project dependencies & target framework
├── App.sln                   # Solution file (groups multiple projects together)
├── bin/ or obj/              # ⚠️ EXCLUDE: Temporary local debug compilation folders
└── publish/                  # 📦 OUTPUT: Hardened production-ready binaries
    ├── App.dll               # Application bytecode
    ├── App.runtimeconfig.json# Runtime environment definitions
    └── [third-party].dll    # Compiled third-party framework dependencies
```

### 🛠️ DevOps Pipeline Cheat Sheet
* **Restore Dependencies:** `dotnet restore`
* **Build & Publish Command:** `dotnet publish -c Release -o ./publish`
* **What to Deploy:** **Only the contents of the `/publish` directory**. Everything else is completely unnecessary in production.
* **Execution:** `dotnet App.dll` (or a native binary if compiled using Ahead-of-Time (AOT) compilation).

---

## 📊 Summary: What DevOps Cares About Most

| Platform | Dependency Manifest | Lockfile (Strict Versions) | Production Artifact Location | Target Runtime |
| :--- | :--- | :--- | :--- | :--- |
| **Node.js** | `package.json` | `package-lock.json` | `/dist` + Production `node_modules` | `node` |
| **Java** | `pom.xml` or `build.gradle` | `pom.xml.tag` / Gradle locks | `target/*.jar` or `build/libs/*.jar` | `java -jar` |
| **.NET** | `*.csproj` | `packages.lock.json` (Optional) | `/publish/*` | `dotnet *.dll` |



### Stack-Specific Service File Configuration (The `ExecStart` Core)
`ExecStart` requires **absolute file paths** for both the runtime interpreter and the application files.

#### Node.js
```ini
WorkingDirectory=/var/www/my-node-app
ExecStart=/usr/bin/node /var/www/my-node-app/server.js
```

#### Python (e.g., FastAPI / Flask via Virtual Environment)
```ini
WorkingDirectory=/var/www/my-python-app
ExecStart=/var/www/my-python-app/venv/bin/gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app
```

#### Java (Spring Boot)
```ini
WorkingDirectory=/var/www/my-java-app
ExecStart=/usr/bin/java -jar /var/www/my-java-app/my-application.jar
```

#### Go (Golang) or Rust (Native Compiled Binary)
```ini
WorkingDirectory=/var/www/my-go-app
ExecStart=/var/www/my-go-app/server-binary
```

---

## 📂 Part 4: Production Stacks & Build Artifacts Blueprint

DevOps pipelines must separate source code from production artifacts. Use this map to know what to build, what to copy, and what to exclude.

### 1. Node.js (JavaScript / TypeScript)
*   **Manifest:** `package.json` | **Lockfile:** `package-lock.json`
*   **Pipeline Build Command:** `npm ci --only=production` *(for pure JS)* or `npm run build` *(for TS)*.
*   **What to Deploy:** `/dist` (or `/build`), `package.json`, `package-lock.json`, and production `node_modules`. **Exclude** the `src/` directory and development dependencies.

### 2. Java (Spring Boot / Maven or Gradle)
*   **Manifest:** `pom.xml` (Maven) or `build.gradle` (Gradle)
*   **Pipeline Build Command:** `mvn clean package -DskipTests` or `./gradlew build -x test`
*   **What to Deploy:** **Only the `.jar` file** generated inside `target/` or `build/libs/`. The entire source tree can be completely discarded after the compilation pipeline stage.

### 3. .NET (C#)
*   **Manifest:** `*.csproj` | **Lockfile:** `packages.lock.json`
*   **Pipeline Build Command:** `dotnet publish -c Release -o ./publish`
*   **What to Deploy:** **Only the contents of the `/publish` directory** (containing the `App.dll`, dependencies, and runtime configurations).

### 4. Cross-Platform Dependency Manifest Equivalents

| Language | The Manifest File <br>*(The Shopping List)* | The Lockfile <br>*(Locks exact versions for production)* | The Installation Tool <br>*(CLI Framework)* |
| :--- | :--- | :--- | :--- |
| **Node.js** | `package.json` | `package-lock.json` / `yarn.lock` | `npm` / `yarn` / `pnpm` |
| **Python** | `requirements.txt` / `pyproject.toml` | `poetry.lock` / `Pipfile.lock` | `pip` / `poetry` |
| **Java** | `pom.xml` or `build.gradle` | *Handled via internal hashes / tags* | `mvn` / `gradle` |
| **.NET (C#)** | `*.csproj` | `packages.lock.json` | `dotnet` CLI |
| **Go (Golang)**| `go.mod` | `go.sum` | `go` CLI |

> ⚠️ **DevOps Rule of Thumb:** Always write pipelines that target the **Lockfile** (like running `npm ci` instead of `npm install`). The manifest file allows version drifts, but the lockfile guarantees that the exact cryptographic version tested by the developer is what gets deployed to production.