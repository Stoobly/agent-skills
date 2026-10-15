# CA Cert

## Stoobly CA Certificate CLI - Questions & Answers

The CA certificate CLI manages SSL/TLS certificate authority operations for intercepting HTTPS traffic. Installing the CA certificate allows Stoobly to decrypt and inspect HTTPS requests for recording, mocking, and testing.

***

### Understanding CA Certificates

#### Q: What is a CA certificate?

**A:** A CA (Certificate Authority) certificate is a root certificate that allows Stoobly to create valid SSL certificates for intercepting HTTPS traffic. It enables the proxy to decrypt, inspect, and re-encrypt HTTPS requests.

**Example:**

```bash
# Install CA certificate to enable HTTPS interception
stoobly-agent ca-cert install
```

#### Q: Why do I need to install a CA certificate?

**A:** Installing the CA certificate allows Stoobly to intercept HTTPS traffic. Without it, you can only record and mock HTTP (non-encrypted) requests.

**Example:**

```bash
# Without CA cert: Only HTTP works
stoobly-agent record http://api.example.com/users

# With CA cert: Both HTTP and HTTPS work
stoobly-agent ca-cert install
stoobly-agent record https://api.example.com/users  # Now works!
```

#### Q: Is it safe to install the CA certificate?

**A:** Yes, the CA certificate is only installed on your local machine and is only trusted by your system. It's used solely for local development and testing. Remove it when done with `ca-cert uninstall`.

**Example:**

```bash
# Install for testing
stoobly-agent ca-cert install

# When done, uninstall
stoobly-agent ca-cert uninstall
```

***

### Installing CA Certificate

#### Q: How do I install the CA certificate?

**A:** Use `ca-cert install` to install the certificate authority certificate on your system.

**Example:**

```bash
stoobly-agent ca-cert install
```

#### Q: Where does the CA certificate get installed?

**A:** The CA certificate is installed in your system's trusted certificate store. The location varies by operating system:

* **macOS**: Keychain Access
* **Linux**: `/usr/local/share/ca-certificates/` or system store
* **Windows**: Certificate Manager

**Example:**

```bash
# Install CA cert
stoobly-agent ca-cert install

# On macOS, verify in Keychain Access
open -a "Keychain Access"
# Search for "stoobly" or "mitmproxy"

# On Linux, check system certificates
ls /usr/local/share/ca-certificates/
```

#### Q: Do I need sudo/admin privileges to install the CA certificate?

**A:** Yes, installing system certificates typically requires administrative privileges.

**Example:**

```bash
# May prompt for password
stoobly-agent ca-cert install

# On Linux, might need sudo
sudo stoobly-agent ca-cert install
```

#### Q: How do I install the CA certificate to a custom directory?

**A:** Use the `--ca-certs-dir-path` option to specify where the CA certificate files are stored.

**Example:**

```bash
# Install from custom directory
stoobly-agent ca-cert install --ca-certs-dir-path /path/to/custom/ca-certs
```

#### Q: What happens during CA certificate installation?

**A:** The installation process creates the CA certificate (if not exists), adds it to your system's trust store, and configures it to be trusted for SSL/TLS connections.

**Example:**

```bash
stoobly-agent ca-cert install
# Output:
# Generating CA certificate...
# Installing CA certificate to system trust store...
# CA certificate installed successfully!
```

***

### Viewing Certificate Path

#### Q: How do I find the path to the CA certificate file?

**A:** Use `ca-cert show` with the `--format` option to output the path to the certificate file in the specified format.

**Example:**

```bash
# Show path to PEM format certificate
stoobly-agent ca-cert show --format pem
# Output: /home/user/.stoobly/ca_certs/mitmproxy-ca-cert.pem

# Show path to CER format certificate
stoobly-agent ca-cert show --format cer
# Output: /home/user/.stoobly/ca_certs/mitmproxy-ca-cert.cer

# Show path to P12 format certificate
stoobly-agent ca-cert show --format p12
# Output: /home/user/.stoobly/ca_certs/mitmproxy-ca-cert.p12
```

#### Q: What certificate formats are available?

**A:** The `show` command supports three formats: `cer`, `p12`, and `pem`. Each format has different use cases.

**Example:**

