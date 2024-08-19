Postmortem
On the launch day of ALX's System Engineering & DevOps project 0x19, at around 06:00 West African Time (WAT) in Nigeria, a significant outage occurred on an isolated Ubuntu 14.04 container running an Apache web server. The problem manifested as 500 Internal Server Error responses when GET requests were made to the server, instead of the expected HTML output for a simple Holberton WordPress site.

Troubleshooting Steps
Our lead troubleshooter, Brennan (affectionately known as BDB—because catchy initials are important), detected the issue around 19:20 PST. The following steps were taken to identify and resolve the problem:

Process Verification: Brennan initiated the investigation by running ps aux to verify active processes. Two apache2 processes were running correctly—one under root and the other under www-data.

Directory Check: He then examined the contents of the sites-available folder within the /etc/apache2/ directory and confirmed that the server was serving files from the /var/www/html/ directory.

Strace Analysis: Brennan utilized strace on the PID of the root Apache process while making curl requests to the server. However, this initial attempt yielded no useful information.

Strace Retry: Undeterred, Brennan ran strace on the www-data process. This time, he struck gold! The output showed an -1 ENOENT (No such file or directory) error when the server attempted to access /var/www/html/wp-includes/class-wp-locale.phpp.

File Examination: Brennan meticulously combed through the files in the /var/www/html/ directory using Vim, searching for the incorrect .phpp extension. The error was located in the wp-settings.php file at Line 137 (require_once(ABSPATH . WPINC . '/class-wp-locale.php');).

Error Correction: He promptly removed the extraneous p from the file extension.

Validation: Brennan tested the fix by making another curl request to the server, which successfully returned a 200 OK response.

Automation: To prevent similar issues in the future, Brennan wrote a Puppet manifest to automate the correction of this type of error.

Summary
The root cause of the outage was a simple typo in the wp-settings.php file—a stray p at the end of the filename class-wp-locale.phpp. This typo prevented the system from locating the correct file, causing the application to crash. The error was corrected, and normal functionality was restored.

Prevention and Recommendations
This incident was due to an application error, not a web server malfunction. To avoid similar issues in the future, the following steps are recommended:

Comprehensive Testing: Always perform thorough testing of the application before deployment. This issue could have been detected and resolved earlier if proper testing had been conducted.

Monitoring Setup: Implement an uptime-monitoring service such as UptimeRobot to receive immediate alerts if the website experiences downtime.

In response to this incident, a Puppet manifest 0-strace_is_your_friend.pp was created to automatically correct any occurrences of phpp extensions in the /var/www/html/wp-settings.php file. However, we're confident this won't be needed again—because as developers, we never make mistakes, right? 😉

