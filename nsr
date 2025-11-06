#!/usr/bin/env python3
# nsr — DNS records fetcher for nslookup.io
# Usage:
#   nsr rcdeiraq.com                 # default dns server: cloudflare
#   nsr rcdeiraq.com --dns google    # cloudflare|google|quad9|opendns
#   nsr rcdeiraq.com --types A,MX,TXT
#   nsr rcdeiraq.com --raw           # dump API JSON
#   nsr --help

import sys, json, argparse, urllib.request, urllib.error, ssl

API_URL = "https://www.nslookup.io/api/v1/records"
DNS_CHOICES = ["cloudflare", "google", "quad9", "opendns"]

def post_json(url, payload, timeout=15):
    data = json.dumps(payload).encode("utf-8")
    req = urllib.request.Request(url, data=data, headers={"Content-Type": "application/json"})
    ctx = ssl.create_default_context()
    with urllib.request.urlopen(req, context=ctx, timeout=timeout) as resp:
        return json.loads(resp.read().decode("utf-8"))

def colfit(values, minw=4, maxw=60):
    w = max(minw, min(max(len(str(v)) for v in values), maxw))
    return w

def print_table(rows, headers):
    if not rows:
        print("  (no answers)")
        return
    cols = list(zip(*([headers] + rows)))
    widths = [colfit(col) for col in cols]
    fmt = "  " + "  ".join(f"{{:{w}}}" for w in widths)
    print(fmt.format(*headers))
    print(fmt.format(*["-" * w for w in widths]))
    for r in rows:
        print(fmt.format(*[("" if x is None else x) for x in r]))

def pick(v, *keys, default=None):
    for k in keys:
        if isinstance(v, dict) and k in v:
            v = v[k]
        else:
            return default
    return v

def extract_rows(record_type, section):
    """Return list of ordered rows (lists/tuples) per record type."""
    answers = pick(section, "response", "answer", default=[]) or []
    rows = []
    for a in answers:
        rec = a.get("record", {})
        ttl = rec.get("ttl")
        name = rec.get("name")
        target = rec.get("target")
        ip_or_target = rec.get("ip") or target
        if record_type == "A":
            rows.append([name, ttl, ip_or_target])
        elif record_type == "AAAA":
            rows.append([name, ttl, ip_or_target])
        elif record_type == "MX":
            rows.append([name, ttl, rec.get("priority"), target])
        elif record_type == "NS":
            rows.append([name, ttl, target])
        elif record_type == "TXT":
            s = rec.get("strings") or []
            rows.append([name, ttl, " ".join(s)])
        elif record_type == "CNAME":
            rows.append([name, ttl, target])
        elif record_type == "SOA":
            rows.append([
                name, ttl,
                rec.get("host"),     # MNAME
                rec.get("admin"),    # RNAME
                rec.get("serial"),
                rec.get("refresh"),
                rec.get("retry"),
                rec.get("expire"),
                rec.get("minimum"),
            ])
    return rows

def print_section(rt, section):
    q = pick(section, "query", default={})
    resp = pick(section, "response", default={})
    rcode = resp.get("rCode")
    server_ip = pick(q, "server", "ip")
    header = f"[{rt}] rcode={rcode}" + (f" via {server_ip}" if server_ip else "")
    print(header)

    if rcode != "NOERROR":
        print(f"  (query failed: {rcode})")
        return

    rows = extract_rows(rt, section)
    if rt in ("A", "AAAA"):
        print_table(rows, ["Name", "TTL", "Address"])
    elif rt == "MX":
        print_table(rows, ["Name", "TTL", "Priority", "Target"])
    elif rt == "NS":
        print_table(rows, ["Name", "TTL", "Target"])
    elif rt == "TXT":
        print_table(rows, ["Name", "TTL", "Text"])
    elif rt == "CNAME":
        print_table(rows, ["Name", "TTL", "Target"])
    elif rt == "SOA":
        print_table(rows, ["Name", "TTL", "MNAME", "RNAME", "Serial", "Refresh", "Retry", "Expire", "MinTTL"])

def main():
    ap = argparse.ArgumentParser(description="Fetch DNS records from nslookup.io and pretty-print them.")
    ap.add_argument("domain", nargs="?", help="Domain name (e.g., example.com)")
    ap.add_argument("--dns", default="cloudflare", choices=DNS_CHOICES, help="Resolver to query")
    ap.add_argument("--types", default="A,AAAA,MX,NS,TXT,SOA", help="Comma-separated list (e.g. A,MX,TXT)")
    ap.add_argument("--timeout", type=int, default=15, help="HTTP timeout seconds")
    ap.add_argument("--raw", action="store_true", help="Print raw JSON response")
    args = ap.parse_args()

    if not args.domain:
        ap.print_help(sys.stderr); sys.exit(2)

    dom = args.domain.strip().rstrip(".")
    req = {"domain": dom, "dnsServer": args.dns}
    try:
        data = post_json(API_URL, req, timeout=args.timeout)
    except urllib.error.HTTPError as e:
        sys.stderr.write(f"HTTP error {e.code}: {e.read().decode('utf-8', errors='ignore')}\n"); sys.exit(1)
    except Exception as e:
        sys.stderr.write(f"Request failed: {e}\n"); sys.exit(1)

    if args.raw:
        print(json.dumps(data, ensure_ascii=False, indent=2)); return

    print(f"Domain: {data.get('unicodeDomain') or dom}\nDNS Server: {args.dns}\n")

    want = [t.strip().upper() for t in args.types.split(",") if t.strip()]
    keymap = {"A":"a","AAAA":"aaaa","MX":"mx","NS":"ns","TXT":"txt","CNAME":"cname","SOA":"soa"}

    records = data.get("records", {})
    for rt in want:
        section = records.get(keymap.get(rt))
        if not section:
            print(f"[{rt}] (no section returned)\n"); continue
        print_section(rt, section); print()

if __name__ == "__main__":
    main()
