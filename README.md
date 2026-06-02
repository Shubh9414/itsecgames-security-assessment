1. Description:
The task was to explore the security posture of the publicly hosted endpoint "itsecgames.com". The website is known for known intentional vulnerabilities (bWAPP). Security scans were conducted on the website to find infrastructure vulnerabilities, Misconfigurations and outdated software.

2. Artifacts collected:

- Target URL (Fully sanitized): hxxp[://]itsecgames[.]com/
- IP Address: 31.3.96.40
- Reverse DNS: web.mmebvba.com
- Open ports: 
           - 22/tcp (SSH)
           - 80/tcp (http)
           - 443/tcp (https)
- Infrastructure software banners and outdated software:
           - OpenSSH 6.7p1 (protocol 2.0)
           - Apache httpd
           - X-Generator: Drupal 7
- SSL certificate subject: They had (commonName=mmebv.be)
- HTTP Secure Headers absence: X-Frame-Options, X-XSS-Protection, X-Content-type-Options, Strict-Transport-Security

3. Analysis:
During the analysis of the website there were multiple infrastructure and application flaws that were identified. using Nmap on website revealed that port 22 was open which was running an outdated version of the OpenSSH (6.7p1), which upon exploring is vulnerable to username enumeration (CVE-2018-15473).

Using Curl and Nikto on the website revealed that web application layer was running an end of life framework Drupal 7 which is now deprecated and leaves the server highly exposed to the public remote execution.

The lack of encryption i was seen that the data traffic was being handled through unsecure port 80(HTTP), attempting to connect to the port 443 revealed an invalid SSL certificate the name on the certificate (mmebv.be) does not match the website name (itsecgames.com).

Finally Wapiti web application scan was used which confirmed that the server leaks internal filesystem by calculating "ETag" headers using Linux inode numbers, further it also proved absence of HTTP Secure Headers which leaves users open to browser based attacks such as clickjacking.

4.Mitigation:

- Infrastructure upgrades from OpenSSH 6.7p1 to the latest stable version and Drupal 7 it is advised to immediately upgrade from OpenSSH and plan a full mitigation plan away from legacy Drupal 7 framework.
- Secure transfer of traffic over HTTPS, produce a valid domain matched TLS certificate covering "itsecgames.com" and configuring a permanent redirect from port 80 to secure port 443.
- Enforcing strict HTTP headers:
          - Strict-Transport-Security (HTTPS browser side)
          - X-Frame-Options: Deny (blocking clickjacking)
          - X-Content-type-options: nosniff (Blocking MIME sniffing)
          - Content-Security-policy (restricts malicious script execution)
- Disabling Inode Token Generation, disabling Apache configuration file to use "FileETag MTime Size" instead of default paameters.
          
