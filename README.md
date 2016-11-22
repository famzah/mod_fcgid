# mod_fcgid
Apache mod_fcgid: Immediate HTTP error 503 if the max total process count is reached

# Motivation
This patch adds a new configuration setting for "mod_fcgid". The source code changes to review follow:
- The whole patch compared to version 2.3.9: https://github.com/famzah/mod_fcgid/compare/2.3.9...maxnowait?diff=split&name=maxnowait
- The whole patch as a single file: https://github.com/famzah/mod_fcgid/compare/2.3.9...maxnowait.patch?diff=unified&name=maxnowait
- Every single commit compared to version 2.3.9: https://github.com/famzah/mod_fcgid/commits/maxnowait

The motivation is that the current behavior to queue up new pending requests differs from the RLimitNPROC directive behavior. When there is a spike in the web hits, lots of Apache children get busy just waiting for up to 30+ seconds to get an idle FastCGI process. Thus we "waste" Apache children doing nothing while they could serve static content. This also puts unneeded memory pressure. Additionally, users today won't wait for a page to load, if it takes more than a few seconds, and we'd rather notify them that we are currently overloaded by sending them a 503 HTTP response immediately.

# FcgidMaxProcessesUsedNoWait
This directive controls the server behavior when the maximum number of active FastCGI application processes FcgidMaxProcesses is reached.

If FcgidMaxProcessesUsedNoWait is set to "Off", then new requests wait in a queue with a timeout to get an idle process slot. Eventually after some waiting those requests either succeed in getting served, or fail with an HTTP 503 Service Unavailable error.

If FcgidMaxProcessesUsedNoWait is set to "On", then an HTTP 503 Service Unavailable error is immediately returned for new requests when all FastCGI application processes are active in servicing other requests.

# Upstream commit
I tried to merge this upstream several times to no avail:
- https://bz.apache.org/bugzilla/show_bug.cgi?id=59656
- http://www.mail-archive.com/dev@httpd.apache.org/msg65748.html
- personal emails to Apache devs
