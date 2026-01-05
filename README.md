# 2026 Hacker News Project

A small PHP page that renders a list of Hacker News stories, while a Python script does the fetching/parsing/filtering “under the hood” and returns JSON to PHP.

## What it does

- PHP (`index.php`) renders the HTML page and calls the Python backend.
- Python (`scripts/hn_fetch.py`) fetches `https://news.ycombinator.com/news` (up to 5 pages), parses stories + metadata, filters, sorts, and prints JSON.
- The UI is styled with `css/styles.css`.

## How it works (high level)

There are two pieces:

- **Frontend (PHP)**: `index.php`
- **Backend (Python)**: `scripts/hn_fetch.py`

### Frontend (PHP)

- **Input**
  - Reads `$_GET['days']` (default `7`, clamped to `1..30`)
  - Reads `$_GET['min_votes']` (default `250`, clamped to `0..5000`)

- **Backend call**
  - Starts a Python process (defaults to `python3`, then falls back to `python`) and runs `scripts/hn_fetch.py`.
  - Reads JSON from stdout and uses it as the story list.

- **Output**
  - Escapes all dynamic values before printing (via `htmlspecialchars`).
  - Adds no-cache headers so refreshes fetch fresh results.
  - Adds a cache-busting query string to the CSS link using `filemtime(css/styles.css)`.

### Backend (Python)

- Fetches `HN_NEWS_URL` pages (`?p=1..5`).
- Parses story title/link from `tr.athing` rows and votes/age from the following metadata row.
- Converts “age … ago” to seconds and filters using `days * 86400`.
- Filters by `min_votes`.
- Sorts by votes (descending).
- Prints a JSON payload like:
  - `generated_at_utc`
  - `days`
  - `min_votes`
  - `stories[]` (each has `title`, `link`, `votes`, `age_text`)

## Requirements

- PHP 8+ (for `proc_open`, and general compatibility)
- Python 3 (available on PATH as `python`, or adjust `$pythonBin` inside `index.php`)
- Python packages (for the backend script): `requests`, `beautifulsoup4`

Install Python deps:

`pip install -r requirements.txt`

### Production note (no virtualenv)

This project does not require a virtual environment. For production, install the Python deps into the *same* Python installation that Apache/PHP will execute.

If Apache can’t find `python` (PATH differences when running as a service), set an environment variable to the full Python path:

- `HN_PYTHON=C:\\Path\\To\\python.exe`

`index.php` will use `HN_PYTHON` if set; otherwise it falls back to `python`.

#### Linux + Apache (generic web host)

Most Linux hosts expose Python as `python3`. The PHP frontend will try `python3` first (then `python`) unless you set `HN_PYTHON`.

Typical install commands (run from your project directory):

`python3 -m pip install -r requirements.txt`

If Apache/PHP uses a different environment than your SSH shell, explicitly set:

- `HN_PYTHON=/usr/bin/python3`

How you set env vars depends on your host (VirtualHost `SetEnv` for mod_php, or PHP-FPM pool `env[]` settings if using php-fpm).

#### Hostinger

Hostinger offers both shared “Web Hosting” plans and VPS. Whether this project works depends on which you have:

- **Hostinger VPS**: Works well. You can install Python packages and control Apache/PHP, so `HN_PYTHON=/usr/bin/python3` and `python3 -m pip install -r requirements.txt` are straightforward.
- **Hostinger Web Hosting (shared)**: This may or may not work. Even if `proc_open()` is enabled, you might not be allowed to install `requests`/`beautifulsoup4` into the Python that PHP can execute.

If your Hostinger plan supports SSH/Terminal access, try:

- `python3 -m pip install --user -r requirements.txt`

If environment variables are hard to set on shared hosting, you can usually set them via `.htaccess` (only if Apache allows it):

- `SetEnv HN_PYTHON /usr/bin/python3`

Troubleshooting:

- If you see `Python backend exited with code 127` and `python: command not found`, it means the web server user can’t find Python on PATH.
  - Fix: set `HN_PYTHON` to the absolute `python3` path (usually `/usr/bin/python3`).

#### Hostinger VPS (recommended)

On a VPS you control the server, so this setup is reliable.

1) Install system packages (example for Ubuntu/Debian):

`sudo apt-get update && sudo apt-get install -y python3 python3-pip`

2) From your project directory, install Python deps (no virtualenv required):

`python3 -m pip install -r requirements.txt`

3) Ensure Apache/PHP uses the right interpreter:

- If you’re using Apache + mod_php, add to your site’s `<VirtualHost>`:

`SetEnv HN_PYTHON "/usr/bin/python3"`

- If you’re using php-fpm, set `HN_PYTHON` in the php-fpm pool (host-specific), then restart php-fpm.

4) Quick backend sanity check:

`/usr/bin/python3 scripts/hn_fetch.py --days 3 --min-votes 250 --max-pages 1`

#### XAMPP (recommended)

In your Apache VirtualHost (XAMPP), set it directly so the running Apache process always has it:

`SetEnv HN_PYTHON "C:/Python314/python.exe"`

Then restart Apache from the XAMPP Control Panel.

## Local URL setup (XAMPP)

Your machine is already running Apache (XAMPP) on port 80, so `http://2026hackernewsproject.localhost/` needs to be served by Apache.

### 1) Add the Apache VirtualHost

1. Open `C:\xampp\apache\conf\extra\httpd-vhosts.conf`.
2. Ensure there is a `<VirtualHost *:80>` entry with `ServerName 2026hackernewsproject.localhost`.
3. Its `DocumentRoot` should point at this project folder: `D:\Websites\2026-FCJamison\portfolio\2026-HackerNewsProject`.
4. Restart Apache in the XAMPP Control Panel.

### 2) Build the site

No build step is required.

Apache serves `index.php` from this folder at:

`http://2026hackernewsproject.localhost/`

Refreshing the page will fetch fresh Hacker News results.

If you ever see Apache showing an old `index.html` instead of `index.php`, remove the old `index.html` from the project root or ensure Apache's `DirectoryIndex` includes `index.php`.

## Example URLs

- Default: `http://2026hackernewsproject.localhost/`
- Last 3 days, 500+ votes: `http://2026hackernewsproject.localhost/?days=3&min_votes=500`
