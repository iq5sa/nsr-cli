# nsr — DNS Record Lookup CLI (macOS / Linux)

`nsr` is a lightweight command-line tool that queries DNS records using the **nslookup.io** API and prints them in a clear, structured table format.

It requires **no external Python libraries**, works offline except for the query itself, and supports multiple resolver backends (Cloudflare, Google, Quad9, OpenDNS).

---

## ✨ Features

- Query DNS records: **A, AAAA, MX, NS, TXT, CNAME, SOA**
- Select DNS resolver: *cloudflare, google, quad9, opendns*
- Pretty formatted terminal output
- Raw JSON mode (`--raw`) for piping to `jq`
- Cross-platform: macOS & Linux
- No dependencies (pure Python standard library)

---

## 🛠 Installation

### 1. Save the script
```bash
nano nsr
````

Paste the script code into the file.

### 2. Make executable

```bash
chmod +x nsr
```

### 3. Move into PATH

#### macOS (Intel):

```bash
sudo mv nsr /usr/local/bin/nsr
```

#### macOS (Apple Silicon):

```bash
sudo mv nsr /opt/homebrew/bin/nsr
```

#### Linux:

```bash
sudo mv nsr /usr/bin/nsr
```

---

## 🚀 Usage

### Basic Lookup

```bash
nsr example.com
```

### Choose Resolver

```bash
nsr example.com --dns google
```

### Query Specific Record Types

```bash
nsr example.com --types A,MX,TXT
```

### Show Raw JSON Response

```bash
nsr example.com --raw | jq .
```

### Help

```bash
nsr --help
```

---

## 🧾 Output Example

```
Domain: example.com.
DNS Server: cloudflare

[MX] rcode=NOERROR via 1.0.0.1
  Name             TTL   Priority  Target
  ---------------  ----  --------  ----------------
  example.com.     300      10     mx.example.com.
  example.com.     300      20     mx2.example.com.
  example.com.     300      50     mx3.example.com.

[NS] rcode=NOERROR via 1.1.1.1
  Name             TTL     Target
  ---------------  -----   --------------------------
  example.com.    86400   host.com.
  example.com.    86400   host.com.
```

---

## ⚙ Customization

You can easily add more record types or resolvers by modifying:

```python
DNS_CHOICES = ["cloudflare", "google", "quad9", "opendns"]
```

and the `extract_rows()` function for new record structures.

---

## 📄 License

MIT — You are free to use, modify, distribute, and integrate this CLI into other tooling.
