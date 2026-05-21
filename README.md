# zyrn.ai

Marketing + documentation site for [Zyrn](https://github.com/shammyali/zyrn) — the AI-native database.

## Stack

Single static `index.html` with embedded CSS. No build step. No JS framework.
Loads in <100KB. Mobile responsive. Light theme (Porkbun warmth).

## Deploy

### Option A: GitHub Pages (current)

1. Settings → Pages → Source: `main` branch, `/` root
2. The `CNAME` file pins the custom domain to `zyrn.ai`
3. At Porkbun (DNS for zyrn.ai), set A records to GitHub Pages IPs:
   - `185.199.108.153`
   - `185.199.109.153`
   - `185.199.110.153`
   - `185.199.111.153`
4. Plus a `CNAME` record on `www`:
   - `www` → `shammyali.github.io`
5. Wait ~10 min for DNS + GitHub's auto-HTTPS provisioning

#### Post-deploy verification (one-shot smoke check)

After any deploy or DNS change, verify HTTPS resolves cleanly. The Pages cert
provisioning step has shipped silently broken before:

```bash
# Should print: HTTP: 200  TLS_verify: 0
curl -sS -o /dev/null -w "HTTP: %{http_code}  TLS_verify: %{ssl_verify_result}\n" https://zyrn.ai/

# Should be CN=zyrn.ai (NOT *.github.io)
echo | openssl s_client -servername zyrn.ai -connect zyrn.ai:443 2>/dev/null \
  | openssl x509 -noout -subject
```

If TLS returns the wildcard `*.github.io` cert, bounce the CNAME via the API
to force Pages to re-provision Let's Encrypt:

```bash
gh api -X PUT repos/shammyali/zyrn-site/pages -f cname=''
sleep 30
gh api -X PUT repos/shammyali/zyrn-site/pages -f cname=zyrn.ai
# wait for the build to finish, then:
gh api -X PUT repos/shammyali/zyrn-site/pages -F https_enforced=true
```

### Option B: Cloudflare Pages

1. Connect this repo to Cloudflare Pages
2. Build command: (none)
3. Output directory: `/`
4. Move DNS for `zyrn.ai` from Porkbun to Cloudflare nameservers (Cloudflare gives instructions on the project page)
5. Cloudflare auto-provisions SSL

Cloudflare Pages is faster globally; GitHub Pages is simpler if DNS already at Porkbun.

## Editing

Just edit `index.html`. There is no preprocessor, no template engine, no
component framework. The whole site is one file you can grep, sed, and ship.

To preview locally:

```bash
python3 -m http.server 8080
# open http://localhost:8080
```

## License

Apache-2.0. Site content + design © Logos Technologies LLC.
