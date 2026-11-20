# Deploy and Host Rocket.Chat with Railway

Rocket.Chat is an open-source team chat platform: channels, direct messages, threads, file sharing, video calls and integrations, on a server you control. This community template runs Rocket.Chat 8.9.1 (Community edition) with a MongoDB 8.0 replica set, NATS for scaling, file uploads stored in a Railway Bucket, and daily database backups.

## About Hosting Rocket.Chat

Rocket.Chat is a Node.js application that needs MongoDB running as a replica set: it relies on change streams for real-time updates. This template ships a single-node MongoDB 8.0 replica set that initiates itself on the first boot, with keyfile authentication, a volume and a daily `mongodump` backup to a Railway Bucket with retention and a one-variable restore. Uploaded files and avatars go to the same Bucket over S3, so the Rocket.Chat service itself is stateless and can run several replicas, which talk to each other over NATS on the private network. The first administrator is created from Railway variables, the setup wizard is skipped and public sign-up starts disabled, so nobody can claim your workspace before you do.

## Common Use Cases

- Internal team chat as a self-hosted Slack or Microsoft Teams alternative
- Chat for communities, schools and open-source projects
- Customer and partner channels with guest access and federation
- ChatOps: webhooks, bots and integrations with CI, monitoring and ticketing tools
- Keeping messages and files on infrastructure you control for privacy or compliance

## Dependencies for Rocket.Chat Hosting

- Rocket.Chat 8.9.1 (`rocketchat/rocket.chat`, official image, pinned by digest)
- MongoDB 8.0 Community Server (Docker Official Image, as a single-node replica set)
- NATS 2.15 (`nats`, Docker Official Image)
- A Railway Bucket (S3-compatible) for uploads and backups

### Deployment Dependencies

- Rocket.Chat documentation: https://docs.rocket.chat/
- Source code and releases: https://github.com/RocketChat/Rocket.Chat
- Official Docker Compose setup: https://github.com/RocketChat/rocketchat-compose
- Mobile and desktop apps: https://www.rocket.chat/download-apps

### Implementation Details

| Service | Role | Public |
|---|---|---|
| Rocket.Chat | Web app, REST and realtime API for the web, desktop and mobile apps | **yes** |
| MongoDB | Single-node replica set `rs0` with keyfile auth and a volume | TCP proxy (optional, for your own tools) |
| MongoDB Backup | Daily cron job (03:15 UTC): `mongodump --oplog` to the Bucket, verified, 14-day retention | no |
| NATS | Message bus between Rocket.Chat instances | no |
| Bucket | Uploads and avatars (S3) and the backup archives | no |

**First login**

1. Wait until Rocket.Chat is healthy. The first boot creates the database and runs migrations, which takes a few minutes.
2. Copy `ADMIN_USERNAME` (default `admin`) and `ADMIN_PASS` (generated) from the Rocket.Chat service's Variables.
3. Open the Rocket.Chat service's public domain and sign in. Change the administrator's e-mail address in your profile; the default is a placeholder.
4. Add people in Administration → Users (or invite them), or open registration in Administration → Settings → Accounts → Registration.

`ADMIN_*` variables are used only on the very first boot, while the workspace has no users. After that, manage accounts in Rocket.Chat.

**Community edition.** Without a license, Rocket.Chat runs as the free Community edition, which covers chat, threads, discussions, file sharing, video calls through Jitsi or Pexip links, apps and integrations. Premium features (some enterprise authentication options, read receipts, advanced omnichannel) need a license from Rocket.Chat; this template does not unlock them. You may be asked to register the workspace with Rocket.Chat Cloud for push notifications to the mobile apps; it is optional.

**File uploads.** Files and avatars are stored in the Railway Bucket and served through Rocket.Chat, so the bucket stays private. The S3 settings are applied from variables on every start (`OVERWRITE_SETTING_FileUpload_*`); remove those variables if you prefer to manage storage in the admin UI.

**Backups and restore.** The MongoDB Backup cron job writes `mongodb/{timestamp}/mongodb.archive.gz` with a `MANIFEST.txt` to the Bucket every night and deletes runs older than 14 days (always keeping 3). To restore, set `BACKUP_RESTORE_RUN` on MongoDB Backup to a run folder, run the job once from its Cron settings, then clear the variable. Uploaded files are already in the Bucket.

**E-mail.** Configure SMTP in Administration → Settings → Email → SMTP. Railway allows outbound SMTP only on the Pro plan.

**Custom domain.** Add it in the Rocket.Chat service's Settings → Networking and set `ROOT_URL` to `https://{your-domain}`.

**Resources and plans.** Rocket.Chat is the heaviest part: plan for 0.6-1 GB of RAM for Rocket.Chat, 0.2-0.4 GB for MongoDB and a few MB for NATS, about $10-15 a month at Railway's usage prices for a small team. Rocket.Chat needs more than the 1 GB per service that Trial allows once in use, so use Hobby or Pro.

**Scaling.** Rocket.Chat has no volume, so you can raise its replica count; instances coordinate over NATS. Give MongoDB more memory before adding replicas.

**Upgrading.** Rocket.Chat is pinned by tag and digest in the service's image source; change it to a newer release to upgrade (migrations run automatically; take a backup first). Rocket.Chat 8 requires MongoDB 8.0, which this template runs.

**License.** Rocket.Chat's core is MIT-licensed; files under `ee/` directories in its source are under the Rocket.Chat Enterprise License and stay inactive without a license. This is a community template, not affiliated with Rocket.Chat Technologies Corp.

### Why Deploy Rocket.Chat on Railway?

Railway runs Rocket.Chat, its MongoDB replica set, NATS and a storage bucket in one project with a private network, HTTPS, logs and usage-based pricing. This template gives you a production-shaped chat server in minutes: replica set ready, uploads on object storage, nightly backups, and no setup wizard left open on the internet.
