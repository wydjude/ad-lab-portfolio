New domain, just had default settings. No minimum password strength enforced, no login banner set up.
Checked the default domain policy. No real password requirements enforced, so users could set something as weak as '1234' or '0000'. No complexity, no minimum length, no filtering of common passwords either.
Opened Group Policy Management from Server Manager, Tools. Created a new GPO and linked it to the Corp OU. Set the password policy: minimum length 12 characters, complexity enabled. Under Security Options, set the login banner title and message text.
