# Notice

This Railway template is a community package. It is not affiliated with or endorsed by Rocket.Chat Technologies Corp., MongoDB, Inc., the NATS project, the rclone project or Railway.

## What this template runs

| Component | Source | License |
|---|---|---|
| Rocket.Chat 8.9.1 | Official image `rocketchat/rocket.chat:8.9.1@sha256:180984d4a2b0e9348484f662cf9c34f549aacbe744e0ad907ccb871e85475f0d`, used unmodified (not rebuilt, not redistributed by this repository) | MIT for the core; files under `ee/` directories are under the Rocket.Chat Enterprise License and stay inactive without a license from Rocket.Chat. This template does not modify or enable them |
| NATS Server 2.15.3 | Docker Official Image `nats:2.15.3-alpine@sha256:558394a23dd3c36f2d3a64b51865fd09908010e80a542cea5e75c2af543ed62a`, unmodified | Apache-2.0 |
| MongoDB Community Server 8.0.34, mongosh, Database Tools | Docker Official Image `mongo:8.0.34-noble@sha256:3c64937121b9437095921c4bb2569911084c05f7b88e4a263d62a8959d2b2399`, extended by our start-up files | SSPL-1.0 (server); Apache-2.0 (mongosh, Database Tools, docker-library build files) |
| rclone 1.75.2 (backup job) | `rclone/rclone:1.75.2@sha256:2687085f718d3c628f7fdfb77c52a1d344332aed543110bb88088f2c70d43eb5` | MIT, Copyright (c) 2012 Nick Craig-Wood |

Rocket.Chat license: https://github.com/RocketChat/Rocket.Chat/blob/8.9.1/LICENSE.

## Files written for this template

| File | Purpose |
|---|---|
| `services/mongodb/*` | Our MongoDB 8 replica-set service (start wrapper, replica-set initiation, keyfile, cache and oplog sizing), copied unchanged from our own "MongoDB 8 Replica Set (Prisma-Ready) with Backups" template |
| `services/backup/*` | Our MongoDB backup/restore cron job (mongodump to a Railway Bucket, verify, retention), copied unchanged from the same template |
| `template.json`, `TEMPLATE.md`, `README.md`, `NOTICE.md` | Written for this template |

## Verified against

| What | Upstream file | Ref | Checked |
|---|---|---|---|
| Releases and support (8.9.1 = newest 8.x; 9.0.0 released 2026-11-10) | GitHub releases of `RocketChat/Rocket.Chat` | `8.9.1` | 2026-11-20 |
| Engine versions (Node 22/24, MongoDB 8.0) | Release notes `8.9.1` | `8.9.1` | 2026-11-20 |
| MongoDB 5/6 dropped, replica set required, `TRANSPORTER` over NATS, compose variables (`ROOT_URL`, `PORT`, `MONGO_URL`, `DEPLOY_METHOD`) | Release notes `8.0.0`; `RocketChat/rocketchat-compose` `compose.yml`, `compose.nats.yml`, `compose.database.yml` | main | 2026-11-20 |
| Health endpoints `/livez`, `/readyz` (and deprecated `/health`) | `apps/meteor/server/routes/health.ts` | `8.9.1` | 2026-11-20 |
| First administrator from `ADMIN_USERNAME`/`ADMIN_PASS`/`ADMIN_EMAIL`, `OVERWRITE_SETTING_*` and initial setting values from the environment | Rocket.Chat server startup and settings code | `8.9.1` | 2026-11-20 |

## Reused or not reused

Market templates in `_market/` carry no license; they were read only for settings. No README text is reused.

## Trademarks

"Rocket.Chat" is a trademark of Rocket.Chat Technologies Corp. "MongoDB" is a trademark of MongoDB, Inc. NATS is a Cloud Native Computing Foundation project. "Railway" belongs to its owner. This community template is not affiliated with or endorsed by them.
