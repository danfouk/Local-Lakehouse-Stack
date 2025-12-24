Here’s a polished, onboarding‑ready **README.md** for your full stack:  
Apache Iceberg + Nessie + Dremio + PySpark + VS Code Dev Containers + external MinIO.

I’ve written it the way you like—explicit, deterministic, and clean enough for new hires to follow without friction.

---

# **README.md**  
## **Iceberg + Nessie + Dremio + PySpark Development Environment (with VS Code Dev Containers & external MinIO)**

This repository provides a fully reproducible local development environment for working with:

- **Apache Iceberg**  
- **Project Nessie** (catalog + versioning)  
- **Dremio** (SQL engine + UI)  
- **PySpark** (Iceberg‑aware Spark runtime)  
- **MinIO (external deployment)** as the S3 object store  
- **VS Code Dev Containers** as the primary IDE  

The goal is to give developers a clean, deterministic, onboarding‑friendly environment for building and testing Iceberg‑based data workflows.

---

## **Architecture Overview**

```
+-------------------+       +-------------------+
|     PySpark       | <---> |     Nessie        |
|  (Dev Container)  |       |   Catalog API     |
+-------------------+       +-------------------+
          |                           |
          |                           |
          v                           v
+------------------------------------------------+
|                MinIO (external)                |
|     S3-compatible object storage backend       |
|     Bucket: warehouse                          |
+------------------------------------------------+
          ^
          |
+-------------------+
|      Dremio       |
|  SQL + UI Engine  |
+-------------------+
```

---

## **Prerequisites**

Before starting, ensure you have:

- **Docker** (or OrbStack)  
- **VS Code**  
- **VS Code Dev Containers extension**  
- A running **MinIO** instance (external)  
  - Endpoint: `http://minio.orb.local:9000`  
  - Access Key: `danray`  
  - Secret Key: `Welcome_123456`  
  - Bucket: `warehouse`

---

## **Repository Structure**

```
.
├── docker-compose.yml
├── Makefile
├── workspace/
│   ├── notebooks/
│   ├── scripts/
│   └── .devcontainer/
│       ├── devcontainer.json
│       └── Dockerfile
└── README.md
```

---

## **1. Starting the Environment**

Use the Makefile for deterministic startup:

```bash
make up
```

This launches:

- Nessie  
- Dremio  
- PySpark dev container  

To stop everything:

```bash
make down
```

To view logs:

```bash
make logs
```

---

## **2. Opening the Project in VS Code**

1. Open VS Code  
2. Press **Ctrl+Shift+P**  
3. Select:  
   **Dev Containers: Open Folder in Container**  
4. Choose the `workspace/` folder  

VS Code will build the dev container using:

```
workspace/.devcontainer/Dockerfile
```

You now have:

- Python  
- PySpark  
- Iceberg libraries  
- Jupyter  
- Full S3 + Nessie connectivity  

---

## **3. Launching PySpark (Iceberg‑ready)**

You can launch PySpark in two ways:

### **Option A — VS Code Task**

Press **Ctrl+Shift+P** →  
**Tasks: Run Task** →  
Select **Launch PySpark (Iceberg)**

### **Option B — Makefile**

```bash
make pyspark
```

This starts PySpark with:

- Iceberg runtime  
- Nessie catalog  
- MinIO S3 backend  
- Path‑style access  
- SSL disabled (local dev)  

---

## **4. Connecting Dremio**

Open Dremio UI:

```
http://localhost:9047
```

### **Add MinIO as S3 Source**

- Access Key: `danray`  
- Secret Key: `Welcome_123456`  
- Endpoint: `http://minio.orb.local:9000`  
- Enable: **Compatibility Mode**  
- Root Path: `warehouse`

### **Add Nessie Source**

- Endpoint: `http://nessie:19120/api/v2`  
- Default branch: `main`  
- Warehouse: `s3a://warehouse/`

---

## **5. Creating Iceberg Tables in PySpark**

Example:

```python
spark.sql("""
CREATE TABLE nessie.db.sample (
    id INT,
    name STRING
)
USING iceberg
""")
```

Data will be stored in:

```
s3a://warehouse/db/sample/
```

---

## **6. Jupyter Notebooks**

Inside the dev container:

```bash
jupyter notebook --ip=0.0.0.0 --allow-root
```

VS Code will auto-detect the server.

---

## **7. Troubleshooting**

### **MinIO permissions**
Ensure MinIO’s backend directory is writable:

```bash
sudo chown -R minio-user:minio-user /mnt/data
sudo chmod -R u+rxw /mnt/data
```

### **Nessie health**
Check:

```bash
curl http://localhost:19120/api/v2/config
```

### **PySpark package mismatch**
Ensure Iceberg runtime matches Spark version:

```
org.apache.iceberg:iceberg-spark-runtime-3.4_2.12:1.5.0
```

---

## **8. Extending the Environment**

You can easily add:

- Spark History Server  
- Dremio executor nodes  
- Iceberg REST catalog  
- DuckDB integration  
- Fugue or Ray for distributed workflows  

Just extend the Compose file or Dev Container.

---

## **9. Summary**

This environment gives you:

- A fully reproducible Iceberg development stack  
- VS Code as a first‑class IDE  
- Seamless integration with external MinIO  
- PySpark configured for Iceberg + Nessie  
- Dremio for SQL + UI exploration  
- Clean onboarding for new engineers  

It’s designed for clarity, determinism, and long‑term maintainability.

---

If you'd like, I can also generate:

- A sample Iceberg notebook  
- A bootstrap script that creates Nessie branches + tables  
- A CI workflow for validating the stack  
- A diagram for the README  

Just tell me what you want next.
