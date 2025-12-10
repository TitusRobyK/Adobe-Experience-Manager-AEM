# AGENTS.md

> **What is this file?**  
> `AGENTS.md` is **not** an AEM or Cloud Manager config file.  
> It is guidance for **AI Coding Agents** (Cursor, GitHub Copilot, Cline, ChatGPT, etc.) on how to safely understand, modify, and extend this AEM Maven project.

---

## 1. Project Purpose & Context

This repository contains one or more **Adobe Experience Manager** (AEM) codebases targeting:

- **AEM as a Cloud Service (AEMaaCS)** – primary runtime target
- **AEM 6.5** – typically used for on-premise or AMS deployments, or for local development parity

The project is a **multi-module Maven build** that follows the standard AEM Maven archetype structure.

AI agents should assume:

- Code must remain **Cloud Service-compatible** (no use of deprecated / classic features).
- AEM 6.5 compatibility is desired where feasible (e.g., APIs that exist in both).
- Changes should be **non-disruptive to existing content** unless explicitly requested.

---

## 2. Technology Stack Overview

**Core technologies:**

- Java (AEM-compatible version, typically 11 or 17)
- Maven (multi-module, archetype-style project)
- AEM APIs:
  - Apache Sling
  - OSGi (R7+ annotations)
  - JCR / Oak
- HTL (Sightly) for component markup
- Clientlibs for front-end (JS/TS, SCSS/CSS)
- Sling Models, OSGi Services, Servlets, Filters
- Dispatcher configuration (Apache HTTPD + dispatcher module) – for AEMaaCS, via `dispatcher` module

**Testing:**

- JUnit 5 (preferred) or 4
- AEM Mocks / WCM.io (required for unit testing AEM context)
- Integration Tests (`it.tests` module)

---

## 3. Repository Layout (Typical)

> Note: Exact module names may differ (e.g., `mysite` vs `wknd`). Replace with real names when you recognize them.

Typical structure:

- `/pom.xml` – root Maven aggregator & parent POM
- `/core` – Java code
  - OSGi services, Sling Models, servlets, schedulers, workflows, utilities, etc.
- `/ui.apps` – AEM **Immutable** runtime artifacts
  - `/apps/...` components, templates, policies, dialogs, clientlibs
- `/ui.content` – AEM **Mutable** sample / initial content
  - `/content/...` sample pages, assets, configurations
  - `/conf/...` editable templates, policies
- `/ui.config`
  - OSGi Configurations (prefer `.cfg.json` format) ensuring runmode separation.
- `/dispatcher` or `/dispatcher.cloud`
  - Dispatcher configuration for Cloud Service (Apache 2.4 + modules).
- `/all`
  - Single package aggregating `ui.apps`, `ui.config`, `ui.content`, etc. for deployment.
- `/ui.frontend` (optional but common)
  - Front-end build (Webpack, Vite, npm) that generates clientlibs into `ui.apps`.

**AI RULE:**  
AI agents should **infer structure from the root `pom.xml`** and module `pom.xml` files before making changes.

---

## 4. Build & Run: Commands AI Agents Should Use

**Do not invent build commands.** Prefer using commands already documented in `README.md` or any `docs/` files. If nothing exists, default to:

### 4.1 Full Build

- **Full build (no deployment):**
  ```bash
  mvn clean install
  ```

- **Build + deploy to local AEM (Author):**
  ```bash
  mvn clean install -PautoInstallPackage
  ```
  or
  ```bash
  mvn clean install -PautoInstallSinglePackage
  ```

### 4.2 Individual Module Build (Faster Dev Cycle)

If you only changed a specific module (e.g., `core` Java code or `ui.apps` components), build only that module to save time.

- **Deploy Core Bundle only:**
  ```bash
  mvn -pl core clean install -PautoInstallBundle
  ```

- **Deploy UI Apps package only:**
  ```bash
  mvn -pl ui.apps clean install -PautoInstallPackage
  ```

