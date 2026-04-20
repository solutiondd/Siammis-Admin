# Deploy to Linux VM

This project builds a static Vite app and deploys the `dist/` output to `/var/www/siamis` on a Linux VM.

## What was added

- GitHub Actions workflow: `.github/workflows/deploy.yml`
- Nginx example config: `docs/nginx/siamis.conf.example`

## GitHub Secrets

Create these repository secrets before running the workflow:

- `VM_HOST`: Public IP or domain of the Linux VM
- `VM_PORT`: SSH port, for example `22` or `2222`
- `VM_USER`: SSH user used for deployment
- `VM_SSH_KEY`: Private SSH key for the deploy user
- `VM_TARGET_DIR`: `/var/www/siamis`

## Recommended VM setup

The deploy user must be able to write to `/var/www/siamis`.

Example setup on the VM:

```bash
sudo mkdir -p /var/www/siamis
sudo chown -R deploy:deploy /var/www/siamis
sudo chmod -R 755 /var/www/siamis
```

Replace `deploy` with your actual SSH user.

## Nginx setup

1. Copy `docs/nginx/siamis.conf.example` to the VM.
2. Save it as `/etc/nginx/sites-available/siamis.conf`.
3. Update `server_name` to your real domain.
4. Enable the site and reload Nginx.

Example commands:

```bash
sudo ln -s /etc/nginx/sites-available/siamis.conf /etc/nginx/sites-enabled/siamis.conf
sudo nginx -t
sudo systemctl reload nginx
```

## Deploy flow

1. Push code to the `main` branch, or run the workflow manually from GitHub Actions.
2. GitHub Actions installs dependencies with `npm ci`.
3. GitHub Actions builds the app with `npm run build`.
4. The workflow connects to the VM over SSH.
5. Existing files under `/var/www/siamis` are removed.
6. New files from `dist/` are uploaded to `/var/www/siamis`.

## Notes

- The current workflow assumes the app is a static frontend and does not restart any application service.
- Because this is a Vue SPA, Nginx uses `try_files ... /index.html` so client-side routes keep working.
- If the deploy user cannot write to `/var/www/siamis`, either grant access to that directory or change the workflow to upload to a temporary path and move files with `sudo`.