```bash
# PEM format (most common, used by many tools)
stoobly-agent ca-cert show --format pem

# CER format (common on Windows/macOS)
stoobly-agent ca-cert show --format cer

# P12 format (PKCS#12, contains private key)
stoobly-agent ca-cert show --format p12
```

#### Q: How do I use the certificate path in scripts?

**A:** The `show` command outputs only the path to stdout, making it ideal for scripting and command substitution.

**Example:**

```bash
# Use in a script to copy certificate
CERT_PATH=$(stoobly-agent ca-cert show --format pem)
cp "$CERT_PATH" /path/to/destination/

# Use with command substitution in Linux installation
sudo cp $(stoobly-agent ca-cert show --format pem) /usr/local/share/ca-certificates/mitmproxy-ca-cert.crt
sudo update-ca-certificates

# Use in environment variable
export CA_CERT_PATH=$(stoobly-agent ca-cert show --format pem)
echo "Certificate at: $CA_CERT_PATH"
```

#### Q: What happens if the certificate doesn't exist?

**A:** The `show` command will output an error message and exit with a non-zero status if the certificate file is not found.

**Example:**

```bash
stoobly-agent ca-cert show --format pem
# Output (if not found):
# Certificate file not found: /path/to/cert.pem
# Run 'stoobly-agent ca-cert install' to generate certificates.

# Check exit status
echo $?
# Output: 1 (error)
```

#### Q: How do I specify a custom CA certs directory?

**A:** Use the `--ca-certs-dir-path` option to search for certificates in a custom directory.

**Example:**

```bash
# Show certificate from custom directory
stoobly-agent ca-cert show --format pem --ca-certs-dir-path /path/to/custom/ca-certs

# Use in script with custom path
CERT_PATH=$(stoobly-agent ca-cert show --format pem --ca-certs-dir-path ~/.custom-stoobly/ca_certs)
```

#### Q: When should I use the show command?

**A:** Use `ca-cert show` when you need the certificate path for manual import (e.g., Firefox), scripting, or when referencing the certificate in other tools.

**Example:**

```bash
# For Firefox manual import
FIREFOX_CERT=$(stoobly-agent ca-cert show --format pem)
# Then import $FIREFOX_CERT in Firefox settings

# For automated deployment scripts
CERT_PATH=$(stoobly-agent ca-cert show --format pem)
scp "$CERT_PATH" server:/tmp/stoobly-ca.pem

# For verification
if stoobly-agent ca-cert show --format pem > /dev/null 2>&1; then
    echo "Certificate exists"
else
    echo "Certificate not found, installing..."
    stoobly-agent ca-cert install
fi
```

***

### Creating SSL Certificates

#### Q: How do I create an SSL certificate for a hostname?

**A:** Use `ca-cert mkcert` with the hostname to generate a signed certificate.

**Example:**

```bash
# Create certificate for a domain
stoobly-agent ca-cert mkcert api.example.com
```

#### Q: How do I create certificates for multiple hostnames?

**A:** Run mkcert for each hostname you need to intercept.

**Example:**

```bash
# Create certificates for multiple domains
stoobly-agent ca-cert mkcert api.example.com
stoobly-agent ca-cert mkcert frontend.example.com
stoobly-agent ca-cert mkcert admin.example.com
```

#### Q: How do I create a wildcard certificate?

**A:** Use wildcard notation with the asterisk (\*) for the subdomain.

**Example:**

```bash
# Wildcard certificate for all subdomains
stoobly-agent ca-cert mkcert "*.example.com"

# Now works for: api.example.com, app.example.com, etc.
```

#### Q: Where are the generated certificates stored?

**A:** Certificates are stored in the certs directory (default: `~/.stoobly/certs/`).

**Example:**

```bash
# Create certificate (stored in default location)
stoobly-agent ca-cert mkcert api.example.com

# Check generated certificates
ls ~/.stoobly/certs/
# api.example.com.pem
# api.example.com-key.pem
```

#### Q: How do I specify a custom output directory for certificates?

**A:** Use the `--certs-dir-path` option to specify where certificates should be saved.

**Example:**

```bash
stoobly-agent ca-cert mkcert api.example.com --certs-dir-path /path/to/output
```

#### Q: How do I specify a custom CA certs directory?

**A:** Use the `--ca-certs-dir-path` option to use a different CA certificate for signing.

**Example:**

