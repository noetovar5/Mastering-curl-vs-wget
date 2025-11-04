

# <p align="center">Mastering-curl-vs-wget
<img align="right" src="https://visitor-badge.laobi.icu/badge?page_id=noetovar5.Mastering-curl-vs-wgetk"/>
<p align="center"> Mastering-curl-vs-wget

<p align="center">
  <a href="https://www.linkedin.com/posts/noe-tovar-mba_mastering-curl-vs-wgetwhen-why-and-how-as-activity-7388249379880681472-2TSj?utm_source=share&utm_medium=member_desktop&rcm=ACoAAAIcqoIB6U5VUvU6idE-fRlySmTZMPpthTo">
    <img src="https://skillicons.dev/icons?i=linkedin" />
  </a>
</p>
Two real-world scenarios
Scenario 1 — API troubleshooting (my go-to: curl)
I’m on call and a web app is failing to submit orders. I need to reproduce the exact HTTP request the frontend sends—headers, JSON body, auth—and capture the raw response and status code. I also want to try variants (change a header, send a different JSON payload) to isolate a bad upstream. This is where curl shines: it’s a surgical tool for crafting and inspecting requests.

Scenario 2 — Grabbing a docs site for offline review (my go-to: wget)
I’m traveling and need a vendor’s documentation site on my laptop—HTML, images, CSS—so I can read it on a flight. I also want to resume if the connection drops and respect the site’s structure. This is classic wget territory: recursive downloading, mirroring, resuming, and rate-limiting to be a good citizen.

The big picture: purpose, strengths, and trade-offs
What each command is for
curl: A request/response Swiss-army knife. Designed to send arbitrary requests (GET/POST/PUT/PATCH…), set headers, send bodies, handle auth, inspect status codes and response metadata. Supports many protocols (HTTP(S), FTP, SMTP, IMAP, etc.). Ideal for APIs, debugging, and automation.
wget: A retriever and mirrorer. Made to download files and sites, follow links (HTML parsing), recurse directories, resume interrupted downloads, obey robots.txt, and throttle. Ideal for bulk/offline downloading and site mirroring.

Where they overlap
Both can download a single file over HTTP(S), follow redirects, set a user-agent, and resume interrupted transfers.
Both can handle basic auth and proxies.

Where they differ (what I actually care about day-to-day)
Request crafting: curl easily sets headers (-H), custom methods (-X), JSON bodies (-d), multipart uploads (-F), and prints status + timing cleanly. wget can POST but it’s not its sweet spot.
Recursive/site mirroring: wget has first-class recursion (-r), mirroring (-m), link conversion for offline use (-k), and timestamping (-N). curl deliberately does not crawl HTML.
Output control: curl can write headers/body separately, show only status code, or emit machine-parsable variables with -w. wget focuses on files on disk, with progress and logs.
Robustness for long pulls: wget has great resume (-c), retry (--tries, --waitretry), and throttling (--limit-rate)—excellent for big or flaky transfers. curl can throttle and retry too, but the ergonomics for huge site pulls favor wget.
Interactive inspection: For low-level HTTP debugging (-v, --trace-ascii, --http2), curl gives me more granular control and visibility.

Rule of thumb:

If I’m talking to an API or need surgical HTTP control → use curl.
If I’m downloading lots of content or mirroring → use wget.

Quick compare (at a glance)
Best at APIs: curl
Best at mirroring/recursion: wget
Best for custom headers/body: curl
Resume big downloads: both, but wget -c is wonderfully simple
Offline site with fixed links: wget -m -k -p
Show only HTTP status code: curl -s -o /dev/null -w "%{http_code}\n"
Respect robots & throttle by default workflows: wget has the knobs front-and-center