- **Deploy UI Content package only:**
  ```bash
  mvn -pl ui.content clean install -PautoInstallPackage
  ```

### 4.3 Local Development Access

- **AEM Author:** [http://localhost:4502](http://localhost:4502) (Default credentials: `admin` / `admin`)
- **AEM Publish:** [http://localhost:4503](http://localhost:4503) (If running)

### 4.4 Dispatcher Validation (Critical for AEMaaCS)

- If a validator script or Docker image is available, use it.
- Typical command: `mvn -pl dispatcher validate` or `./dispatcher/scripts/validate.sh` (if present).

**AI RULE:**  
When modifying POMs, **do not** arbitrarily upgrade AEM dependencies or plugin versions. Only adjust versions when explicitly requested and keep them consistent with the parent POM (`<dependencyManagement>`).

---

## 5. AEM Runtimes & Compatibility Notes

### 5.1 AEM as a Cloud Service (AEMaaCS) - STRICT RULES

- **Immutable Code:** `/apps` and `/libs` are read-only at runtime. Do **not** write code that attempts to modify nodes under `/apps` at runtime.
- **Ephemeral Containers:**
  - **No Local File I/O:** Do not write to the local filesystem (`java.io.File`). Instances are ephemeral and files will be lost. Use Blob Stores or external storage if needed.
  - **No Long-Running Threads:** Do not spawn unmanaged threads or `Thread.sleep()`. Use **Sling Jobs** or **Sling Commons Scheduler**.
- **State:** Keep instances stateless. Store state in the JCR (repo) or external databases, not in memory or local files.
- **Replication:** Use Sling Content Distribution (SCD). Classic replication agents are not used for publishing content in CS.

### 5.2 AEM 6.5

- Code should generally be compatible with both.
- Use APIs available in both AEMaaCS and 6.5 unless a clear justification exists.

---

## 6. Coding Guidelines for AI Agents

### 6.1 Java (Core module)

**Patterns to follow:**

- Use **OSGi R7 annotations** (`@Component`, `@Activate`, `@Modified`, `@Reference`, `@Designate`)
- Use **Sling Models** for component logic:
  - `@Model(adaptables = { Resource.class, SlingHttpServletRequest.class }, defaultInjectionStrategy = DefaultInjectionStrategy.OPTIONAL)`
  - Use `@ValueMapValue`, `@OSGiService`, `@Self`, `@ScriptVariable`, etc.
- For servlets:
  - Use `@Component(service = Servlet.class, property = { ... })`
  - Register with `sling.servlet.paths` (bin-type) or `sling.servlet.resourceTypes` (component-bound).
- **Logging:**
  - Use **SLF4J** (`LoggerFactory.getLogger(...)`).
  - No `System.out.println` or raw stack traces.
  - Log levels: `DEBUG` for dev details, `INFO` for lifecycle, `WARN`/`ERROR` for issues.

**AI RULES:**

- Do **not** hardcode environment values (URLs, credentials). Use OSGi configs or Secret/Environment Variables.
- Do **not** use `admin-login` (AdminResourceResolver). Use **Service Users** with strict ACLs (`Map<String, Object> param = Collections.singletonMap(ResourceResolverFactory.SUBSERVICE, "myServiceUser");`).

---

### 6.2 HTL (Sightly) & Components

- Use `.html` files with HTL syntax under `/apps/<project>/components/...`.
- **Logic:** Avoid Java logic in HTL. Use **Sling Models** for business logic.
- **Separation of Concerns:** Component HTML structure in `ui.apps`. Dialog definitions (`cq:dialog`) in `ui.apps`.
- **Proxy Components:** Always extend core components where possible (`sling:resourceSuperType="core/wcm/components/..."`).
- **Security:** usage of HTL context (`context='html'`, `context='unsafe'`) must be strictly justified. Default context is usually sufficient.
- **HTL Style:**
  - Use `data-sly-use` to initialize Sling Models.
  - Use `data-sly-list` or `data-sly-repeat` for loops.
  - Use `data-sly-test` for conditional rendering.

**AI RULES:**

- Don’t remove or rename component resource types (`sling:resourceType`) without explicit instruction – this breaks existing content.
- Ensure `cq:dialog` XML structures are valid and match the Sling Model properties.

---

### 6.3 Clientlibs & Front-End

- Use `clientlibs` under `/apps/<project>/clientlibs/...`.
- **Structure:**
  - Separate `clientlib-base`, `clientlib-site`, `clientlib-dependencies`.
  - Use `categories` to link to templates/components.
- **Deployment:** If `ui.frontend` module exists, respect its build process. Do not manually edit minified JS/CSS in `ui.apps`.

---

### 6.4 Dispatcher / Apache Config

- **Environment Specifics:** Understand that Dispatcher config differs slightly between Local SDK and Cloud Service.
- **Validation:** Syntax errors in Dispatcher config will fail the Cloud Manager pipeline.
- **Structure:** Follow the `conf.d` and `conf.dispatcher.d` structure (symlinks in `enabled_farms`, etc.).

**AI RULES:**

- Do not add overly broad allow rules in `filters.any`. Be restrictive by default.
- Do not expose `/system`, `/crx`, `/crxde`, `/libs`, or `/bin/querybuilder` publicly.

---

## 7. Configuration Management (OSGi)

- **Location:** `/ui.config/src/main/content/jcr_root/apps/<project>/osgiconfig`
- **Format:** Prefer **`.cfg.json`** for AEMaaCS projects.
- **Runmodes:**
  - `config` (global)
  - `config.author`, `config.publish`
  - `config.prod`, `config.stage`, `config.dev`
- **Secrets:**
  - Use `$[env:SECRET_VAR_NAME]` (Cloud Manager secret variables).
  - Do NOT commit actual passwords/keys to Git.

---

## 8. Indexing (Oak)

- **Custom Indexes:** Define custom Oak indexes for any query that is not covered by out-of-the-box indexes.
- **Location:** `ui.apps/src/main/content/jcr_root/_oak_index` containing valid Oak index definitions (often XML or JSON).
- **Rule:** Do NOT manage indexes manually in valid environments via CRXDE. They must be committed to code.

---

## 9. Asset Processing

- **Asset Compute Service:** AEMaaCS uses microservices for asset processing (renditions, metadata).
- **Legacy Workflows:** Avoid migrating heavy "Dam Update Asset" workflow customizations directly. Use **Processing Profiles** and **Asset Compute Workers** where possible.

---

## 10. Testing Strategy

- **Unit Tests (`core`)**:
  - Mandatory for new Java code.
  - Use `AemContext` (wcm.io) to mock JCR/Sling resources.
- **Integration Tests (`it.tests`)**:
  - Run against a running AEM instance.
- **Cloud Manager:** Pipeline includes functionality testing, UI testing, and security scanning.

**AI RULES:**

- Ensure `mvn clean install` passes locally before proposing code.
- Write tests that verify both success and failure cases.

---

## 11. Common Tasks – Hints for AI Agents

### 11.1 Create a New Sling Model

1. Create Java Interface + Impl (or just Class) in `core`.
2. Annotate with `@Model`.
3. Use `@ValueMapValue` for properties.
4. Create corresponding Component in `ui.apps`.
5. Create `cq:dialog` to allow authoring properties.

### 11.2 Add an OSGi Service

1. Create Interface + Implementation in `core`.
2. Annotate `@Component(service = MyService.class)`.
3. Create Configuration Object Design (`@ObjectClassDefinition`).
4. Add default config in `ui.config` (`.cfg.json`).

---

## 12. If You’re Unsure…

1. **Inspect existing code** (especially `core` models and `ui.config`).
2. **Mimic existing conventions**.
3. **Ask** the user for clarification on architectural decisions (e.g., "Should this be a new component or an extension?").

---