```bash
stoobly-agent ca-cert mkcert api.example.com \
  --ca-certs-dir-path /path/to/ca-certs \
  --certs-dir-path /path/to/output
```

***

### Uninstalling CA Certificate

#### Q: How do I uninstall the CA certificate?

**A:** Use `ca-cert uninstall` to remove the certificate from your system (coming soon).

**Example:**

```bash
# Uninstall CA certificate
stoobly-agent ca-cert uninstall
# Output: Not yet implemented. Stay tuned!
```

#### Q: How do I manually remove the CA certificate?

**A:** Manually remove it from your system's certificate store until uninstall is implemented.

**Example:**

```bash
# macOS: Use Keychain Access
# 1. Open Keychain Access
# 2. Search for "stoobly" or "mitmproxy"
# 3. Delete the certificate

# Linux: Remove from certificate directory
sudo rm /usr/local/share/ca-certificates/stoobly-ca.crt
sudo update-ca-certificates

# Windows: Use Certificate Manager (certmgr.msc)
# 1. Open Certificate Manager
# 2. Navigate to Trusted Root Certification Authorities
# 3. Find and delete the Stoobly/mitmproxy certificate
```

***

### Workflow Integration

#### Q: When should I install the CA certificate?

**A:** Install it before recording or testing HTTPS traffic. It's typically done once per machine during initial setup.

**Example:**

```bash
# Initial setup
stoobly-agent ca-cert install

# Now you can record HTTPS
stoobly-agent run --intercept --intercept-mode record
```

#### Q: How do I use CA certificates with scaffold workflows?

**A:** The scaffold workflow automatically prompts for CA certificate installation when needed.

**Example:**

```bash
# Start scaffold workflow
stoobly-agent scaffold workflow up record --app-dir-path ./my-app
# Prompt: Installing CA certificate is required for recording requests, continue? (y/N)

# Or skip prompt with environment variable
export STOOBLY_CA_CERTS_INSTALL_CONFIRM=y
stoobly-agent scaffold workflow up record --app-dir-path ./my-app

# Or using Makefile
make -f .stoobly/services/.Makefile record
```

#### Q: How do I verify the CA certificate is installed?

**A:** Try recording HTTPS traffic. If it works without certificate errors, the CA cert is properly installed.

**Example:**

```bash
# Test HTTPS recording
stoobly-agent run --intercept --intercept-mode record

# In another terminal, try recording HTTPS
stoobly-agent record https://api.example.com/test

# If successful, CA cert is working
```

***

### Troubleshooting

#### Q: What do I do if HTTPS recording fails?

**A:** Ensure the CA certificate is installed and trusted by your system.

**Example:**

```bash
# Reinstall CA certificate
stoobly-agent ca-cert install

# Check if certificate exists
ls ~/.mitmproxy/
# Should see: mitmproxy-ca.pem, mitmproxy-ca-cert.pem

# Try recording again
stoobly-agent record https://api.example.com/users
```

#### Q: What if I get SSL verification errors?

**A:** The CA certificate may not be properly trusted. Reinstall or manually verify in your system's certificate store.

**Example:**

```bash
# Reinstall
stoobly-agent ca-cert install

# On macOS, verify trust settings in Keychain Access
open -a "Keychain Access"
# Find certificate → Get Info → Trust → "Always Trust"

# On Linux, update certificates
sudo update-ca-certificates
```

#### Q: Why does my Node.js process still get certificate errors after installing the CA certificate?

**A:** `ca-cert install` adds the certificate to your system's trust store, which browsers (Chrome, Edge, Safari) and most OS-level tools read from. Node.js doesn't consult that store — it uses its own bundled CA list — so a Node process making HTTPS requests through the Stoobly proxy (a dev server, a script, a non-browser test runner) will still reject Stoobly's certificate even though the browser trusts it fine. Point Node at the certificate explicitly with `NODE_EXTRA_CA_CERTS`.

**Example:**

```bash
# Get the certificate path
CERT_PATH=$(stoobly-agent ca-cert show --format pem)

# Trust it for this Node process
NODE_EXTRA_CA_CERTS="$CERT_PATH" node server.js
```

As a quicker but less safe alternative, `NODE_TLS_REJECT_UNAUTHORIZED=0` disables certificate verification for the whole process — it accepts any certificate, not just Stoobly's, so only use it in local/throwaway dev or test environments, never for anything that also talks to real external services.

