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
