![9Drive cover](https://i.ibb.co.com/35BySv1C/image.png)

# 9Drive

9Drive is a storage gateway web app for connecting multiple Google Drive accounts into one virtual storage dashboard. Users can register with email/password or Google, automatically connect their first Google Drive account during Google sign-in, track quota, upload files into a dedicated `9drive` Drive folder, organize files with virtual folders, preview files, sync MySQL from Google Drive, and let the backend route uploads to the Drive account with enough free space.

## License

Copyright 2026 Zenhosta

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for details.

## Features

- Google Drive and S3-compatible storage gateway in one virtual storage dashboard.
- S3-compatible storage support with custom endpoints for providers like MinIO, Cloudflare R2, Wasabi, Backblaze B2, and AWS S3.
- Direct upload stream to Google Drive. Files are not stored on the server.
- Google Drive uploads are stored under a root `9drive` folder.
- Direct upload stream to S3-compatible storage through the backend without exposing storage credentials to the frontend.
- Upload routing policies with most-available, round-robin, and priority-order modes.
- External upload API using API keys at `POST /api/v1/uploads`.
- API key management with one-time secret display, hashed key storage, last-used tracking, and revocation.
- Email/password auth plus Google sign-in/register with automatic first Drive connection.
- Multi-account storage quota summary.
- Quota tracker page.
- Manual sync from the Google Drive `9drive` folder back into MySQL.
- Virtual folders.
- File preview, download, rename, move, and delete actions.
- In-app API documentation with cURL and JavaScript upload examples.
- Bottom-right upload progress panel.
- Bearer token authentication.
- Global Google OAuth config stored encrypted in DB (can be set via seed command or directly in Settings UI).
- Automated system updates via `update.sh` directly from the Settings UI (PM2 setup).
- Optional reCAPTCHA on email/password registration.
- MySQL database with Prisma migrations.
- Express + TypeScript backend.
- React + Vite frontend.

## Preview

Live preview: https://9drive.zenhosta.com

![9Drive dashboard preview](https://i.ibb.co.com/HLjG3JRf/image.png)

![9Drive shared file preview](https://i.ibb.co.com/QLpYGmx/image.png)

## Star History

[![Star History Chart](https://star-history.dera.page/svg?repos=zenhosta/9drive&type=Date)](https://star-history.dera.page/#zenhosta/9drive&type=Date)

## Project Structure

```txt
backend/   Express API, Prisma schema, Google Drive integration
frontend/  Vite React app
```

## Requirements

- Node.js 20+
- npm
- MySQL running locally
- Google Cloud project
- Google OAuth Client ID and Client Secret

Default database used by this project:

```txt
host: localhost
port: 3306
database: 9drive
user: root
password: empty
```

## 1. Quick Setup & Installation (Recommended)

The easiest way to prepare the project is using the automated setup script. It generates local environment files with secure keys, installs dependencies, and generates the Prisma client. MySQL 8+ must already be running for local development; Docker setup includes MySQL.

### Windows (PowerShell)
Make sure to open PowerShell and navigate to the project directory first. For example, if you cloned the project to `E:\AUTO KLIK\9Drive`:

```powershell
# 1. Switch to the drive where the project is located (if necessary)
E:

# 2. Navigate to the project folder
cd "E:\AUTO KLIK\9Drive"

# 3. Run the automated setup script
powershell -ExecutionPolicy Bypass -File .\setup.ps1
```

### Linux/macOS

```bash
bash ./setup.sh
```

Enter your MySQL connection URL when prompted. Google Client ID/Secret can be skipped and configured later.

Once setup is complete, apply migrations and start backend and frontend in separate terminals:

```bash
cd backend
npm run prisma:migrate
npm run dev
```

```bash
cd frontend
npm run dev
```

---

## 2. Manual Installation (Alternative)

If you prefer to configure the project manually:

### 2.1 Install Dependencies
Install backend dependencies:

```bash
cd backend
npm install
```

Install frontend dependencies:

```bash
cd ../frontend
npm install
```

### 2.2 Create Database (For MySQL)
Create a database:
```sql
CREATE DATABASE 9drive;
```
If using MySQL CLI:
```bash
mysql -u root -e "CREATE DATABASE IF NOT EXISTS 9drive;"
```

### 2.3 Environment Setup


Create `backend/.env`:

```env
DATABASE_URL="mysql://root@localhost:3306/9drive"
APP_PORT=4000
FRONTEND_URL="http://localhost:5173"
JWT_ACCESS_SECRET="change-this-jwt-secret-at-least-32-chars"
TOKEN_ENCRYPTION_KEY="change-this-encryption-key-32bytes!"
ACCESS_TOKEN_TTL_SECONDS=900
REFRESH_TOKEN_TTL_DAYS=30
MAX_UPLOAD_BYTES=5368709120
RECAPTCHA_SECRET_KEY=""

# Used only by `npm run seed:google-config`.
# These values are encrypted and stored in DB as global Google OAuth config.
GOOGLE_CLIENT_ID=""
GOOGLE_CLIENT_SECRET=""
GOOGLE_REDIRECT_URI="http://localhost:4000/connected-accounts/google/callback"
```

Important:

- `JWT_ACCESS_SECRET` should be long and random.
- `TOKEN_ENCRYPTION_KEY` should be long and random.
- Do not commit `backend/.env`.
- Google OAuth credentials are used by the seed script, then stored encrypted in the database.

## 4. Frontend Environment

Create or confirm `frontend/.env`:

```env
VITE_API_URL=http://localhost:4000
VITE_RECAPTCHA_SITE_KEY=
```

Captcha is disabled when `VITE_RECAPTCHA_SITE_KEY` or backend `RECAPTCHA_SECRET_KEY` is empty. Set both values to enable captcha on registration.

## 5. Run Prisma Migrations

```bash
cd backend
npm run prisma:migrate
```

If Prisma client generation is blocked on Windows by a running Node process, stop running backend/frontend dev servers and run:

```bash
npx prisma generate
```

## 6. Google Cloud Setup

Google setup is done in Google Cloud Console, not Google Search Console. Google Search Console is for website indexing/search ownership. OAuth and Drive API are managed in Google Cloud Console.

Open Google Cloud Console:

```txt
https://console.cloud.google.com/
```

### 6.1 Create Or Select Project

1. Open Google Cloud Console.
2. Click project selector in top bar.
3. Create a new project or select an existing project.
4. Remember the project name because OAuth client and Drive API must be in the same project.

### 6.2 Enable Google Drive API

1. Go to:

```txt
APIs & Services -> Library
```

2. Search:

```txt
Google Drive API
```

3. Open `Google Drive API`.
4. Click `Enable`.
5. Wait a few minutes if Google says the API was enabled recently.

Direct URL pattern:

```txt
https://console.developers.google.com/apis/api/drive.googleapis.com/overview?project=YOUR_PROJECT_ID
```

If Google Drive API is disabled, you will see an error like:

```txt
Google Drive API has not been used in project ... before or it is disabled.
```

### 6.3 Configure OAuth Consent Screen

1. Go to:

```txt
APIs & Services -> OAuth consent screen
```

2. Choose app type:

```txt
External
```

3. Fill required fields:

```txt
App name
User support email
Developer contact email
```

4. Add scopes:

```txt
https://www.googleapis.com/auth/drive.file
https://www.googleapis.com/auth/userinfo.email
https://www.googleapis.com/auth/userinfo.profile
```

`drive.file` is a non-sensitive scope, so it does not require Google verification. It only grants access to files and folders created by this app. Files added manually to the `9drive` folder from outside the app are not visible to `POST /files/sync-google`. Existing connected accounts must reconnect to receive the new scope.

5. If publishing status is `Testing`, add test users.

Add every Google account that will test the app:

```txt
OAuth consent screen -> Test users -> Add users
```

If you do not add test users, Google may show:

```txt
Access blocked: app has not completed the Google verification process
Error 403: access_denied
```

### 6.4 Create OAuth Client

1. Go to:

```txt
APIs & Services -> Credentials
```

2. Click:

```txt
Create Credentials -> OAuth client ID
```

3. Application type:

```txt
Web application
```

4. Add authorized JavaScript origin:

```txt
http://localhost:5173
```

5. Add authorized redirect URI:

```txt
http://localhost:4000/connected-accounts/google/callback
```

6. Click Create.
7. Copy:

```txt
Client ID
Client Secret
```

### 6.5 Seed Google OAuth Config

Put values into `backend/.env`:

```env
GOOGLE_CLIENT_ID="your-client-id"
GOOGLE_CLIENT_SECRET="your-client-secret"
GOOGLE_REDIRECT_URI="http://localhost:4000/connected-accounts/google/callback"
```

Then run:

```bash
cd backend
npm run seed:google-config
```

This stores the Google OAuth config as a global encrypted provider config in MySQL. Google sign-in uses the same config and automatically connects the first Drive account. Logged-in users can still click `Connect Drive` in Settings to add more Drive accounts.

## 7. Run Development Servers

Start backend:

```bash
cd backend
npm run dev
```

Backend runs at:

```txt
http://localhost:4000
```

Start frontend:

```bash
cd frontend
npm run dev
```

Frontend runs at:

```txt
http://localhost:5173
```

## Docker Deployment

Docker levanta MySQL, backend y frontend. Las migraciones y el seed de Google corren solos al arrancar el backend.

### 1. Configurar `.env`

```bash
cp .env.docker.example .env
```

En Windows PowerShell:

```powershell
Copy-Item .env.docker.example .env
```

Cambia como mínimo:

- `MYSQL_ROOT_PASSWORD`, `MYSQL_PASSWORD`
- `JWT_ACCESS_SECRET`, `TOKEN_ENCRYPTION_KEY` (la clave de cifrado debe tener al menos 32 caracteres)
- `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`

Captcha queda desactivado si `VITE_RECAPTCHA_SITE_KEY` o `RECAPTCHA_SECRET_KEY` están vacíos.

### 2. Levantar

```bash
docker compose up -d --build
```

- Frontend: http://localhost:5173
- Backend: http://localhost:4000

Si cambias `VITE_API_URL` o `VITE_RECAPTCHA_SITE_KEY`, reconstruye con el mismo comando. Vite los embebe al compilar.

### 3. Logs y parada

```bash
docker compose logs -f backend
docker compose down
```

`docker compose down -v` también borra la base de datos.

### 4. Actualizar credenciales de Google

Si cambias `GOOGLE_CLIENT_ID` o `GOOGLE_CLIENT_SECRET` con los contenedores ya corriendo:

```bash
docker compose up -d --build
docker compose exec backend npm run seed:google-config
```

### 5. HTTPS en un VPS (opcional)

1. Apunta dos dominios (o subdominios) a la IP del servidor, por ejemplo `9drive.tudominio.com` y `api.9drive.tudominio.com`.
2. En `.env`:

   ```env
   FRONTEND_DOMAIN=9drive.tudominio.com
   API_DOMAIN=api.9drive.tudominio.com
   FRONTEND_URL=https://9drive.tudominio.com
   VITE_API_URL=https://api.9drive.tudominio.com
   GOOGLE_REDIRECT_URI=https://api.9drive.tudominio.com/connected-accounts/google/callback
   ```

3. Levanta con el perfil `https`. Caddy obtiene los certificados solo:

   ```bash
   docker compose --profile https up -d --build
   ```

4. En Google Cloud Console, en el cliente OAuth Web:
   - Authorized JavaScript origins: `https://9drive.tudominio.com`
   - Authorized redirect URIs: `https://api.9drive.tudominio.com/connected-accounts/google/callback`

Cuando cambies `VITE_API_URL`, reconstruye el frontend con `docker compose --profile https up -d --build`.

### Notas de producción

- No expongas el puerto de MySQL (el compose no lo publica).
- Usa secretos largos y aleatorios en `JWT_ACCESS_SECRET` y `TOKEN_ENCRYPTION_KEY`.
- Si no usas el perfil `https`, pon tu propio proxy HTTPS delante de 5173 y 4000.

## Despliegue en Dokploy

Usa [docker-compose.dokploy.yml](docker-compose.dokploy.yml). Dokploy pone HTTPS y dominios con Traefik, así que ese archivo no publica puertos ni incluye Caddy.

### 1. Crear el servicio

1. En Dokploy, crea un proyecto y un servicio de tipo **Compose**.
2. Fuente: repositorio Git de este proyecto, rama `main`.
3. Compose path: `docker-compose.dokploy.yml`.

### 2. Variables de entorno

Pestaña **Environment**. Usa la plantilla de [.env.docker.example](.env.docker.example) y define como mínimo:

```env
MYSQL_ROOT_PASSWORD=...
MYSQL_PASSWORD=...
JWT_ACCESS_SECRET=...
TOKEN_ENCRYPTION_KEY=...           # al menos 32 caracteres
FRONTEND_URL=https://9drive.tudominio.com
VITE_API_URL=https://api.9drive.tudominio.com
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
GOOGLE_REDIRECT_URI=https://api.9drive.tudominio.com/connected-accounts/google/callback
```

`VITE_API_URL` se embebe al compilar. Si la cambias, vuelve a desplegar con rebuild.

### 3. Dominios

Pestaña **Domains**, dos entradas con HTTPS activado:

| Servicio | Host | Puerto |
|---|---|---|
| `frontend` | `9drive.tudominio.com` | `80` |
| `backend` | `api.9drive.tudominio.com` | `4000` |

Antes, apunta los dos registros DNS a la IP del servidor.

### 4. Google Cloud Console

En el cliente OAuth Web:

- Authorized JavaScript origins: `https://9drive.tudominio.com`
- Authorized redirect URIs: `https://api.9drive.tudominio.com/connected-accounts/google/callback`

### 5. Desplegar

Pulsa **Deploy**. Las migraciones y el seed de Google corren solos al arrancar el backend. Para revisar, mira los logs del servicio `backend`.

### Non-Docker Production Startup

Run production migrations before starting the backend:

```bash
cd backend
npm run db:migrate:deploy
npm run start
```

Or use the combined command:

```bash
cd backend
npm run start:deploy
```

`npm run db:migrate:deploy` uses Prisma production migrations and does not reset the database. If Prisma reports migration drift, stop the deploy and repair migration history first; do not run `prisma migrate reset` on production.

## 8. Manual Test Flow

1. Open frontend:

```txt
http://localhost:5173
```

2. Register a user with email/password and captcha, or click `Continue with Google and connect Drive`.
3. If using Google sign-in, approve Drive access once and confirm `/settings` already shows the connected account.
4. If using email/password, open `Settings`, click `Connect Drive`, approve access, and confirm the account appears.
5. Open `Quota Tracker`.
6. Confirm quota appears.
7. Open `All Files`.
8. Create nested virtual folders.
9. Upload a file and confirm it appears under Google Drive root folder `9drive`.
10. Add or remove a file manually inside Google Drive folder `9drive`, then click `Sync Drive` in All Files.
11. Watch bottom-right upload progress.
12. Right-click file row for actions:

```txt
View
Download
Rename
Move to Folder
Delete
```

## API Overview

Auth:

```txt
POST /auth/register
POST /auth/login
GET /auth/google/url
GET /auth/google/callback
POST /auth/google/exchange
POST /auth/refresh
POST /auth/logout
GET /auth/me
```

Google accounts:

```txt
GET /connected-accounts/google/connect-url
GET /connected-accounts/google/callback
GET /connected-accounts
POST /connected-accounts/:id/sync-quota
DELETE /connected-accounts/:id
```

Storage:

```txt
GET /storage/summary
```

Folders:

```txt
GET /folders
GET /folders/recent?limit=4
POST /folders
DELETE /folders/:id
```

Files:

```txt
GET /files
GET /files?folderId=<id>
GET /files?q=<search>
GET /files/shared-links
GET /files/:id
PATCH /files/:id
PATCH /files/batch
DELETE /files/batch
POST /files/sync-google
POST /files/:id/share
DELETE /files/:id/share
POST /files/:id/preview-token
GET /files/:id/view-url
GET /files/:id/download
DELETE /files/:id
GET /files/preview/:token
```

Uploads:

```txt
POST /uploads
```

Upload is `multipart/form-data`. Metadata fields should be appended before the file:

```txt
sizeBytes
fileName
mimeType
folderId optional
file
```

## Security Notes

- Backend never stores uploaded files on disk.
- Uploads are streamed through the backend to Google Drive folder `9drive`.
- Google tokens are encrypted in MySQL.
- Refresh tokens for app sessions are hashed in MySQL.
- Google auth handoff tokens, public share tokens, and preview tokens are hashed before lookup/use.
- `backend/.env` is ignored by git.
- Do not expose `TOKEN_ENCRYPTION_KEY`, `JWT_ACCESS_SECRET`, `RECAPTCHA_SECRET_KEY`, OAuth client secrets, or raw share/preview/handoff tokens.

## Production Notes

- Replace localhost redirect URIs with production URLs.
- Add production domain to Google OAuth authorized origins.
- Set OAuth consent screen to production when ready.
- Google may require verification for public apps.
- Use strong secrets.
- Put the backend behind HTTPS.
- Consider secure cookies or stronger token storage for production.

## Google OAuth Configuration via UI

Instead of seeding Google credentials manually using `npm run seed:google-config`, you can set them up directly from the frontend dashboard:
1. Log in to the dashboard.
2. Go to **Settings** -> **Google Credentials**.
3. Input your **Google Client ID**, **Google Client Secret**, and **Redirect URI** (e.g. `https://103.65.237.136.nip.io:4000/connected-accounts/google/callback`).
4. Click **Save Configuration**.

The config is automatically encrypted and saved into the database, enabling Google sign-in and Google Drive connections instantly.

## Automated Updates & PM2 Management

For native VPS setups running with PM2, 9Drive includes a fully automated system update trigger and log monitor in the **Settings** UI.

### How it works
1. When you trigger an update from the frontend dashboard, the backend triggers the `update.sh` script in the background.
2. The script:
   - Resets any local Git conflicts (`git reset --hard`) and pulls the latest changes.
   - Installs dependencies and builds both backend and frontend.
   - Deploys Prisma database migrations.
   - Restarts the backend process using PM2 (`pm2 restart 9drive-backend`).
3. You can monitor the real-time rebuild progress using the log viewer inside the Settings UI.

### Manual update command
If you want to update manually via the terminal, run:
```bash
./update.sh
```
Or run the commands individually:
```bash
git reset --hard
git pull origin main
cd backend && npm install && npx prisma generate && npm run build && npx prisma migrate deploy
cd ../frontend && npm install && npm run build
pm2 restart 9drive-backend
```

## Build

Backend:

```bash
cd backend
npm run build
```

Frontend:

```bash
cd frontend
npm run build
```
