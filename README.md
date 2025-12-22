Here you go, Daniel — a **clean, professional, onboarding‑ready README** tailored to the full enhanced stack you’re building. It’s written the way you’d hand it to a team: explicit, reproducible, and operationally clear.

If you want, I can also generate a **diagram**, **quickstart cheatsheet**, or **Iceberg demo notebook** to accompany it.

---

# **README — Local Lakehouse Stack (Nessie + Iceberg + MinIO + Spark + Dremio)**

This repository provides a fully integrated **local lakehouse environment** built on:

- **Nessie** — Git‑like catalog for Iceberg  
- **MinIO** — S3‑compatible object store  
- **Apache Spark 3.5** — Compute engine with Jupyter Lab  
- **Dremio OSS** — SQL engine + UI for Iceberg  
- **Nessie UI** — Web interface for branches, commits, tables  
- **MinIO Bootstrap** — Automated bucket creation + IAM‑style policies  

The stack is designed for **reproducible, production‑grade local development**, with explicit ordering, isolated credentials, and clean networking.

---

# **1. Architecture Overview**

```
                   ┌──────────────────────────┐
                   │        Nessie UI          │
                   │    http://localhost:19121 │
                   └──────────────┬───────────┘
                                  │
                     ┌────────────▼────────────┐
                     │        Nessie API        │
                     │   http://localhost:19120 │
                     └────────────┬────────────┘
                                  │
                     ┌────────────▼────────────┐
                     │       Spark Master       │
                     │  Jupyter @ :8888         │
                     └────────────┬────────────┘
                                  │
                     ┌────────────▼────────────┐
                     │      Spark Worker        │
                     └────────────┬────────────┘
                                  │
                     ┌────────────▼────────────┐
                     │         Dremio           │
                     │  http://localhost:9047   │
                     └────────────┬────────────┘
                                  │
                     ┌────────────▼────────────┐
                     │         MinIO            │
                     │  http://localhost:9000   │
                     └──────────────────────────┘
```

---

# **2. Prerequisites**

- Docker  
- Docker Compose v2  
- 8GB RAM minimum (recommended 16GB)  
- Ports available:
  - 9000–9001 (MinIO)
  - 19120–19121 (Nessie)
  - 7077, 8080–8081, 8888, 18080 (Spark)
  - 9047, 31010, 32010, 45678 (Dremio)

---

# **3. Quick Start**

### **Start the entire stack**

```bash
docker compose up -d
```

### **Stop everything**

```bash
docker compose down
```

### **Stop and remove volumes**

```bash
docker compose down -v
```

---

# **4. Service Endpoints**

| Service       | URL / Port | Notes |
|---------------|------------|-------|
| **Nessie API** | http://localhost:19120 | Iceberg catalog REST endpoint |
| **Nessie UI** | http://localhost:19121 | Browse branches, commits, tables |
| **MinIO Console** | http://localhost:9001 | S3 browser |
| **MinIO S3 Endpoint** | http://localhost:9000 | Use in Spark/Dremio |
| **Spark Master UI** | http://localhost:8080 | Cluster overview |
| **Spark Worker UI** | http://localhost:8081 | Worker metrics |
| **Spark History Server** | http://localhost:18080 | Job history |
| **Jupyter Lab** | http://localhost:8888 | No token/password |
| **Dremio UI** | http://localhost:9047 | SQL + Iceberg exploration |

---

# **5. MinIO Buckets & IAM Setup**

The `minio-init` service automatically:

### **Creates buckets**
- `datalake`
- `datalakehouse`
- `warehouse`
- `seed`

### **Creates IAM-style user**
```
username: sparkuser
password: sparkpassword
```

### **Applies policy**
`datalake-readwrite.json` → grants full access to `datalake/*`.

You can add more policies in `./minio-policies`.

---

# **6. Spark Configuration (Iceberg + Nessie)**

Inside Jupyter, configure Spark like this:

```python
spark = (
    SparkSession.builder.appName("iceberg")
    .config("spark.sql.catalog.nessie", "org.apache.iceberg.spark.SparkCatalog")
    .config("spark.sql.catalog.nessie.catalog-impl", "org.apache.iceberg.nessie.NessieCatalog")
    .config("spark.sql.catalog.nessie.uri", "http://nessie:19120/api/v1")
    .config("spark.sql.catalog.nessie.ref", "main")
    .config("spark.sql.catalog.nessie.authentication.type", "NONE")
    .config("spark.sql.catalog.nessie.warehouse", "s3a://datalake/")
    .config("spark.hadoop.fs.s3a.endpoint", "http://minio:9000")
    .config("spark.hadoop.fs.s3a.access.key", "admin")
    .config("spark.hadoop.fs.s3a.secret.key", "password")
    .config("spark.hadoop.fs.s3a.path.style.access", "true")
    .getOrCreate()
)
```

---

# **7. Dremio Configuration**

### **Add MinIO as an S3 Source**
In Dremio UI:

- **Source Type:** Amazon S3  
- **Access Key:** `admin`  
- **Secret Key:** `password`  
- **Root Path:** `/`  
- **Enable compatibility mode:** ✔  
- **Endpoint:** `http://minio:9000`  
- **Encryption:** Off (unless TLS enabled)

### **Add Nessie as an Iceberg Catalog**
- **Type:** Nessie  
- **REST API URL:** `http://nessie:19120/api/v1`  
- **Authentication:** None  
- **Default branch:** `main`  
- **Warehouse:** `s3a://datalake/`

---

# **8. Example: Create an Iceberg Table in Spark**

```python
spark.sql("""
CREATE TABLE nessie.demo.customers (
    id INT,
    name STRING,
    country STRING
)
USING iceberg
""")
```

Verify in Nessie UI → `main` branch → `demo.customers`.

---

# **9. Example: Query the Table in Dremio**

In Dremio SQL Runner:

```sql
SELECT * FROM nessie.demo.customers;
```

---

# **10. Directory Structure**

```
.
├── docker-compose.yml
├── nessie-data/
├── minio-data/
├── minio-seed/
├── minio-policies/
│   └── datalake-readwrite.json
├── notebook-seed/
└── README.md
```

---

# **11. Troubleshooting**

### **Spark cannot access MinIO**
- Ensure `path.style.access=true`
- Ensure endpoint is `http://minio:9000`
- Ensure buckets exist (`minio-init` must complete)

### **Dremio cannot read Iceberg tables**
- Check Nessie URL: `http://nessie:19120/api/v1`
- Ensure warehouse path matches Spark

### **Nessie UI shows no tables**
- Ensure Spark wrote to `nessie` catalog
- Ensure branch is `main`

### **MinIO console not loading**
- Check port 9001 availability
- Restart MinIO: `docker compose restart minio`

---

# **12. Extending the Stack**

You can easily add:

- TLS for MinIO  
- Additional Spark workers  
- A dedicated Spark History volume  
- A Nessie PostgreSQL backend  
- A Dremio coordinator/executor split  

Ask if you want templates for any of these.

---

If you want, I can also generate:

- A **quickstart notebook** that creates Iceberg tables  
- A **diagram** of the full data flow  
- A **team‑ready onboarding PDF**  
- A **Makefile** for common commands  

Just tell me what direction you want to take this next.