```bash
# Less safe — skips all certificate verification, not just Stoobly's
NODE_TLS_REJECT_UNAUTHORIZED=0 node server.js
```

#### Q: How do I handle multiple CA certificates?

**A:** Use different `--ca-certs-dir-path` for different projects or environments.

**Example:**

```bash
# Project A
stoobly-agent ca-cert install --ca-certs-dir-path ~/project-a/.stoobly/ca_certs

# Project B
stoobly-agent ca-cert install --ca-certs-dir-path ~/project-b/.stoobly/ca_certs

# Use with specific project
stoobly-agent run --ca-certs-dir-path ~/project-a/.stoobly/ca_certs
```

#### Q: What do I do if certificate installation fails?

**A:** Check for permission issues, ensure the directory exists, and verify you have admin privileges.

**Example:**

```bash
# Create directory if missing
mkdir -p ~/.stoobly/ca_certs

# Try with sudo (Linux)
sudo stoobly-agent ca-cert install

# Check for errors
stoobly-agent ca-cert install --ca-certs-dir-path ~/.stoobly/ca_certs
```

***

### Browser Configuration

#### Q: Do I need to configure my browser after installing the CA certificate?

**A:** Most browsers use the system certificate store, but some (like Firefox) use their own. You may need to import the certificate manually.

**Example:**

```bash
# For Firefox:
# 1. Go to Settings → Privacy & Security → Certificates → View Certificates
# 2. Click "Import"
# 3. Select: ~/.mitmproxy/mitmproxy-ca-cert.pem
# 4. Trust for identifying websites
```

#### Q: How do I test if my browser trusts the CA certificate?

**A:** Start Stoobly proxy and visit an HTTPS site through it. If no certificate warning appears, it's working.

**Example:**

```bash
# Start proxy
stoobly-agent run --headless

# Configure browser proxy to localhost:8080
# Visit: https://example.com
# No certificate warning = CA cert is trusted
```

***

### Team Collaboration

#### Q: Do team members need to install the CA certificate?

**A:** Yes, each team member needs to install the CA certificate on their machine to intercept HTTPS traffic.

**Example:**

```bash
# Each team member runs:
stoobly-agent ca-cert install

# Now everyone can record/test HTTPS
```

#### Q: Can I share CA certificate files with my team?

**A:** You can share the CA certificate files, but each team member still needs to install them on their system.

**Example:**

```bash
# Share via git (optional - for custom CA)
git add .stoobly/ca_certs/
git commit -m "Add shared CA certificate"

# Team members install
git pull
stoobly-agent ca-cert install --ca-certs-dir-path ./.stoobly/ca_certs
```

#### Q: Should CA certificates be committed to version control?

**A:** Generally no, as they're generated per machine. However, for team consistency, you can share a common CA certificate by committing it.

**Example:**

```bash
# .gitignore (typical setup)
.stoobly/ca_certs/  # Don't commit

# OR for team sharing:
# Remove .stoobly/ca_certs/ from .gitignore
# Commit shared CA certificate
git add .stoobly/ca_certs/
git commit -m "Add shared CA certificate for team"
```

***

### CI/CD Integration

#### Q: How do I handle CA certificates in CI/CD?

**A:** Install the CA certificate in the CI environment before running tests.

**Example:**

```bash
#!/bin/bash
# CI/CD script

# Install CA certificate
stoobly-agent ca-cert install --ca-certs-dir-path ./ci-ca-certs

# Run tests with HTTPS
stoobly-agent run --intercept --intercept-mode test &
AGENT_PID=$!

# Wait for agent to start
sleep 2

# Run test suite
npm test

# Cleanup
kill $AGENT_PID
```

#### Q: How do I use certificates in Docker containers?

**A:** Mount the CA certificate directory and install it in the container.

**Example:**

```dockerfile
# Dockerfile
FROM python:3.14

# Install stoobly-agent
RUN pip install stoobly-agent

# Copy CA certificate
COPY .stoobly/ca_certs /app/.stoobly/ca_certs

# Install CA certificate
RUN stoobly-agent ca-cert install --ca-certs-dir-path /app/.stoobly/ca_certs

# Run your application
CMD ["your-app"]
```

***

### Security Considerations

#### Q: Is the CA certificate secure?

