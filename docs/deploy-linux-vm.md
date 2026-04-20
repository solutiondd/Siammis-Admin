# Deploy to Linux VM

This project builds a static Vite app and deploys the `dist/` output to `/var/www/siamisadmin` on a Linux VM.

## What was added

- GitHub Actions workflow: `.github/workflows/deploy.yml`
- Nginx example config: `docs/nginx/siamis.conf.example`

## GitHub Secrets

Create these repository secrets before running the workflow:

- `VM_HOST`: Public IP or domain of the Linux VM
- `VM_PORT`: SSH port, for example `22` or `2222`
- `VM_USER`: SSH user used for deployment
- `VM_SSH_KEY`: Private SSH key for the deploy user
- `VM_TARGET_DIR`: `/var/www/siamisadmin`
- `VITE_APP_BASE_URL`: Production API base URL used during `vite build`
- `VITE_IMG_PROFILE_URL`: Production image/profile base URL used during `vite build`

## Recommended VM setup

The deploy user must be able to write to `/var/www/siamisadmin`.

Example setup on the VM:

```bash
sudo mkdir -p /var/www/siamisadmin
sudo chown -R deploy:deploy /var/www/siamisadmin
sudo chmod -R 755 /var/www/siamisadmin
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
3. GitHub Actions creates `.env.production` from repository secrets.
4. GitHub Actions builds the app with `npm run build`.
5. The workflow connects to the VM over SSH.
6. Existing files under `/var/www/siamisadmin` are removed.
7. New files from `dist/` are uploaded to `/var/www/siamisadmin`.

## Notes

- The current workflow assumes the app is a static frontend and does not restart any application service.
- Because this is a Vue SPA, Nginx uses `try_files ... /index.html` so client-side routes keep working.
- If the deploy user cannot write to `/var/www/siamisadmin`, either grant access to that directory or change the workflow to upload to a temporary path and move files with `sudo`.
- The workflow writes `VITE_APP_BASE_URL` and `VITE_IMG_PROFILE_URL` into `.env.production` before running the Vite build.
