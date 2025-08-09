# Convex Self-Hosting Automation (DigitalOcean + Nginx + HTTPS)

This repository contains an Ansible playbook that provisions a **production-ready Convex self-hosted backend** on a fresh Ubuntu DigitalOcean droplet.

It automates:

- Installing Docker (official repo) + Compose plugin
- Pulling & running Convex backend + dashboard in Docker (`/opt/convex`)
- Installing & configuring Nginx reverse proxy with HTTPS via Let’s Encrypt
- Mapping:
  - `api.<domain>` → Convex API (port 3210 internally)
  - `<domain>` → Convex HTTP actions (port 3211 internally)
  - `dashboard.<domain>` → Convex dashboard (port 6791 internally)
- Locking down raw ports with UFW firewall rules
- Generating an admin key and saving it to `/root/convex_admin_key.txt`

---

## 📋 Requirements

- Fresh Ubuntu 22.04+ droplet (tested on DigitalOcean)
- Your domain’s DNS A records for:
  - `@` → droplet IP
  - `api` → droplet IP
  - `dashboard` → droplet IP
- Local machine with Ansible installed (`pip install ansible` or `brew install ansible`)

---

## 🚀 Setup

### 1. Clone the repo

```bash
git clone <your-private-repo-url>
cd convex-selfhost-ansible
```

### 2. Edit `inventory.ini`

Replace with your droplet’s IP or hostname:

```ini
[convex]
<droplet-ip> ansible_user=root
```

### 3. Edit variables in `site.yml`

At the top of `site.yml`:

```yaml
droplet_ip: "x.x.x.x" # droplet IP
domain_base: "englishbystories.com" # your domain
letsencrypt_email: "you@example.com" # certbot email
```

Optional:

```yaml
enable_dashboard_basic_auth: true # password-protect dashboard
dashboard_basic_auth_user: "admin"
dashboard_basic_auth_password: "change-me"
```

### 4. Run the playbook

```bash
ansible-playbook -i inventory.ini site.yml
```

The playbook will:

- Install Docker + Nginx
- Deploy Convex backend + dashboard
- Issue HTTPS certificates
- Lock down internal ports
- Save the admin key to `/root/convex_admin_key.txt`

---

## 🔑 Getting the admin key

The key is printed at the end of the run and stored on the server:

```bash
cat /root/convex_admin_key.txt
```

**Treat this like a password** — anyone with this key can push to your backend.

---

## 💻 Connecting your local Convex project

In your Convex project:

```bash
npm install convex@latest
```

Create `.env.local`:

```env
CONVEX_SELF_HOSTED_URL=https://api.englishbystories.com
CONVEX_SELF_HOSTED_ADMIN_KEY=<your-admin-key>
```

Push code:

```bash
npx convex dev     # watch & push changes live
npx convex deploy  # deploy a fixed version
```

---

## 🔄 Updating the server

### Update Convex container images:

```bash
ssh root@<droplet-ip>
cd /opt/convex
docker compose pull && docker compose up -d
```

### Regenerate admin key:

```bash
cd /opt/convex
docker compose exec backend ./generate_admin_key.sh
```

Update your `.env.local` with the new key.

---

## 📦 Backups

Export all Convex data:

```bash
npx convex export --path convex-backup.snapshot
```

Restore:

```bash
npx convex import --replace-all convex-backup.snapshot
```

---

## 🛡 Security

- UFW allows only 22 (SSH), 80 (HTTP), and 443 (HTTPS)
- Internal Convex ports (3210, 3211, 6791) are blocked externally
- Optional Basic Auth can be enabled for the dashboard

---

## 📌 Notes

- Default DB is SQLite inside a Docker volume (good for small/medium workloads)
- For production-grade DB persistence, switch to managed Postgres/MySQL and set `POSTGRES_URL` or `MYSQL_URL` in `/opt/convex/.env`
- Certbot auto-renews certificates
- Works best on a fresh droplet

---

## 🆘 Help

- Convex docs: [https://docs.convex.dev/](https://docs.convex.dev/)
- Convex Discord: [https://discord.gg/convex](https://discord.gg/convex)

```

```
