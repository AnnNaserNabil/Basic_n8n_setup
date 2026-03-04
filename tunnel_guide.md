# Setting Up a Permanent Cloudflare Tunnel

Cloudflare Tunnels provide a secure way to expose your local services to the internet without opening ports on your router or exposing your public IP. This guide covers setting up a permanent tunnel with a custom domain.

## Prerequisites

- A [Cloudflare account](https://dash.cloudflare.com/sign-up) (verified via email).
- A registered domain name (e.g., from GoDaddy, Namecheap, or Hostinger).
- `cloudflared` CLI installed on your local machine.
- A local service running (e.g., a Next.js app on port 3000).

---

## 1. Onboarding Your Domain to Cloudflare

Cloudflare Tunnel requires your domain's DNS to be managed by Cloudflare.

1.  Log in to the [Cloudflare Dashboard](https://dash.cloudflare.com/).
2.  Click **Add a Site** (or **Onboard a domain**).
3.  Enter your domain name (e.g., `example.com`) and click **Continue**.
4.  Select the **Free Plan** and click **Continue**.
5.  Cloudflare will scan for existing DNS records. Review them and click **Continue**.

## 2. Updating Nameservers

To finalize the transfer, you must update the nameservers at your domain registrar (e.g., GoDaddy).

1.  Cloudflare will provide two nameservers (e.g., `dash.ns.cloudflare.com`).
2.  Log in to your domain registrar's dashboard.
3.  Find the **DNS Management** or **Nameservers** section for your domain.
4.  Select **Use my own nameservers** and replace the existing ones with the Cloudflare nameservers.
5.  Save the changes. It may take anywhere from a few minutes to 24 hours for DNS changes to propagate.
6.  Once active, your domain status in Cloudflare will show as **Active**.

## 3. Authorizing the CLI

Authorize the `cloudflared` CLI to manage tunnels on your account.

```bash
cloudflared tunnel login
```

A browser window will open. Select your domain and grant permission.

## 4. Creating the Tunnel

Create a new tunnel and note the generated Tunnel ID.

```bash
cloudflared tunnel create <tunnel-name>
```

*Example:* `cloudflared tunnel create my-server`

Cloudflare will generate a unique Tunnel ID (e.g., `1ed8140...`) and a credentials file (JSON) in your `~/.cloudflared` directory.

## 5. Routing the Subdomain

Map a subdomain to your tunnel in the Cloudflare DNS settings.

```bash
cloudflared tunnel route dns <tunnel-name> <subdomain.yourdomain.com>
```

*Example:* `cloudflared tunnel route dns my-server demo.example.com`

## 6. Configuration

Create a `config.yml` file in your Cloudflare configuration directory (usually `~/.cloudflared/`).

### Finding the Credentials File Path
You need the absolute path to the JSON credentials file created in Step 4.

- **Linux/macOS:** `realpath ~/.cloudflared/<tunnel-id>.json`
- **Windows (PowerShell):** `Resolve-Path ~\.cloudflared\<tunnel-id>.json`

### Creating `config.yml`
Open `config.yml` and add the following configuration:

```yaml
tunnel: <TUNNEL_ID>
credentials-file: /absolute/path/to/credentials.json

ingress:
  - hostname: demo.example.com
    service: http://localhost:3000
  - service: http_status:404
```

> [!TIP]
> The `ingress` section maps your public hostname to your local service. The final `service: http_status:404` acts as a catch-all fallback.

## 7. Running the Tunnel

Start the tunnel to make your local service public.

```bash
cloudflared tunnel run <tunnel-name>
```

Your app should now be accessible at `https://demo.example.com`.

---

## Important Considerations

- **Outbound Only**: Cloudflare Tunnel creates a secure outbound connection. You don't need to touch your router's port forwarding.
- **Local Dependency**: Your app must be running locally for the tunnel to work. If your local machine goes offline or the process stops, the domain will return an error.
- **Versatility**: This setup works for any local server (Next.js, Python, PHP, Docker, etc.). Just update the `service` port in `config.yml`.