Mini-quiz (pick your answer, then explain “why”)
You need to send a JSON POST with custom headers and inspect the exact HTTP status + response time. a) curl b) wget
You want to mirror https://docs.example.com/ to read offline, convert links for local browsing, and resume if it breaks. a) curl b) wget
You must upload a file via multipart/form-data to a REST endpoint. a) curl b) wget
You need to download a 20-GB file over a spotty connection and automatically retry with backoff. a) curl b) wget (either can work—explain your choice)
You’re debugging a weird redirect chain and want verbose headers for each hop. a) curl b) wget (either can show, but which do you prefer and why?)

Reflection prompt: Which tool is “best”? My answer: neither—pick the right tool for the job. Tell me which you’d choose for your daily workflow and why.

Now its your turn with Hands-on: curl step-by-step
Tip: Replace example.com with your target. For safe practice, you can use https://httpbin.org which echoes requests.
1) Basic GET + follow redirects
curl -L https://example.com 
-L follows 3xx redirects. Great for shortened links or sites forcing HTTPS.

2) Save to a file (use server’s name vs your own)
# Save using remote filename
curl -O https://example.com/file.tar.gz

# Save to a specific name
curl -o app.tar.gz https://example.com/file.tar.gz 
3) Show only the status code (perfect for scripts/health checks)
curl -s -o /dev/null -w "%{http_code}\n" https://example.com/health 
4) Inspect headers and TLS (debugging)
curl -v https://example.com
# or even deeper traces:
curl --trace-ascii trace.log https://example.com 
5) Custom headers and JSON POST
curl -X POST https://api.example.com/orders \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -d '{"sku":"ABC-123","qty":2}' 
6) Multipart file upload
curl -X POST https://api.example.com/upload \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -F 'file=@report.pdf' 
7) Basic auth and a custom user agent
curl -u user:password -A 'MyAgent/1.0' https://example.com/secure 
8) Follow redirects and show headers only (HEAD)
curl -I -L https://example.com/resource 
9) Resume a partial download
curl -C - -O https://example.com/big.iso 
10) Throttle and retry (be kind to servers)
curl --limit-rate 200k --retry 5 --retry-delay 2 -O https://example.com/large.tar 
Hands-on: wget step-by-step
1) Simple download (with useful progress bar)
wget https://example.com/file.tar.gz 
2) Resume an interrupted download
wget -c https://example.com/big.iso 
3) Mirror a site for offline reading (my favorite trio)
wget --mirror --convert-links --page-requisites \
     --adjust-extension --no-parent \
     https://docs.example.com/ 
--mirror = -r -N -l inf --no-remove-listing
--convert-links rewrites links for local browsing
--page-requisites grabs images/CSS/JS required to render pages
--adjust-extension makes local files end with .html where appropriate
--no-parent prevents climbing up to parent dirs

4) Respect bandwidth and avoid hammering servers
wget --limit-rate=300k --wait=1 --random-wait -r https://example.com/data/ 
5) Basic auth and custom headers (less common but possible)
wget --user=user --password=pass \
     --header='X-Env: staging' \
     https://example.com/secure/file.txt 
6) Follow redirects and set a user agent
wget --max-redirect=10 --user-agent='MyAgent/1.0' https://example.com 
7) Download a list of URLs from a file
wget -i urls.txt 
8) “Spider mode” to check link health without downloading files
wget --spider -r -l 3 https://example.com 
9) Timestamping to skip unchanged files (great for nightly jobs)
wget -N https://example.com/daily/report.csv 
10) Proxy support (both tools support this; here’s wget)
https_proxy=https://proxy.local:8443 wget https://example.com/file 
Advanced tips I actually use
Separate headers and body (curl)
Machine-parsable output for scripts (curl)
Log everything (wget)
Be a good citizen Throttle (--limit-rate), wait between requests (--wait, --random-wait), obey robots.txt where appropriate, and avoid scraping sites that forbid it.

Choosing confidently
If I need precision HTTP surgery → curl first.
If I need to bring content home (bulk or whole sites) → wget first.
For one-off file downloads? Either is fine—use the one you remember fastest.
