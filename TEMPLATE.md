# Rocket.Chat 8 Team Chat: composer spec

Listing name: **Rocket.Chat 8 Team Chat**
Description (70 chars): Slack-style team chat with MongoDB replica set, S3 uploads and backups
Category: Other. Icon: `https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/rocket-chat.svg`
Repository: `baranberkay96/rocketchat-railway` (this folder is the repo root).

## Services

| Service | Source | Public | Volume | Healthcheck |
|---|---|---|---|---|
| Rocket.Chat | image `rocketchat/rocket.chat:8.9.1@sha256:1809…5f0d` (official, unmodified) | domain → 3000 | none (uploads in the Bucket) | `/readyz`, 1200 s |
| MongoDB | repo `/services/mongodb` (our MongoDB 8.0.34 replica-set wrapper, from 88-mongodb) | TCP proxy 27017 | `/data/db` | — |
| MongoDB Backup | repo `/services/backup` (from 88-mongodb), cron `15 3 * * *` | no | none | — |
| NATS | image `nats:2.15.3-alpine@sha256:5583…d62a`, `nats-server --addr :: --port 4222 --http_port 8222` | no | none | — |
| Bucket | Railway Bucket `Bucket` (uploads at the root, backups under `mongodb/`) | no | — | — |

Pinned 2026-11-20: Rocket.Chat 8.9.1 (2026-11-06, newest 8.x). Rocket.Chat 9.0.0 (2026-11-10) moves LDAP/SAML/custom OAuth to premium plans and removes several login providers; staying on 8.x until 9.x has patch releases.

## Configuration

- `MONGO_URL` = `MongoDB.DATABASE_URL` (`/rocketchat?replicaSet=rs0&authSource=admin`); no `MONGO_OPLOG_URL` (Rocket.Chat 8 uses change streams).
- `TRANSPORTER=monolith+nats://nats.railway.internal:4222`, as in the official compose (`compose.nats.yml`).
- First admin: `ADMIN_USERNAME`/`ADMIN_PASS`/`ADMIN_EMAIL`/`ADMIN_NAME` (first boot with no users). `OVERWRITE_SETTING_Show_Setup_Wizard=completed`. `Accounts_RegistrationForm=Disabled` as initial value (changeable in the UI).
- Uploads: `OVERWRITE_SETTING_FileUpload_Storage_Type=AmazonS3` + `FileUpload_S3_*` from the Bucket (virtual-hosted style, proxied uploads/avatars).

## Improvements over market templates

Market (2026-11): "Rocket Chat" (6.5.3, `mongo` unpinned) ≈85 active; "Rocket.Chat [Updated Nov '26]" (`rocket.chat:latest`, `mongo:8.0`) ≈41 active; "Rocket Chat | Open Source Slack…" (`latest`) ≈32 active.

- Pinned Rocket.Chat 8.9.1 and MongoDB 8.0.34 (market: `latest` or 6.5.3, which is end-of-life).
- MongoDB as a real replica set with keyfile auth, initiated automatically (market: `mongod` started with a custom command; replica-set setup left to chance), plus nightly verified backups to a Bucket.
- Uploads in S3 (Railway Bucket) instead of GridFS in MongoDB, so Rocket.Chat is stateless and can scale; NATS included for multi-instance operation.
- Admin from variables, setup wizard skipped, public registration disabled by default (market: wizard open on the public URL).

## Resources

Measured on the 2026-11-20 test deploy (first 20-30 min, after a login, a message, a file upload and one backup run):

| Service | RAM |
|---|---|
| Rocket.Chat | 0.58-0.72 GB |
| MongoDB | 0.31-0.38 GB |
| NATS | 0.004 GB |
| MongoDB Backup | cron, 0 between runs |

Total about 1.0-1.1 GB, roughly $10-12 a month in RAM at usage prices. Rocket.Chat alone stays under 1 GB idle but has little headroom, so Hobby is the realistic minimum plan; Trial is too tight once a few users are active.

Railway prices: RAM $10/GB-month, vCPU $20/vCPU-month, volume $0.15/GB-month.

## Verification checklist

1. All services SUCCESS (MongoDB Backup = cron); `GET /readyz` 200.
2. `POST /api/v1/login` with the variables → 200, roles include admin; wrong password → 401.
3. `POST /api/v1/users.register` → refused (registration disabled).
4. Post a message to #general; upload a file with `rooms.media` + `rooms.mediaConfirm`; download it through Rocket.Chat; `FileUpload_Storage_Type` = `AmazonS3`; object visible in the Bucket.
5. Run MongoDB Backup once → `run … complete`.
6. Idle and in-use RAM per service; Trial-sized (1 GB) limit check.

## Open questions

- Push notifications to the official mobile apps go through Rocket.Chat's cloud gateway, which needs workspace registration.
