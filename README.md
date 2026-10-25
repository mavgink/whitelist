# whitelist

A personal DNS allowlist for [Technitium DNS Server](https://technitium.com/dns/).
It lists the domains that popular services (Google, Microsoft, Apple, Plex,
streaming, gaming and so on) need in order to work, so they keep working when
block lists are turned on. New domains get added as I find services that break.

## Files

| File | What it is |
| --- | --- |
| [`sorted-whitelist.txt`](sorted-whitelist.txt) | **The main list.** Domains grouped by service under `# ===` section headers. Use this one. |
| [`whitelist.txt`](whitelist.txt) | The older, flat list in plain alphabetical order. Kept for reference. |

Both files use the plain domain-list format Technitium understands: one domain
per line. Lines starting with `#` are comments.

## Using it in Technitium

Technitium treats an allowed domain as covering all of its subdomains. An entry
for `plex.tv` also allows `app.plex.tv`, `clients.plex.tv` and so on, so the
list only needs the parent domain.

### Option 1: subscribe to the list URL (recommended)

Technitium downloads the list and refreshes it automatically.

1. Open the Technitium web console and go to **Settings > Blocking**.
2. Make sure **Enable Blocking** is on.
3. Add this URL to **Allow List URLs**:

   ```
   https://raw.githubusercontent.com/mavgink/whitelist/main/sorted-whitelist.txt
   ```

4. Pick a **Block List URL Update Interval** (the allow list refreshes on the
   same schedule) and click **Save Settings**.

### Option 2: import it once

1. Download [`sorted-whitelist.txt`](https://raw.githubusercontent.com/mavgink/whitelist/main/sorted-whitelist.txt).
2. In the web console go to **Allowed > Import** and paste or upload the
   domains.

Imported domains don't update by themselves, so you'll need to re-import after
the list changes.

## Adding a domain

1. Find the domain that's being blocked, for example from Technitium's
   **Logs** or **Dashboard > Top Blocked Domains**.
2. Add it to `sorted-whitelist.txt` under the section for that service. If no
   section fits, add a new one with the same header style:

   ```
   # ==========================
   # SERVICE NAME
   # ==========================
   example.com
   ```

3. Keep the list tidy:
   - **Prefer the parent domain.** Add `example.com` instead of
     `cdn.example.com` when it's safe to allow the whole domain.
   - **Skip covered subdomains.** Don't add `a.example.com` if `example.com`
     is already listed, because the parent already allows it.
   - **No duplicates.** Search the file before adding.
   - Use lowercase and no trailing dot or `http://`.

You can check for duplicates and covered subdomains with:

```sh
# exact duplicates
grep -v '^\s*\(#\|$\)' sorted-whitelist.txt | sort | uniq -d

# entries already covered by a listed parent domain
grep -v '^\s*\(#\|$\)' sorted-whitelist.txt | sort -u > /tmp/domains
while read -r d; do
  p=$d
  while [ "${p#*.}" != "$p" ]; do
    p=${p#*.}
    grep -qx "$p" /tmp/domains && echo "$d (covered by $p)" && break
  done
done < /tmp/domains
```

Both commands print nothing when the list is clean.
