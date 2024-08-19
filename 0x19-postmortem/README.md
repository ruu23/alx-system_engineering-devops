On the release day of ALX's System Engineering & DevOps project 0x19, at approximately 06:00 West African Time (WAT) in Nigeria, an outage occurred on a standalone Ubuntu 14.04 container hosting an Apache web server. The issue was that GET requests to the server resulted in 500 Internal Server Error responses, instead of serving the expected HTML file for a basic Holberton WordPress site.

Troubleshooting Steps
Our primary debugger, Brennan (who cleverly goes by BDB—catchy, right?), identified the problem around 19:20 PST when tasked with investigating the issue. The following steps were taken to resolve the problem:

Process Check: Brennan verified running processes using ps aux. Two apache2 processes—root and www-data—were confirmed to be active.

Directory Examination: He navigated to the sites-available folder within the /etc/apache2/ directory and confirmed that the web server was serving files from /var/www/html/.

Strace Analysis: In one terminal, he ran strace on the PID of the root Apache process while curling the server in another terminal. Unfortunately, strace provided no useful output.

Strace Retry: Brennan repeated the strace procedure on the PID of the www-data process, this time with lower expectations, but struck gold! The strace output revealed an -1 ENOENT (No such file or directory) error when trying to access /var/www/html/wp-includes/class-wp-locale.phpp.

File Inspection: He searched the /var/www/html/ directory files individually using Vim to locate the erroneous .phpp file extension. The issue was found in the wp-settings.php file (Line 137, require_once(ABSPATH . WPINC . '/class-wp-locale.php');).

Error Fix: Brennan removed the extra trailing p from the line.

Validation: A follow-up curl request to the server returned a successful 200 OK response.

Automation: To prevent future occurrences, a Puppet manifest was written to automate the correction of similar errors.

Summary
The root cause of the outage was a simple typo in the wp-settings.php file, where a trailing p was mistakenly added, resulting in the system failing to locate the correct file (class-wp-locale.php). The typo was corrected, and normal service was restored.

Prevention and Recommendations
This incident was an application-level error, not a web server malfunction. To avoid similar outages in the future, consider the following:

Thorough Testing: Ensure the application is rigorously tested before deployment. This error could have been identified and resolved much earlier if proper testing had been conducted.

Monitoring: Implement an uptime-monitoring service like UptimeRobot to receive immediate alerts if the website goes down.

As a response to this issue, a Puppet manifest 0-strace_is_your_friend.pp was created to automatically correct any instances of phpp file extensions in the /var/www/html/wp-settings.php file. However, we're confident this won't be needed again—because as developers, we never make mistakes, right? :wink:
