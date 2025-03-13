# 🚀 Keycloak Setup with Docker & PostgreSQL

This document details the steps I took to install, configure, and troubleshoot Keycloak with a PostgreSQL database using Docker Compose.

---

## 📌 Phase 1: Project Initialization & Setup

### ✅ Install Required Tools
Before setting up Keycloak, I ensured all required tools were installed:
```bash
sudo apt update && sudo apt install -y docker docker-compose
```

### ✅ Initialize Git & Project Structure
I set up my project directory and initialized a Git repository:
```bash
mkdir keycloak-setup && cd keycloak-setup
git init
git branch -M main
git remote add origin https://github.com/<your-username>/keycloak-setup.git
```

### ✅ Create `docker-compose.yml`
I created a `docker-compose.yml` file to define the Keycloak and PostgreSQL services:
```yaml
version: '3.8'

services:
  keycloak:
    image: quay.io/keycloak/keycloak:latest
    environment:
      - KC_DB=postgres
      - KC_DB_URL_HOST=keycloak_postgres
      - KC_DB_URL_DATABASE=keycloak
      - KC_DB_USERNAME=keycloak
      - KC_DB_PASSWORD=keycloak
      - KEYCLOAK_ADMIN=admin
      - KEYCLOAK_ADMIN_PASSWORD=admin
    command: ["start-dev"]
    ports:
      - "8080:8080"
    container_name: keycloak
    depends_on:
      - keycloak_postgres
    networks:
      - keycloak-network

  keycloak_postgres:
    image: postgres:15
    environment:
      - POSTGRES_DB=keycloak
      - POSTGRES_USER=keycloak
      - POSTGRES_PASSWORD=keycloak
    ports:
      - "5432:5432"
    container_name: keycloak_postgres
    networks:
      - keycloak-network
    volumes:
      - keycloak_db_data:/var/lib/postgresql/data

networks:
  keycloak-network:

volumes:
  keycloak_db_data:
```

---

## 📌 Phase 2: Deploying & Running Keycloak

### ✅ Start Keycloak & PostgreSQL
To spin up the services:
```bash
docker-compose up -d
```
This pulled the necessary images, created a network, and started both containers.

### ✅ Verify Running Containers
I checked if Keycloak and PostgreSQL were running:
```bash
docker ps
```

### ✅ View Keycloak Logs
To ensure Keycloak was running without errors:
```bash
docker-compose logs -f
```

---

## 📌 Phase 3: Fixing Admin Login Issues

### ❌ Issue: Unable to Log In as Admin
After deployment, I was unable to log into Keycloak with `admin/admin`. To investigate, I checked the database:

```bash
docker exec -it keycloak_postgres psql -U keycloak -d keycloak_db -c "SELECT username FROM user_entity;"
```
This query returned **zero rows**, meaning the admin user was not created.

---

### ✅ Manually Creating an Admin User
Since Keycloak's `bootstrap-admin` command failed, I manually inserted an admin user into the database:

```sql
INSERT INTO user_entity (id, username, email, email_constraint, enabled, created_timestamp)
VALUES ('admin-id-123', 'admin', 'admin@example.com', 'admin@example.com', true, extract(epoch from now()) * 1000);
```

Then, I manually inserted the admin's password hash:
```sql
INSERT INTO credential (id, type, created_date, user_id, secret_data, credential_data)
VALUES ('cred-id-123', 'password', extract(epoch from now()) * 1000, 'admin-id-123',
'{"algorithm":"pbkdf2-sha256","hashIterations":27500,"salt":"abcdefg1234567","value":"$pbkdf2-sha256$27500$.JQ5/4rUEN6g$gN2oy2gP6a5SgCfIETrRHvYheNBRsRfh"}', '{}');
```

Finally, I assigned the `admin` role:
```sql
INSERT INTO user_role_mapping (user_id, role_id)
SELECT 'admin-id-123', id FROM keycloak_role WHERE name = 'admin' LIMIT 1;
```

---

## 📌 Phase 4: Resetting Everything & Starting Fresh

### ✅ Stopping and Removing Everything
After multiple failed login attempts, I decided to reset Keycloak and PostgreSQL completely:
```bash
docker-compose down -v
docker system prune -a --volumes -f
```
This removed:
- **Containers** (`keycloak`, `keycloak_postgres`)
- **Networks** (`keycloak_keycloak_network`)
- **Volumes** (`keycloak_db_data`)

### ✅ Restarting Fresh
Once everything was wiped, I started again:
```bash
docker-compose up -d
```
This pulled fresh images, created new containers, and initialized the database.

---

## 📌 Phase 5: Final Testing & Verification

### ✅ Checking the Logs
```bash
docker-compose logs -f
```

### ✅ Accessing Keycloak
I opened **http://localhost:8080** and attempted to log in.

### ✅ Creating the Admin User Again
Since the database reset removed all previous changes, I ran:
```bash
docker exec -it keycloak /opt/keycloak/bin/kc.sh bootstrap-admin --user admin --password admin
```

### ✅ Restarting Keycloak
```bash
docker-compose restart keycloak
```

### ✅ Final Database Check
```bash
docker exec -it keycloak_postgres psql -U keycloak -d keycloak_db -c "SELECT username FROM user_entity;"
```
---

## 📌 Final Setup Summary

### ✅ What I Did
✔️ Set up **Keycloak with PostgreSQL** using Docker Compose  
✔️ Checked and troubleshooted **admin login issues**  
✔️ Manually created an **admin user** in the database  
✔️ Reset **Keycloak and PostgreSQL** to start fresh  
✔️ Rebootstrapped the **admin user**  
✔️ Verified the setup by **logging into Keycloak**  

---

## 📌 Useful Commands

### 🚀 Start Keycloak
```bash
docker-compose up -d
```

### 🛑 Stop Keycloak
```bash
docker-compose down -v
```

### 🔄 Restart Keycloak
```bash
docker-compose restart keycloak
```

### 📌 Check Running Containers
```bash
docker ps
```

### 📜 View Keycloak Logs
```bash
docker-compose logs -f
```

### 🔍 Check Database Users
```bash
docker exec -it keycloak_postgres psql -U keycloak -d keycloak_db -c "SELECT username FROM user_entity;"
```

---

## 📌 Next Steps

### ✅ Push this repository to GitHub:
```bash
git init
git remote add origin https://github.com/your-username/keycloak-setup.git
git add .
git commit -m "Initial Keycloak setup"
git push -u origin main
```

### 🚀 Further Enhancements:
- ✅ Configure a **reverse proxy** (e.g., Nginx) for Keycloak  
- 🔐 Secure Keycloak with **TLS/SSL**  
- 📜 Automate admin creation with **environment variables**  

---

## 🔗 References
- [Keycloak Documentation](https://www.keycloak.org/documentation)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Docker Compose Docs](https://docs.docker.com/compose/)
