## DevOps setup Docker // Beginner
## How to Install  InfluxDB, PostgreSQL, Cassandra (with Web UI) on Docker

Sure! Here’s the full step-by-step **installation and setup guide in English** for InfluxDB, PostgreSQL (with pgAdmin), and Cassandra (with Web UI) on Docker, so that all services work like InfluxDB and are accessible from the web.



# **Step-by-Step Installation**

## **1️⃣ Prepare the project folder**

1. Create a folder for your Docker project:

```bash
mkdir docker-db-project
cd docker-db-project
```

2. Create a file called `docker-compose.yml` in this folder.

> You can use the ready-made file I provided earlier (with passwords) or create a new one.



## **2️⃣ Write `docker-compose.yml`**

Minimal working example:

```yaml
version: '3.8'

services:
  influxdb:
    image: influxdb:2.7
    restart: unless-stopped
    ports:
      - "8086:8086"
    environment:
      - INFLUXDB_2_INIT_USERNAME=influx_admin
      - INFLUXDB_2_INIT_PASSWORD=InfluxPass123!
      - INFLUXDB_2_INIT_ORG=my-org
      - INFLUXDB_2_INIT_BUCKET=my-bucket
      - INFLUXDB_2_INIT_TOKEN=my-token-0123456789
    volumes:
      - influxdb-data:/var/lib/influxdb2
    networks:
      - dbnet

  postgres:
    image: postgres:15
    restart: unless-stopped
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=pguser
      - POSTGRES_PASSWORD=PgPass123!
      - POSTGRES_DB=mydatabase
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - dbnet

  pgadmin:
    image: dpage/pgadmin4
    restart: unless-stopped
    depends_on:
      - postgres
    ports:
      - "8080:80"
    environment:
      - PGADMIN_DEFAULT_EMAIL=admin@example.com
      - PGADMIN_DEFAULT_PASSWORD=PgAdminPass123!
    volumes:
      - pgadmin-data:/var/lib/pgadmin
    networks:
      - dbnet

  cassandra:
    image: cassandra:4.1
    restart: unless-stopped
    ports:
      - "9043:9042"   # host port 9043, container port 9042
    platform: linux/arm64/v8
    volumes:
      - cassandra-data:/var/lib/cassandra
    networks:
      dbnet:
        ipv4_address: 172.28.0.10
    healthcheck:
      test: ["CMD-SHELL", "nodetool status >/dev/null 2>&1 || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 10

  cassandra-web:
    image: dcagatay/cassandra-web:latest
    restart: unless-stopped
    depends_on:
      - cassandra
    ports:
      - "3000:3000"
    environment:
      - CASSANDRA_HOST_IPS=cassandra
      - CASSANDRA_PORT=9042
      - CASSANDRA_USERNAME=cassandra
      - CASSANDRA_PASSWORD=cassandra
    networks:
      dbnet:
        ipv4_address: 172.28.0.20

volumes:
  influxdb-data:
  pgdata:
  pgadmin-data:
  cassandra-data:

networks:
  dbnet:
    driver: bridge
    ipam:
      config:
        - subnet: 172.28.0.0/16
```

**Quick explanation of each service**

* **InfluxDB:** Time-series DB, web UI on port 8086
* **PostgreSQL:** Main DB, port 5432
* **pgAdmin:** Web UI for PostgreSQL, port 8080
* **Cassandra:** NoSQL DB, port 9042 (host maps to 9043)
* **Cassandra Web UI:** Web interface for Cassandra, port 3000



## **3️⃣ Check ports and remove old containers**

Make sure no container or process is using the same ports:


# Check if ports are in use
sudo lsof -nP -iTCP -sTCP:LISTEN | egrep "8086|5432|8080|9043|3000" || true

# Remove old conflicting containers
docker rm -f influxdb postgres pgadmin cassandra cassandra-web || true


> ⚠️ If ports conflict, you can change the host port in docker-compose (e.g., `5433:5432`).



## **4️⃣ Run Docker Compose**

docker compose up -d --build

* `-d` → run in background
* `--build` → rebuild images if modified

Check containers are running:

docker compose ps


## **5️⃣ Check logs**

Check each service to make sure it is ready:

docker compose logs -f influxdb postgres pgadmin cassandra cassandra-web


* InfluxDB ready → [http://localhost:8086](http://localhost:8086)
* pgAdmin ready → [http://localhost:8080](http://localhost:8080)
* Cassandra ready → [http://localhost:3000](http://localhost:3000) (after healthcheck passes)


## **6️⃣ Configure web UIs**

### **InfluxDB**

* Go to [http://localhost:8086](http://localhost:8086)
* Login: `influx_admin / InfluxPass123!`
* Org, Bucket, and Token will be auto-created

### **PostgreSQL via pgAdmin**

1. Go to [http://localhost:8080](http://localhost:8080)
2. Login: `admin@example.com / PgAdminPass123!`
3. Add New Server:

   * Name: any name
   * Hostname/address: `postgres`
   * Port: `5432`
   * Username: `pguser`
   * Password: `PgPass123!`

> ❗ Use the service name `postgres` instead of `localhost` — `localhost` would point to the pgAdmin container itself

### **Cassandra**

* Connect via cqlsh:


docker exec -it <cassandra_container_name> cqlsh

* Username/password: `cassandra / cassandra`
* Cassandra Web UI: [http://localhost:3000](http://localhost:3000)


## **7️⃣ Tips to make it fully usable**

* Add `depends_on` to ensure container startup order
* Cassandra may take time to fully boot — wait for healthcheck
* Avoid host port conflicts
* pgAdmin must use the Docker service name (`postgres`) as host


💡 **Summary**

1. Prepare folder → create docker-compose.yml
2. Write compose + environment variables
3. Check ports/remove old containers
4. `docker compose up -d --build`
5. Check logs
6. Configure web UIs with correct host/service name
7. Wait for Cassandra to boot → all UIs fully accessible

## Convert .yml to Diagram

![6F318B48-9E72-472A-BC18-73B2B148D819_1_105_c](https://github.com/user-attachments/assets/7b2bf52e-75a1-4af0-afba-512c7c84bbb8)


### 🔍 Overall Structure

The image you sent is from **ToDiagram**, an online tool used to visualize **architecture or system structures** (such as Docker Compose, Kubernetes, or network topologies) from YAML or JSON code.

From what’s shown, this diagram likely represents a **Docker Compose or container network architecture**, where each block corresponds to a service, database, or configuration component like:

* **InfluxDB**
* **PostgreSQL**
* **Grafana**
* **Backend (API service)**
* **Networks & Volumes**


### 🧩 Main Components

#### 1. **InfluxDB**

* Has environment variables such as:

  * `INFLUXDB_2_INIT_USERNAME`
  * `INFLUXDB_2_INIT_PASSWORD`
  * `INFLUXDB_2_INIT_ORG`
* Indicates it’s a **time-series database** (used for IoT or metrics data)
* Exposes port **8086**

#### 2. **PostgreSQL**

* Uses environment variables like:

  * `POSTGRES_USER`
  * `POSTGRES_PASSWORD`
  * `POSTGRES_DB`
* Serves as a **relational database**
* Exposes port **5432**

#### 3. **Grafana**

* Connected to InfluxDB for **data visualization**
* Runs on port **3000**
* Uses a **volume mount** to persist dashboard data

#### 4. **Backend or Application Service**

* Likely your main app (e.g., Node.js, Flask, or FastAPI)
* Connects to both **InfluxDB** and **PostgreSQL**
* Includes environment variables like API keys or database URLs

#### 5. **Networks and Volumes**

* All services connect through a **shared network** (e.g., `monitoring-network`, `app-network`)
* Persistent **volumes** store data safely, e.g., `influx_data`, `postgres_data`, `grafana_data`


### ⚙️ Connections

The lines between each block show how containers **communicate with each other** within the same network:

* **Grafana → InfluxDB** (to read time-series data)
* **Backend → PostgreSQL** (to read/write relational data)
* **InfluxDB and PostgreSQL** store different types of data but can work together in the system.


### 💡 Summary

This diagram visualizes a **multi-service Docker Compose system** showing how:

* **InfluxDB** (for time-series data)
* **PostgreSQL** (for structured relational data)
* **Grafana** (for monitoring dashboards)
* and an **Application Service (API/Backend)**
  are all connected using **networks and volumes**.

Each service has its own environment settings, ports, and dependencies — giving a clear overview of how the entire system works without reading the raw YAML code.


## Introduction.

In today’s world, software needs to be built, tested, and deployed faster than ever before. Developers face challenges like application inconsistency across different environments, slow deployments, and communication gaps between teams.

That’s where Docker and DevOps come in.
Docker helps developers create portable and consistent environments, while DevOps focuses on improving collaboration between development and operations teams.
Together, they make the process of building and deploying software faster, more reliable, and more efficient. So, let’s start by understanding what Docker really is."*

## **1. What and Why DevOps?**

DevOps is a set of practices that combines **Development (Dev)** and **Operations (Ops)** to shorten the development lifecycle and deliver high-quality software continuously.

* Focus on collaboration between developers and operations teams.
* Automates testing, deployment, and monitoring.
* Speeds up feedback and release cycles.

## **2. Why use DevOps?**

* Faster delivery of software and updates.
* Improved collaboration and communication.
* Reduced errors and higher quality.
* Continuous feedback and improvement.


## **3. What is Docker?**
Docker is an open-source platform that automates the deployment, scaling, and management of applications inside lightweight, portable containers. It allows developers to package applications and their dependencies into a single unit that can run consistently across different environments.


## **4. What can Docker do and what can it connect to?**

* Run any application consistently across multiple environments.
* Package apps with all dependencies (libraries, frameworks, settings).
* Integrate with CI/CD pipelines for DevOps.
* Connect to databases (MySQL, PostgreSQL, MongoDB, InfluxDB).
* Communicate between containers using Docker Networking.
* Connect to cloud services (AWS, Azure, GCP) and orchestration tools like Kubernetes.


## **5. Why use Docker?**

* **Consistency:** Solves the "It works on my machine" problem.
* **Portability:** Move apps between environments without reconfiguration.
* **Isolation:** Each app runs in its own environment without conflicts.
* **Efficiency:** Lightweight compared to full virtual machines.


## **6. What is a Container?**

A container is a lightweight, standalone package that includes an application and all its dependencies, libraries, and configuration files. Containers share the host OS kernel but run isolated processes.


## **7. What is Docker Compose?**

Docker Compose is a tool for defining and running multi-container Docker applications using a single YAML file (`docker-compose.yml`). It simplifies the process of starting, stopping, and connecting multiple containers.

Docker Compose
<img width="1470" height="956" alt="ภาพถ่ายหน้าจอ 2568-10-06 เวลา 07 03 11" src="https://github.com/user-attachments/assets/746b3f16-ce24-4d12-882c-5770580bd161" />

# everything is run in terminal
![69677641-3AA5-41AC-8650-399EB959BB5A_4_5005_c](https://github.com/user-attachments/assets/26efea66-2fa3-430a-a354-2207677d4a12)

# everything is run in docker
<img width="1470" height="956" alt="ภาพถ่ายหน้าจอ 2568-10-06 เวลา 07 03 17" src="https://github.com/user-attachments/assets/e321ce37-4e23-49dc-8f7b-adea84176d93" />

# Influx_db
<img width="1470" height="956" alt="09141D65-B850-4053-A832-CBE9CF72350E" src="https://github.com/user-attachments/assets/172f2423-2168-45e7-9969-737155a71991" />

<img width="1470" height="956" alt="ภาพถ่ายหน้าจอ 2568-10-06 เวลา 07 07 22" src="https://github.com/user-attachments/assets/b6c4e824-995a-4732-84cb-d30a8459f050" />


## **8. Conclusion**

Docker and DevOps are powerful tools that work together to make software development and deployment faster, more reliable, and easier to manage.
Docker provides a consistent, portable, and efficient way to run applications using containers, while Docker Compose simplifies managing multiple containers.
DevOps integrates development and operations, automates workflows, and improves collaboration, allowing teams to deliver high-quality software continuously.
