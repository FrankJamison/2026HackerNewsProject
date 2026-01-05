# Local URL setup (XAMPP)

This project is served by Apache (XAMPP) on port 80 at:

`http://2026hackernewsproject.localhost/`

(These steps are also included in `README.md`.)

## 1) Add the Apache VirtualHost

1. Open `C:\xampp\apache\conf\extra\httpd-vhosts.conf`.
2. Ensure there is a `<VirtualHost *:80>` entry with `ServerName 2026hackernewsproject.localhost`.
3. Its `DocumentRoot` should point at this project folder: `D:\Websites\2026-FCJamison\portfolio\2026-HackerNewsProject`.
4. (Recommended) Set the Python path Apache should use (so it can import `requests`/`bs4`):

	`SetEnv HN_PYTHON "C:/Python314/python.exe"`

4. Restart Apache in the XAMPP Control Panel.

## 2) Build the site

No build step is required.

Apache serves `index.php` from this folder at:

`http://2026hackernewsproject.localhost/`

Refreshing the page will fetch fresh Hacker News results.

If you ever see Apache showing an old `index.html` instead of `index.php`, remove the old `index.html` from the project root or ensure Apache's `DirectoryIndex` includes `index.php`.
