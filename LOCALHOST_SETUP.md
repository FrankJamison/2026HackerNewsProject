# Local URL setup (XAMPP)

This project is served by Apache (XAMPP) on port 80 at:

`http://2026hackernewsproject.localhost/`

Note: the `.localhost` TLD resolves to your own machine (127.0.0.1) on modern OSes/browsers, so you typically do **not** need to edit your hosts file for this.

(These steps are also included in `README.md`.)

## 1) Add the Apache VirtualHost

1. Open `C:\xampp\apache\conf\extra\httpd-vhosts.conf`.
2. Ensure there is a `<VirtualHost *:80>` entry with `ServerName 2026hackernewsproject.localhost`.
3. Its `DocumentRoot` should point at this project folder: `D:\Websites\2026-FCJamison\portfolio\2026HackerNewsProject`.
4. (Recommended) Set the Python path Apache should use (so it can import `requests`/`bs4`):

	`SetEnv HN_PYTHON "C:/Python314/python.exe"`

4. Restart Apache in the XAMPP Control Panel.

## 2) Build the site

No build step is required.

Apache serves `index.php` from this folder at:

`http://2026hackernewsproject.localhost/`

Refreshing the page will fetch fresh Hacker News results.

If you ever see Apache showing an old `index.html` instead of `index.php`, remove the old `index.html` from the project root or ensure Apache's `DirectoryIndex` includes `index.php`.

---

# Production subdomain setup (Apache)

In production you generally do **not** use `.localhost`. Keep the same “subdomain-style” URL by using a real domain you control, for example:

`https://2026hackernewsproject.yourdomain.com/`

Minimum requirements:

1) DNS: create an `A`/`AAAA` record for your subdomain pointing at the server.
2) Apache: create a VirtualHost with `ServerName 2026hackernewsproject.yourdomain.com` and `DocumentRoot` pointing at this project folder.
3) Python: ensure Apache/PHP can run Python and import `requests` + `beautifulsoup4`.

Recommended Apache env var (so PHP executes the correct interpreter):

- Linux example: `SetEnv HN_PYTHON "/usr/bin/python3"`
- Windows example: `SetEnv HN_PYTHON "C:/Path/To/python.exe"`

Then restart Apache.
