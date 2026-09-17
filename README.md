# subsiv

Public repository that generates one VLESS subscription file per route/tag. Each subscription contains multiple VLESS configs (one per path). All links share the same strict SNI and Host values (loaded exclusively from GitHub Secrets) while every individual link receives a fresh random UUIDv4. Remarks follow the pattern `sivd1`, `sivd2`, … (label + 1-based path index, reset per subscription file).

Sensitive values never appear in any committed file. Build runs in GitHub Actions; output is published only to the `gh-pages` branch.

## Repo layout

```
/
├── .github/
│   └── workflows/
│       └── build-subscription.yml
├── parts/
│   ├── protocol.txt
│   ├── port.txt
│   ├── type.txt
│   ├── security.txt
│   ├── encryption.txt
│   ├── fingerprint.txt
│   ├── label.txt
│   ├── paths.txt
│   └── routes.txt
├── CNAME
├── .gitignore
└── README.md
```

## parts/ files

| File | Purpose |
|------|---------|
| `protocol.txt` | Protocol string (default: `vless`) |
| `port.txt` | Port number (default: `443`) |
| `type.txt` | Transport type (default: `ws`) |
| `security.txt` | Security (default: `tls`) |
| `encryption.txt` | Encryption (default: `none`) |
| `fingerprint.txt` | TLS fingerprint (default: `chrome`) |
| `label.txt` | Remark prefix (default: `sivd`) |
| `paths.txt` | One `ip:port` per line (no leading `/`). Blank lines and `#` comments ignored. |
| `routes.txt` | One `Tag Address` per line. Tag becomes the subscription filename; Address is used in the VLESS URI. Duplicate tags cause the build to fail. |

## How to use

### Add a new route
Append a line to `parts/routes.txt`:
```
NewTag 1.2.3.4
```
Commit / push. The workflow will create `newtagsubs.txt`.

### Add or remove a path
Edit `parts/paths.txt`. Each non-comment line becomes one VLESS entry inside every subscription.

### Rotate SNI / HOST
Change the GitHub repository secrets `SNI` and `HOST`. No code change required. Next build will pick up the new values.

### Subscription URL pattern
```
https://<custom-domain>/<lowercase-tag>subs.txt
```
Example: `https://sub.example.com/dixisubs.txt`

### Manual trigger
GitHub → Actions → "Build Subscription" → Run workflow

## Cloudflare DNS setup

1. In Cloudflare DNS, create a **CNAME** record:
   - Name: your subdomain (e.g. `sub`)
   - Target: `durgaa17.github.io`
   - Proxy status: **Proxied** (orange cloud) if you want Cloudflare caching / WAF / DDoS protection.

2. SSL/TLS mode: set to **Full (strict)**.

3. Wait for GitHub Pages to detect the custom domain and issue a certificate (or let Cloudflare terminate TLS).

4. After the first successful deploy, the files will be available at `https://your-domain/<tag>subs.txt`.

## Secrets (configure manually)

Repository Settings → Secrets and variables → Actions:

- `SNI` — strict SNI value used in every generated link
- `HOST` — strict Host header value used in every generated link

These values are never written to any file and are masked in workflow logs.

## Notes

- The repository is public and contains **no secrets**.
- Generated `*subs.txt` files live only on the `gh-pages` branch.
- Windows (CRLF) line endings in `parts/` files are handled automatically.
- Path values are normalized (leading `/` added if missing) and URL-encoded before insertion into the VLESS URI.
- Base64 output is single-line (`base64 -w0`).
