Enuke Software website
The source for www.enukesoftware.com: an SEO-optimised, static website with a PHP contact form, built to run on standard cPanel (Apache + PHP) hosting.
This guide covers putting the code on GitHub and deploying it from GitHub to your server.
> **Important: don’t use GitHub Pages to host this site.**
> GitHub Pages can’t run PHP or read `.htaccess`, so the contact form, the HTTPS/www redirects and the security settings would all stop working. Use GitHub to **store and manage** the code, and deploy it to your own server as described below.
---
Contents
What’s in this repository
Before you start
Put the code on GitHub
Deploy to your server
Clear out the old website
Check everything after going live
Making changes later
Troubleshooting
---
1. What’s in this repository
```
enuke-website/
├── public/                     ← the website itself (only this folder goes live)
│   ├── index.html              Homepage
│   ├── privacy-policy/         Privacy policy page
│   ├── 404.html                “Page not found” page
│   ├── contact.php             Contact form handler (emails sales@enukesoftware.com)
│   ├── .htaccess               HTTPS/www redirects, security, caching (hidden file)
│   ├── assets/page.css         Styles for the inner pages
│   ├── fonts/                  Self-hosted Geist fonts and their licence
│   ├── enuke-logo*.png         Logos
│   ├── favicon.png             Browser tab icon
│   ├── robots.txt              Instructions for search engines
│   └── sitemap.xml             Page list for Google Search Console
├── .cpanel.yml                 Deployment recipe for cPanel (Option A)
├── .github/workflows/deploy.yml  Automatic deployment from GitHub (Option B)
├── .gitignore                  Files Git should never upload
└── README.md                   This guide
```
Files that start with a dot (`.htaccess`, `.cpanel.yml`, `.github`, `.gitignore`) are hidden on Mac and Windows by default. They are essential, so make sure they are uploaded.
Mac: press <kbd>Cmd</kbd> + <kbd>Shift</kbd> + <kbd>.</kbd> in Finder to show hidden files.
Windows: File Explorer → View → tick Hidden items.
---
2. Before you start
You’ll need:
A GitHub account with access to the Enuke organisation (or your own account).
Your cPanel login for enukesoftware.com.
The server running PHP 7.4 or newer (check in cPanel → Select PHP Version or MultiPHP Manager).
The sales@enukesoftware.com mailbox set up (cPanel → Email Accounts), or a forwarder to whoever handles enquiries.
Back up the current website first. In cPanel open File Manager, go to `public_html`, select everything, click Compress, choose Zip Archive, then download the zip to your computer. If anything goes wrong you can restore it.
---
3. Put the code on GitHub
Step 1: Create the repository
Sign in at github.com and click New (or + → New repository).
Repository name: `enuke-website`.
Choose Private. The site is public anyway, but a private repository keeps your work history and configuration to your team.
Don’t tick “Add a README”, “.gitignore” or a licence. This repository already has them.
Click Create repository.
Step 2: Upload the files
Pick whichever method you’re comfortable with.
Method 1: GitHub Desktop (recommended for non-developers)
Install GitHub Desktop and sign in.
File → Add local repository → choose the unzipped `enuke-website` folder.
When it says the folder isn’t a Git repository, click create a repository, keep the name, and click Create repository.
Type a summary such as `Initial website` and click Commit to main.
Click Publish repository, select the GitHub account or organisation, keep Keep this code private ticked, then Publish.
GitHub Desktop uploads hidden files automatically.
Method 2: Command line (for developers)
```bash
cd enuke-website
git init -b main
git add .
git commit -m "Initial website"
git remote add origin https://github.com/YOUR-ORG/enuke-website.git
git push -u origin main
```
Method 3: Upload in the browser
On the new repository page, click uploading an existing file.
With hidden files visible (see section 1), drag everything inside `enuke-website`, including the `public` and `.github` folders and the `.cpanel.yml` and `.gitignore` files, into the page.
Click Commit changes.
Browser uploads sometimes skip hidden files, so check the result: open `public` on GitHub and confirm `.htaccess` is listed, and confirm `.cpanel.yml` and `.github` appear at the top level. If one is missing, click Add file → Create new file, type its exact name and path (for example `public/.htaccess`), paste in its contents from your computer, and commit.
---
4. Deploy to your server
Choose one option.
	Option A: cPanel Git Version Control	Option B: automatic deploy with GitHub Actions
