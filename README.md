# Ultimate Proxy Server Deployment

This guide provides the steps to deploy a comprehensive proxy and network security suite using Docker Compose. The stack includes Gluetun for VPN connectivity, Nginx Proxy Manager for reverse proxying and SSL management, AdGuard Home for DNS-level ad blocking, and 3x-ui/s-ui for creating censorship-circumvention proxies.

## Prerequisites

*   A server (VPS/Dedicated) with a public IP address.
*   A domain name registered and managed through Cloudflare.

## Step 1: Installation

1.  SSH into your server and install Docker, Docker Compose, and Git with the following command:
    ```bash
    curl -fsSL https://get.docker.com | sudo sh && apt install git -y
    ```
2.  Clone the repository containing the `docker-compose.yml` file.
    ```bash
    # Assuming the file is in a git repository. If not, just create the file manually.
    # git clone <repository_url>
    # cd <repository_directory>
    ```

## Step 2: Cloudflare DNS Configuration

Before deploying, you need to point a domain or subdomain to your server's IP address. This is required for obtaining a valid SSL certificate.

1.  Log in to your Cloudflare dashboard.
2.  Go to the DNS settings for your domain.
3.  Create a new 'A' record with the following details:
    *   **Type:** `A`
    *   **Name:** Your desired subdomain (e.g., `proxy`, `panel`, or `@` for the root domain).
    *   **IPv4 address:** Your server's public IP address (e.g., `123.123.123.123`).
    *   **Proxy status:** Make sure it is set to **DNS only** (gray cloud). This is crucial for the initial SSL certificate validation.

    **Example:** `proxy.domain.com` pointing to `123.123.123.123`.

## Step 3: Docker Compose Configuration

You must edit the `docker-compose.yml` file to include your personal VPN configuration.

1.  Open the `docker-compose.yml` file with a text editor.
2.  Find the `gluetun` service section.
3.  Modify the following environment variables according to your VPN provider's details:
    *   `VPN_SERVICE_PROVIDER`: Change `surfshark` to your VPN provider.
    *   `WIREGUARD_PRIVATE_KEY`: Replace the placeholder key with your actual WireGuard private key.
    *   `SERVER_COUNTRIES`: Change `Netherlands` to your desired server location.

## Step 4: Deployment

Launch all the services using Docker Compose.

1.  Save your changes to the `docker-compose.yml` file.
2.  From the same directory, run the following command:
    ```bash
    docker-compose up -d
    ```

## Step 5: Post-Deployment Setup

### 5.1 Nginx Proxy Manager (NPM) Initial Setup

1.  Access the NPM admin interface in your browser: `http://<your_server_ip>:81`.
2.  Log in with the default credentials:
    *   **Email:** `admin@example.com`
    *   **Password:** `changeme`
3.  Follow the prompts to immediately change the default email and password.

### 5.2 Obtain SSL Certificate via NPM

1.  In the NPM dashboard, navigate to the **SSL Certificates** tab.
2.  Click **Add SSL Certificate** and select **Let's Encrypt**.
3.  In the **Domain Names** field, enter the domain you configured in Cloudflare (e.g., `proxy.domain.com`).
4.  Enable the **Use a DNS Challenge** option if needed, but the standard challenge should work if you configured the DNS record correctly.
5.  Agree to the Let's Encrypt Terms of Service and click **Save**.

NPM will now communicate with Let's Encrypt to issue a certificate. The files for this certificate will be stored in the `npm_letsencrypt` volume. The first certificate you create will be located in a directory named `npm-1`.

### 5.3 Configure TLS in 3x-ui / s-ui Panels

1.  Access your proxy panels directly via IP and port to complete the setup:
    *   **3x-ui:** `http://<your_server_ip>:2053`
    *   **s-ui:** `http://<your_server_ip>:2095`

2.  Navigate to the panel settings area where TLS or security settings are managed.

3.  Input the full path to the certificate and private key files you just generated with NPM. Since the `npm_letsencrypt` volume is mounted into the proxy containers, you can access them directly.

    *   **For 3x-ui (path inside container):**
        *   Certificate Public Key Path: `/root/cert/live/npm-1/fullchain.pem`
        *   Certificate Private Key Path: `/root/cert/live/npm-1/privkey.pem`

    *   **For s-ui (path inside container):**
        *   Certificate Public Key Path: `/app/cert/live/npm-1/fullchain.pem`
        *   Certificate Private Key Path: `/app/cert/live/npm-1/privkey.pem`

4.  Save the settings.

Your proxy panels are now configured with valid TLS encryption.