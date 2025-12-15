# DBS_CLOUD – ESTEBAN BARDOLET POMAR

This document briefly summarizes the steps I completed to prepare the project and deploy it on Google Cloud. It also clarifies which files are excluded from the repository because they are unnecessary or sensitive for this work.

## Project

- Project ID: `dbs-cloud-481310`
- Secret `DATABASE_URL`:
  `postgresql://USER:PASSWORD@/client_info?host=/cloudsql/dbs-cloud-481310:europe-west2:my-postgres-instance`

---

## 1. Set Up Google Cloud Project

I performed the initial setup in Google Cloud:

1. Created a new project in the GCP Console.
2. Identified and used the Project ID: `dbs-cloud-481310`.
3. Set the default project in `gcloud` CLI:
   ```bash
   gcloud config set project dbs-cloud-481310
   ```
4. Enabled billing for the project.

## 2. Enable Required Services

Enabled the necessary APIs for App Engine, Cloud SQL, Cloud Build and Secret Manager:

```bash
gcloud services enable cloudbuild.googleapis.com
gcloud services enable appengine.googleapis.com
gcloud services enable sqladmin.googleapis.com
gcloud services enable secretmanager.googleapis.com
```

## 3. Set Up App Engine

Initialized App Engine in region `europe-west2`:

```bash
gcloud app create --region=europe-west2
```

## 4. Set Up Cloud SQL

Created and configured a Cloud SQL (PostgreSQL) instance:

### 4.1 Create instance (public IP for testing)

```bash
gcloud sql instances create my-postgres-instance \
	--database-version=POSTGRES_17 \
	--region=europe-west2 \
	--authorized-networks=0.0.0.0/0 \
	--edition=ENTERPRISE \
	--tier=db-custom-1-3840 \
	--availability-type=ZONAL \
	--no-backup \
	--storage-size=10 \
	--storage-type=SSD \
	--no-storage-auto-increase \
	--maintenance-window-day=SUN \
	--maintenance-window-hour=4 \
	--maintenance-release-channel=production \
	--replication=asynchronous
```

### 4.2 Secure the `postgres` user password

```bash
gcloud sql users set-password postgres \
	--instance=my-postgres-instance \
	--password=YOUR_SECURE_PASSWORD
```

### 4.3 Create database and application user

```bash
gcloud sql databases create client_info --instance=my-postgres-instance

gcloud sql users create secure_user \
	--instance=my-postgres-instance \
	--password=YOUR_APP_PASSWORD
```

## 5. Store DATABASE_URL in Secret Manager

Fetched the connection name and created the `DATABASE_URL` secret:

### 5.1 Get connection name

```bash
gcloud sql instances describe my-postgres-instance --format="value(connectionName)"
```

Expected output (example): `dbs-cloud-481310:europe-west2:my-postgres-instance`

### 5.2 Create the `DATABASE_URL` secret

```bash
echo -n "postgresql://secure_user:YOUR_APP_PASSWORD@/client_info?host=/cloudsql/dbs-cloud-481310:europe-west2:my-postgres-instance" | \
gcloud secrets create DATABASE_URL --data-file=-
```

### 5.3 Verify the secret

```bash
gcloud secrets versions access latest --secret=DATABASE_URL
```

## 6. Secret Manager permissions for App Engine

Granted access for the App Engine service account to read the secret:

```bash
gcloud secrets add-iam-policy-binding DATABASE_URL \
	--member="serviceAccount:dbs-cloud-481310@appspot.gserviceaccount.com" \
	--role="roles/secretmanager.secretAccessor"
```

## 7. `app.yaml` configuration

The `app.yaml` file already exists and defines `nodejs22`, `production` mode, and basic autoscaling.

## 8. `cloudbuild.yaml` configuration

The `cloudbuild.yaml` file is present and prepared to:

- Install dependencies.
- Start Cloud SQL Proxy and run migrations with `DATABASE_URL`.
- Deploy to App Engine.

## 9. Deployment

Manual deploy:

```bash
gcloud app deploy
gcloud app browse
```

Logs:

```bash
gcloud app logs tail -s default
```

---

## Excluded or unnecessary files

The repository includes a comprehensive `.gitignore` that excludes common and sensitive items. Relevant for this project:

- `.env` (sensitive local config) – `!.env.template` is kept.
- `node_modules/` (generated dependencies).
- `*.log`, `log/`, `logs/` (log output).
- `.DS_Store`, `.vscode/` (with useful exceptions), `.idea/` (IDE files).
- `db_volume/` (local PostgreSQL data used by Docker Compose).
- `.dockerignore` is listed for Docker coherence.

No source or configuration files required for operation or deployment are deleted. Any generated artifacts (dependencies, logs, local data, IDE histories) are kept out of version control via `.gitignore`.