How it works	cPanel pulls the code from GitHub when you click a button	Every change pushed to GitHub goes live automatically
Best for	Simple, controlled updates	Teams who update the site often
Needs	A cPanel host with Git Version Control	FTP access to your server
Option A: cPanel Git Version Control
1. Let your server read the repository. For a private repository, the server needs a key to access it.
In cPanel open Terminal (or SSH Access → Manage SSH Keys) and create a key:
```bash
   ssh-keygen -t ed25519 -C "cpanel-deploy" -f ~/.ssh/github_deploy -N ""
   cat ~/.ssh/github_deploy.pub
   ```
Copy the line that starts with `ssh-ed25519`.
On GitHub, open the repository → Settings → Deploy keys → Add deploy key. Name it `cPanel server`, paste the key, leave Allow write access unticked, and click Add key.
Back in the cPanel Terminal, tell the server to use this key for GitHub:
```bash
   cat >> ~/.ssh/config <<'EOF'
   Host github.com
     IdentityFile ~/.ssh/github_deploy
     IdentitiesOnly yes
   EOF
   chmod 600 ~/.ssh/config
   ssh -T git@github.com
   ```
Type `yes` if asked. A message saying you’ve “successfully authenticated” means it worked.
If your repository is public, you can skip this whole step and use the HTTPS address in the next step instead.
2. Connect cPanel to the repository.
In cPanel open Git™ Version Control → Create.
Turn on Clone a Repository.
Clone URL: `git@github.com:YOUR-ORG/enuke-website.git` (find it on GitHub under the green Code button → SSH).
Repository Path: `repositories/enuke-website`. Keep it outside `public_html`.
Repository Name: `enuke-website` → click Create.
3. Deploy.
Do the clean-up in section 5 first if an old site is still in `public_html`.
In Git Version Control, click Manage next to `enuke-website` → Pull or Deploy tab.
Click Update from Remote to fetch the latest code, then Deploy HEAD Commit.
cPanel follows the recipe in `.cpanel.yml`: it copies the contents of `public/` (including `.htaccess`) into `public_html`.
Option B: automatic deploy with GitHub Actions
In cPanel open FTP Accounts and note the FTP server (usually `ftp.enukesoftware.com`) and a username and password. You can use your main cPanel account or create a dedicated one.
On GitHub, open the repository → Settings → Secrets and variables → Actions → New repository secret, and add three secrets:
Name	Value
`FTP_SERVER`	e.g. `ftp.enukesoftware.com`
`FTP_USERNAME`	the FTP username
`FTP_PASSWORD`	the FTP password
Do the clean-up in section 5 if an old site is still in `public_html`.
Open the Actions tab → Deploy website → Run workflow. Watch it finish with a green tick.
From now on, every change pushed to the `main` branch is uploaded automatically. Until the secrets are added, the workflow runs harmlessly and deploys nothing.
Note on the upload folder: the workflow uploads to `public_html/` inside the FTP account’s home folder. If your FTP account opens directly inside `public_html` (common for extra FTP accounts), edit `.github/workflows/deploy.yml` and change `server-dir: ./public_html/` to `server-dir: ./`.
---
5. Clear out the old website
Deployment adds and overwrites files, but it doesn’t delete old ones. Leftover files from the previous site would stay reachable and could confuse search engines.
After taking the backup from section 2:
In cPanel File Manager, open `public_html`.
Create a folder outside `public_html` (for example in your home folder) called `old-site-backup`.
Move the old website’s files there. Keep anything your host or other services rely on, such as `.well-known` (used for SSL certificates) and the `cgi-bin` folder.
Deploy (Option A or B).
If the old site was WordPress, also make sure no old `index.php` remains in `public_html`.
---
6. Check everything after going live
Check	What should happen
Visit `http://enukesoftware.com`	Redirects to `https://www.enukesoftware.com/`
Homepage	Loads with the Enuke logo and correct fonts
`https://www.enukesoftware.com/privacy-policy/`	Privacy policy loads
`https://www.enukesoftware.com/any-made-up-page`	Branded “page not found” page
Submit the contact form	“Thanks, we’ve received your enquiry…”, and an email arrives at sales@enukesoftware.com (check spam too)
Phone view	The menu button opens the navigation; nothing scrolls sideways
Then:
Google Search Console: add `https://www.enukesoftware.com`, verify it, and submit `https://www.enukesoftware.com/sitemap.xml` under Sitemaps.
After a few days of HTTPS working correctly: in `public/.htaccess`, remove the `#` at the start of the `Strict-Transport-Security` line, commit, and deploy. This tells browsers to always use HTTPS.
---
7. Making changes later
Edit files in the `public/` folder. You can do this on your computer and commit with GitHub Desktop, or directly on GitHub: open a file, click the pencil icon, then Commit changes.
Deploy:
Option A: cPanel → Git Version Control → Manage → Update from Remote → Deploy HEAD Commit.
Option B: happens automatically within a minute or two of the commit.
Never edit files directly in `public_html`. The next deployment would overwrite those changes. Always change the code in GitHub.
Common settings
To change	Edit
Where enquiries are sent	`public/contact.php` → `TO_EMAIL`
Redirects from old page addresses	`public/.htaccess` → section 2
Page title and description shown in Google	`<title>` and `<meta name="description">` in `public/index.html`
New pages	Add them to `public/sitemap.xml` too
---
8. Troubleshooting
Every page shows “Too many redirects”
Your host or Cloudflare handles HTTPS in front of the server. In `public/.htaccess`, replace the line `RewriteCond %{HTTPS} off [OR]` with `RewriteCond %{HTTP:X-Forwarded-Proto} !https [OR]`, then redeploy.
“500 Internal Server Error” on every page
Your server is missing an Apache module that `.htaccess` uses. Ask your host to enable `mod_rewrite` and `mod_headers`.
The contact form says it couldn’t send
“Submissions are only accepted from enukesoftware.com”: you’re testing on a different address. The form only works on the live domain.
“A problem on our side”: the server’s mail function failed. Ask your host to confirm PHP `mail()` is enabled.
The form says “Thanks” but no email arrives
Check spam first. Then ask your host to set up SPF and DKIM records for enukesoftware.com (cPanel → Email Deliverability shows their status). If your email is hosted with Google Workspace or Microsoft 365, ask a developer to switch `contact.php` to send through your mailbox’s SMTP login.
cPanel deployment fails or does nothing
Make sure `.cpanel.yml` is at the top level of the repository and that cPanel shows a clean working tree. If the error mentions the path, edit `.cpanel.yml` and replace `$HOME` with your full home directory, e.g. `/home/enukeuser`, which is shown in cPanel under General Information.
GitHub Actions deploy fails
Open the failed run in the Actions tab. Login errors mean the FTP secrets are wrong. “Can’t connect” errors usually mean your host doesn’t support FTPS: change `protocol: ftps` to `protocol: ftp` in `deploy.yml`, or ask your host to enable FTPS.
The site looks unchanged after deploying
Your browser is showing a stored copy. Press <kbd>Ctrl</kbd> + <kbd>F5</kbd> on Windows or <kbd>Cmd</kbd> + <kbd>Shift</kbd> + <kbd>R</kbd> on Mac.
---
Contact: sales@enukesoftware.com | +91 99584 90489
© Enuke Software Private Limited
