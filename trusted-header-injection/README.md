# Access-Control Bypass Header Wordlists

Categorized, deduplicated HTTP header wordlists for testing whether an
application trusts spoofable client-supplied headers for IP allowlisting,
virtual-host routing, internal path rewriting, or scheme/port decisions.

## Files

| File | Purpose |
|---|---|
| `01-ip-headers.txt` | Headers apps sometimes trust to identify the "real" client IP (X-Forwarded-For, X-Real-IP, True-Client-IP, etc.) |
| `02-host-headers.txt` | Headers used to override the Host the backend thinks it's serving |
| `03-path-url-headers.txt` | Headers that carry a path or full URL, used by some rewrite modules / WAFs (X-Original-URL, X-Rewrite-URL, etc.) |
| `04-scheme-port-method-headers.txt` | Headers signaling protocol/port/method, used behind TLS-terminating proxies |
| `05-ip-values-to-test.txt` | Common bypass values (localhost variants, decimal/hex/octal encodings of 127.0.0.1, RFC1918 ranges) to pair with the IP headers |
| `00-all-headers-combined.txt` | All 69 headers merged, sorted, deduped — for a single Intruder payload set or `-H` fuzzing loop |

## How to use

**Burp Intruder:** load `01-ip-headers.txt` (or the combined file) as the
payload set on the header *name* position, sniper or pitchfork with
`05-ip-values-to-test.txt` on the value position.

**ffuf:**
```
ffuf -u https://target/admin -H "FUZZ: 127.0.0.1" -w 01-ip-headers.txt
```

**Manual/curl loop:**
```
while read h; do
  curl -s -o /dev/null -w "%{http_code} $h\n" -H "$h: 127.0.0.1" https://target/admin
done < 01-ip-headers.txt
```

## Sources consulted (Nov 2026)

- PayloadsAllTheThings — Reverse Proxy Misconfigurations
  (swisskyrepo.github.io/PayloadsAllTheThings)
- Vidoc Security Lab — "Ultimate 401 and 403 bypass methods"
  (blog.vidocsecurity.com)
- Trickest — "403 Bypass: Header, Path, and Method Techniques"
- shubs.io — "Enumerating IPs in X-Forwarded-Headers to bypass 403 restrictions"
- `slicingmelon/gobypass403` (Go WAF/403 bypass tool, headers_ip module)
- `Exript/ipspoof` (PyPI — 100+ client-IP header discovery tool)
- `Tagoletta/AccessContextFuzzer` (Burp extension — categorized IP/Host bypass tables)
- `Subhashis360/bypass-403` (bypass-403.py header/value pairs)
- `aw-junaid/bug-bounty` methodology doc — header-injection.md
- Public CVE/advisory write-ups confirming real-world exploitation
  (calibre X-Forwarded-For ban bypass, rustfs SourceIp bypass,
  WordPress DoLogin plugin CVE-2023-4631)

## Notes

- No single public repo kept these four categories as clean separate files —
  this list was assembled and deduplicated from the above by category.
- Values with no public exploitation evidence (pure typo variants like
  `Refferer`, `Referrer`) are kept because real codebases have been found
  checking them by mistake — low cost to include, occasionally hits.
- `Via` and `X-Forwarded-By` are borderline (they name the proxy, not the
  client) but are included in IP headers since some apps misuse them for
  IP logic.