**A:** The CA certificate is only trusted on your local machine. However, anyone with access to the private key could intercept your HTTPS traffic, so protect the certificate files.

**Example:**

```bash
# Protect CA certificate files
chmod 600 ~/.mitmproxy/mitmproxy-ca.pem
chmod 600 ~/.mitmproxy/mitmproxy-ca-cert.pem

# Don't share private keys publicly
echo ".stoobly/ca_certs/" >> .gitignore
```

#### Q: Should I use the same CA certificate in production?

**A:** No! The CA certificate is for local development and testing only. Never use it in production environments.

**Example:**

```bash
# Development only
stoobly-agent ca-cert install

# Production: Use proper SSL certificates from trusted CAs
# (Let's Encrypt, DigiCert, etc.)
```

#### Q: How do I rotate CA certificates?

**A:** Delete old certificates and generate new ones.

**Example:**

```bash
# Remove old CA certificate
rm -rf ~/.mitmproxy/
rm -rf ~/.stoobly/ca_certs/

# Generate and install new one
stoobly-agent ca-cert install

# Regenerate host certificates
stoobly-agent ca-cert mkcert api.example.com
```

***

### Quick Reference

#### Q: What are the most common ca-cert commands?

**A:** Here's a quick reference of frequently used commands:

**Example:**

```bash
# Install CA certificate (required for HTTPS)
stoobly-agent ca-cert install

# Install to custom directory
stoobly-agent ca-cert install --ca-certs-dir-path /path/to/ca-certs

# Show certificate path
stoobly-agent ca-cert show --format pem
stoobly-agent ca-cert show --format cer
stoobly-agent ca-cert show --format p12

# Create certificate for hostname
stoobly-agent ca-cert mkcert api.example.com

# Create wildcard certificate
stoobly-agent ca-cert mkcert "*.example.com"

# Create certificate with custom paths
stoobly-agent ca-cert mkcert api.example.com \
  --ca-certs-dir-path /path/to/ca-certs \
  --certs-dir-path /path/to/output

# Uninstall CA certificate (coming soon)
stoobly-agent ca-cert uninstall
```

***

### Complete Setup Example

#### Q: What's the complete workflow for setting up CA certificates?

**A:** Install CA cert → Create host certificates → Configure proxy → Test HTTPS.

**Example:**

```bash
# Step 1: Install CA certificate
stoobly-agent ca-cert install

# Step 2: Create certificates for your domains (optional, auto-generated by proxy)
stoobly-agent ca-cert mkcert api.example.com
stoobly-agent ca-cert mkcert "*.myapp.local"

# Step 3: Start Stoobly proxy
stoobly-agent run --intercept --intercept-mode record

# Step 4: Configure your app to use proxy (localhost:8080)
export HTTP_PROXY=http://localhost:8080
export HTTPS_PROXY=http://localhost:8080

# Step 5: Make HTTPS requests (they get intercepted)
curl https://api.example.com/users

# Step 6: View recorded requests
stoobly-agent request list
```

***

### Platform-Specific Notes

#### Q: Are there any macOS-specific considerations?

**A:** On macOS, you may need to approve the certificate in Keychain Access and restart your browser.

**Example:**

```bash
# Install on macOS
stoobly-agent ca-cert install

# Open Keychain Access to verify
open -a "Keychain Access"

# Find "mitmproxy" certificate
# Double-click → Trust → "Always Trust"
# Close Keychain Access (it saves automatically)

# Restart browsers for changes to take effect
```

#### Q: Are there any Linux-specific considerations?

**A:** On Linux, you may need to update the certificate store after installation.

**Example:**

```bash
# Install on Linux
sudo stoobly-agent ca-cert install

# Update certificate store
sudo update-ca-certificates

# For Firefox, import manually
# Settings → Privacy & Security → Certificates → View Certificates → Import
# Select: ~/.mitmproxy/mitmproxy-ca-cert.pem
```

#### Q: Are there any Windows-specific considerations?

**A:** On Windows, you may need to run the command prompt as Administrator.

**Example:**

```bash
# Run Command Prompt as Administrator
# Right-click → "Run as administrator"

# Install CA certificate
stoobly-agent ca-cert install

# Certificate is added to Windows Certificate Store
# View: certmgr.msc → Trusted Root Certification Authorities
```
