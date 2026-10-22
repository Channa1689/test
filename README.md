# CHANNA video creator portfolio

A responsive bilingual portfolio and social-promotion website built with Next.js, TypeScript, Tailwind CSS, SQLite, and server-side admin authentication.

The portfolio starts with the nine TikTok links supplied by CHANNA. Each entry uses TikTok's official embedded player, which displays the video's own preview and lets visitors play it on this site without opening TikTok. Titles remain numbered because custom titles were not supplied, and views and likes are unset until verified. No client results or testimonials are pre-filled.

## What is included

- Home, portfolio, case studies, services, about, contact, and protected admin pages.
- English/Khmer interface toggle and Khmer-capable Noto Sans Khmer font.
- Portfolio text search, platform/category filters, newest/most-viewed sorting, featured content, details modal, and load-more behavior.
- Editable SQLite content for videos, case studies, services, clients, testimonials, and homepage statistics.
- Project inquiries are validated, sent to the configured CHANNA Telegram chat, and saved to SQLite for review in the admin dashboard after Telegram confirms delivery.
- Password-hashed admin accounts, signed HTTP-only session cookies, login throttling, server-side authorization, Zod validation, and protected image uploads.
- SEO metadata, sitemap, robots rules, responsive navigation, and reduced-motion support.

## Requirements

- Node.js 20.9 or newer (Node 22 LTS recommended).
- npm.

The local database and uploaded images are stored in `data/` and `public/uploads/`. Both locations are ignored by Git.

## macOS setup

Open Terminal and move into the project folder:

```bash
cd "$HOME/Desktop/new web"
```

Install dependencies and make a local environment file:

```bash
npm install
cp .env.example .env.local
```

Generate a unique session secret and put it in `.env.local` (keep this file private):

```bash
openssl rand -base64 48
```

Set `SESSION_SECRET` to the generated value. Keep the other placeholder contact values in `.env.local` until you are ready to update them. Set `NEXT_PUBLIC_SITE_URL` to your local URL for development or your HTTPS site URL after deployment.

Create your first admin account. Use a unique email and a strong password of at least 14 characters. Do not add the password to source code or commit it:

```bash
export ADMIN_EMAIL="you@example.com"
read -s "ADMIN_PASSWORD?Choose a strong admin password: "
export ADMIN_PASSWORD
npm run admin:create
unset ADMIN_EMAIL ADMIN_PASSWORD
```

The setup command creates a bcrypt password hash in the local SQLite database. It will not overwrite an existing account. Back up the database before moving/deploying it.

Start the development server:

```bash
npm run dev
```

Open <http://localhost:3000>. Admin sign-in is at <http://localhost:3000/admin/login>.

## Telegram inquiry notifications

The contact form sends each valid project inquiry from the server to the Telegram chat configured by `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID`. These are server-only variables; do not rename them with a `NEXT_PUBLIC_` prefix, put them in frontend code, or commit real values.

1. In Telegram, open the official **@BotFather**, send `/newbot`, follow its instructions, and copy the new bot token. Keep it private.
2. Open the new bot using its username and press **Start**. Telegram bots cannot start a conversation with you first.
3. Open your new bot in Telegram, press **Start**, and send it a short message. This creates the update from which Telegram reports your private chat ID.
4. In the project folder, copy `.env.example` to `.env.local` if you have not already. First put the token in `TELEGRAM_BOT_TOKEN` and save the file. Keep `.env.local` private; it is excluded from Git.
5. Get the numeric chat ID from the update for your bot using this local command. It reads the token from `.env.local` and prints only chat IDs; the token is not part of the command:

   ```bash
   node -e 'require("@next/env").loadEnvConfig(process.cwd());const token=process.env.TELEGRAM_BOT_TOKEN;if(!token||token.startsWith("replace-"))throw new Error("Set TELEGRAM_BOT_TOKEN in .env.local first.");fetch("https://api.telegram.org/bot"+token+"/getUpdates").then(async r=>{const data=await r.json();if(!r.ok||!data.ok)throw new Error("Telegram could not return updates.");const ids=[...new Set(data.result.flatMap(u=>u.message?.chat?.id==null?[]:[String(u.message.chat.id)]))];console.log(ids.join("\n")||"No chat ID found. Send a message to your bot, then run this command again.");}).catch(error=>{console.error(error.message);process.exitCode=1;});'
   ```

   Add the numeric ID for your private chat to `TELEGRAM_CHAT_ID` in `.env.local`. Do not share your bot token. If Telegram reports no updates, send another message to your bot and repeat the command.
6. Restart the local server with `npm run dev` so it loads both environment variables.
7. Submit a test inquiry on <http://localhost:3000/contact>. The success message appears only when Telegram confirms delivery. Verify the message arrived in your bot conversation.

On deployment, set `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` in the hosting provider's **server-side environment/secret settings**, then redeploy or restart the Node.js app. This project uses SQLite and local uploads, so follow the persistent Node.js host instructions below. Configure HTTPS for the production website. Never add Telegram credentials to public build variables.

The API applies server-side validation, a hidden honeypot field, a limit of five inquiries per source address per hour, a 12-second Telegram request timeout, and an idempotency key to avoid sending the same form submission twice. Telegram delivery failures show an error and keep the entered form data available for retry. The existing admin inbox stores a local copy after successful Telegram delivery.

## Editing content

1. Sign in at `/admin/login`.
2. Choose a section in the left navigation and select a record to edit.
3. For a new entry, choose **New**, update the JSON fields in the editor, and select **Save changes**.
4. Portfolio thumbnails may use HTTPS image URLs or be uploaded as JPEG, PNG, or WebP files up to 5 MB. Upload images to serve them from this site.
5. To play a TikTok portfolio video directly on this site, upload its original MP4 using **Upload playable video**. This stores the MP4 locally and uses the browser's built-in video player instead of relying on TikTok's embed service. Files must be 100 MB or smaller. A TikTok link by itself cannot be converted to a locally hosted video; obtain the original video file from the rights holder.
6. Add only client quotes, campaign details, and metrics that you are authorized to publish and have verified.

The admin content editor uses JSON to keep all record fields editable without changing code. The server checks types and limits before saving. For example, a portfolio entry looks like:

```json
{
  "id": "brand-product-launch",
  "type": "portfolio",
  "title": "Product launch film",
  "thumbnail": "https://example.com/your-thumbnail.jpg",
  "videoUrl": "https://www.tiktok.com/@yourbrand/video/...",
  "platform": "TikTok",
  "client": "Your client",
  "campaign": "Launch campaign",
  "category": "Product Review",
  "publishedAt": "2026-10-01",
  "views": null,
  "likes": null,
  "description": "A short project description.",
  "featured": true
}
```

Allowed platform values: `Facebook`, `TikTok`, `Instagram`, `YouTube`, `Other`. Video links must use HTTPS. Content IDs may contain letters, numbers, dashes, and underscores. Case studies reference related portfolio record IDs in `relatedVideoIds`.

## Commands

```bash
npm run dev        # Local development
npm run typecheck  # TypeScript validation
npm run lint       # ESLint validation
npm run build      # Production build
npm start          # Serve the production build
```

`better-sqlite3` compiles a native SQLite binding during install. Use a supported Node.js version and Xcode Command Line Tools if npm asks for a C++ compiler.

## Production deployment

This application uses SQLite and locally stored admin uploads, so it needs a **persistent Node.js host and persistent disk**. It cannot safely use a normal ephemeral serverless deployment. A Render Blueprint is included in `render.yaml`; its persistent disk stores both SQLite and future Admin uploads. The `/uploads/...` route serves those files from the mounted disk, including byte-range streaming for MP4 playback.

### Deploy to Render

1. Create a new **private** GitHub repository for this portfolio. Keep it separate from unrelated repositories and home-directory files. From this project folder, initialize Git, inspect the staged files, then commit and push:

   ```bash
   git init -b main
   git add .
   git status --short
   git commit -m "Prepare CHANNA portfolio for deployment"
   git remote add origin https://github.com/YOUR-ACCOUNT/YOUR-PRIVATE-REPO.git
   git push -u origin main
   ```

   Confirm `.env.local`, `data/channa.sqlite`, `node_modules`, `.next`, original source-photo folders, and uploaded files are absent from `git status --short`. Never add real credentials or private customer data to Git.
2. In Render, choose **New → Blueprint**, connect the new private GitHub repository, and select `render.yaml`. The Blueprint selects a paid Node web service and a 1 GB persistent disk; check Render's current pricing before confirming.
3. Set the requested `NEXT_PUBLIC_SITE_URL` to the HTTPS Render URL once Render assigns the service URL. If the initial Blueprint asks for it before creating the service, use the intended `https://<your-service-name>.onrender.com` URL. Set `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` in Render's private environment settings only when ready to enable Telegram notifications; they are deliberately not included in the Blueprint.
4. Wait for the deployment and confirm the service health check at `/api/health`. The first start creates the SQLite database and seeds the portfolio/services. The persistent disk is mounted at `/var/data`; SQLite uses `/var/data/channa.sqlite` and uploads use `/var/data/uploads`.
5. Create the first admin account from the Render service's Shell. Enter the email and use a hidden prompt for the password:

   ```bash
   export ADMIN_EMAIL="you@example.com"
   read -s ADMIN_PASSWORD
   export ADMIN_PASSWORD
   npm run admin:create
   unset ADMIN_EMAIL ADMIN_PASSWORD
   ```

6. Visit `/admin/login`, sign in, and test a thumbnail/MP4 upload. Confirm the uploaded image/video remains available after a later deploy. Add a custom domain in Render if desired and configure its DNS as Render instructs; keep HTTPS enabled.
7. Back up the persistent disk/database and uploaded media regularly. Do not resize or replace the disk without first backing up its contents. Check Render's current disk backup options and retention limits.

The project also works on a VPS or a Node container with a persistent volume: install Node.js 20.9+, run `npm ci`, set `NODE_ENV=production`, `SESSION_SECRET`, `DATABASE_PATH`, `UPLOADS_DIRECTORY`, and `NEXT_PUBLIC_SITE_URL`, then run `npm run build` and `npm start` behind an HTTPS reverse proxy. Set `DATABASE_PATH` and `UPLOADS_DIRECTORY` to locations on the same persistent storage. Configure the proxy to discard untrusted incoming `X-Forwarded-For` headers before setting its own client address; these addresses are used for rate limiting.

The contact page and footer use the supplied CHANNA email, Telegram, and Facebook links by default. To change them, set `NEXT_PUBLIC_CONTACT_EMAIL`, `NEXT_PUBLIC_TELEGRAM_URL`, and `NEXT_PUBLIC_FACEBOOK_URL`; rebuild after changing public variables.

Keep `.env.local`, production environment variables, database files, and uploaded private material out of Git. HTTPS is required in production so secure admin cookies are transmitted safely. Use a firewall and regular operating-system updates on self-hosted servers.

## Troubleshooting

- **Admin login says session secret is missing:** Set `SESSION_SECRET` in `.env.local`, restart Next.js, and use a random value at least 32 characters long.
- **No admin account yet:** Set `ADMIN_EMAIL` and `ADMIN_PASSWORD` in the shell and run `npm run admin:create`.
- **Native SQLite module fails to install:** Check `node --version` and install Apple's Xcode Command Line Tools, then run `npm install` again.
- **Changes are not visible:** Refresh the relevant admin collection; the public site reads SQLite content on each request. Restart `npm run dev` after changing environment variables.
- **Uploaded images disappear after deployment:** Set `UPLOADS_DIRECTORY` to persistent storage and make sure the `/uploads/...` route is using the same configured directory as the admin upload endpoints.
- **Render Blueprint or health check fails:** Confirm the Blueprint is using `render.yaml`, the paid persistent disk is mounted at `/var/data`, and `/api/health` returns `{"status":"ok"}`. Review the Render deploy logs for the first failing build or start step.
- **Telegram inquiry notifications are not configured:** Create a bot with @BotFather, start a conversation with it, and set `TELEGRAM_BOT_TOKEN` and your numeric `TELEGRAM_CHAT_ID` in `.env.local` or the production host's secret manager. Restart/redeploy after changing environment variables.
- **Telegram notification failed:** Confirm you pressed Start in your bot conversation, the chat ID is numeric and belongs to that conversation, and the bot token is valid. Check the server is allowed to make outbound HTTPS requests to `api.telegram.org`. The form keeps its entries so you can retry.
