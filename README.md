# WordPress Site Health Troubleshooting Guide

A practical guide to diagnosing and resolving WordPress Site Health warnings, loopback request failures, PHP compatibility issues, HTTPS problems, scheduled task warnings, and server configuration issues.

## Table of Contents

- [Overview](#overview)
- [Understanding Site Health Results](#understanding-site-health-results)
- [1. Access the Site Health Screen](#1-access-the-site-health-screen)
- [2. Troubleshoot Loopback Requests](#2-troubleshoot-loopback-requests)
- [3. Diagnose Scheduled Task Warnings](#3-diagnose-scheduled-task-warnings)
- [4. Check PHP Compatibility](#4-check-php-compatibility)
- [5. Troubleshoot HTTPS and SSL Warnings](#5-troubleshoot-https-and-ssl-warnings)
- [6. Investigate REST API and HTTP Request Issues](#6-investigate-rest-api-and-http-request-issues)
- [7. Check Filesystem Permissions](#7-check-filesystem-permissions)
- [8. Review Caching and Performance](#8-review-caching-and-performance)
- [9. Diagnose WordPress.org Connectivity](#9-diagnose-wordpressorg-connectivity)
- [10. Export Site Health Information](#10-export-site-health-information)
- [Site Health Audit Checklist](#site-health-audit-checklist)
- [Security Best Practices](#security-best-practices)
- [Official References](#official-references)

## Overview

WordPress Site Health helps administrators identify potential configuration, security, and performance issues.

The feature is available under:

**WordPress Dashboard → Tools → Site Health**

It provides two primary views:

- **Status:** Displays critical issues, recommended improvements, and passed tests.
- **Info:** Displays technical details about WordPress, the active theme, plugins, PHP, the database, media handling, and filesystem permissions.

A warning does not always mean that the website is broken. Investigate the specific finding and determine whether it affects the site's functionality, security, or performance.

Official documentation: https://wordpress.org/documentation/article/site-health-screen/

## Understanding Site Health Results

| Result | Meaning | Recommended action |
|---|---|---|
| Critical issue | A potentially serious configuration, security, or performance concern | Investigate promptly |
| Recommended improvement | A configuration or maintenance improvement may be beneficial | Evaluate and plan a fix |
| Passed test | The tested condition passed | No action usually required |
| Informational detail | Technical information about the environment | Review when diagnosing an issue |

Do not attempt to eliminate every warning without understanding its purpose. Some checks depend on the site's hosting architecture and operational requirements.

## 1. Access the Site Health Screen

1. Sign in to WordPress with an administrator account.
2. Open **Tools → Site Health**.
3. Wait for the status checks to complete.
4. Record the exact warning text.
5. Open the **Info** tab for relevant technical details.
6. Check the affected feature before making changes.

For example, if Site Health reports a loopback request failure, test whether scheduled tasks or other site functions are affected before changing firewall rules.

### Initial diagnostic questions

- When did the warning first appear?
- Did it start after a plugin, theme, PHP, or hosting change?
- Does the website still function normally?
- Are other websites on the same server affected?
- Is there a corresponding error in the PHP or web server logs?

## 2. Troubleshoot Loopback Requests

WordPress uses loopback requests to make HTTP requests back to its own site. These requests support certain background tasks and code checks.

### Common symptoms

- Site Health reports that a loopback request failed.
- Background operations do not complete as expected.
- Plugin or theme code checks fail.
- A local HTTP request returns an error or times out.

### Possible causes

- Incorrect site URL configuration
- DNS resolution problems
- SSL certificate or HTTPS configuration errors
- Firewall or security rules blocking the request
- PHP execution problems
- Web server or proxy restrictions
- Plugin conflicts

### Troubleshooting procedure

1. Confirm the site's WordPress Address and Site Address under **Settings → General**.
2. Verify that the site's HTTPS URL loads without certificate errors.
3. Check the PHP and web server logs around the time of the failure.
4. Review firewall or security logs for blocked loopback requests.
5. Check whether the server can resolve the site's hostname correctly.
6. Test on staging if a plugin or theme conflict is suspected.
7. Retest Site Health after correcting the confirmed cause.

Do not disable the firewall or SSL verification as a general workaround. Make only the narrowly scoped change required to resolve the issue.

## 3. Diagnose Scheduled Task Warnings

WordPress uses WP-Cron to trigger scheduled work, including certain publishing, maintenance, and update tasks.

A Site Health warning about scheduled events may indicate that scheduled work is not running as expected.

### Check scheduled events

If WP-CLI is available and you have authorized access to the correct installation, run:

```bash
wp cron event list
```

Run the command from the WordPress directory or specify the correct installation path:

```bash
wp --path=/path/to/wordpress cron event list
```

Replace `/path/to/wordpress` with the actual path.

### Troubleshooting steps

1. Confirm that the site can handle loopback requests.
2. Review the scheduled event list for overdue or repeatedly failing tasks.
3. Check whether `DISABLE_WP_CRON` is configured.
4. If a system cron job is intended to run WordPress scheduled events, verify that it is configured and executing correctly.
5. Review PHP logs for failed scheduled callbacks.
6. Check whether a plugin's scheduled task is repeatedly generating errors.

If `DISABLE_WP_CRON` is enabled, determine whether an external scheduler is configured before changing it.

Do not add duplicate cron jobs or run destructive maintenance commands just to clear a warning.

Reference: https://developer.wordpress.org/plugins/cron/

## 4. Check PHP Compatibility

An outdated PHP version, missing extension, or incompatible plugin can trigger Site Health warnings or cause runtime errors.

### Review the active environment

Open **Tools → Site Health → Info → Server** and record:

- PHP version
- PHP SAPI and server information
- Memory limit
- Maximum execution time
- Maximum upload size
- Relevant PHP extensions

The PHP version reported by the command line may differ from the version used by the website. Check the PHP version assigned to the domain in the hosting control panel.

### cPanel checks

1. Open **MultiPHP Manager**, if available.
2. Identify the PHP version assigned to the domain.
3. Check the PHP extensions available for that version.
4. Review the website's error logs.
5. Test compatibility on staging before changing PHP versions.

Use a PHP version supported by WordPress and compatible with the site's themes and plugins.

Do not switch production PHP versions without a backup and compatibility plan.

Reference: https://wordpress.org/about/requirements/

## 5. Troubleshoot HTTPS and SSL Warnings

Site Health may report HTTPS or certificate-related problems when the site URL, server configuration, or certificate chain is incorrect.

### Verification steps

1. Open the site's HTTPS URL in a browser.
2. Check whether the certificate is valid for the requested hostname.
3. Confirm that the certificate has not expired.
4. Review WordPress Address and Site Address under **Settings → General**.
5. Check the web server or hosting control panel's SSL status.
6. Review reverse-proxy or CDN settings if HTTPS terminates outside the origin server.
7. Check for mixed-content warnings if the site loads but some resources use HTTP.

### Common causes

- Expired or incorrectly issued certificate
- Missing intermediate certificate
- Incorrect domain or hostname
- Reverse-proxy HTTPS detection problems
- Inconsistent HTTP and HTTPS URLs
- Redirect loops between the origin and proxy

Do not disable certificate verification to suppress a warning. Correct the certificate or proxy configuration instead.

## 6. Investigate REST API and HTTP Request Issues

Some Site Health checks depend on WordPress HTTP requests and REST API functionality.

### Symptoms

- A REST API check fails.
- Loopback requests return HTTP errors.
- Requests to external services time out.
- Site Health reports that WordPress.org cannot be reached.

### Diagnostic steps

1. Identify the exact failed URL or request reported by the diagnostic.
2. Record the HTTP status code and response.
3. Check PHP and web server logs.
4. Review DNS resolution and outbound network restrictions.
5. Check whether a security plugin or firewall is blocking the request.
6. Test suspected plugin conflicts on staging.
7. Re-run Site Health after fixing the underlying issue.

A response such as HTTP 401, 403, 404, or 500 has a different meaning. Diagnose the specific response rather than applying a generic fix.

If `WP_HTTP_BLOCK_EXTERNAL` is configured, verify whether the restriction is intentional and whether the required hosts are allowed.

Reference: https://developer.wordpress.org/rest-api/

## 7. Check Filesystem Permissions

Site Health can report whether WordPress can write to directories needed for updates, uploads, themes, and plugins.

### Common directories

```text
wp-content/
wp-content/uploads/
wp-content/plugins/
wp-content/themes/
```

The exact write requirements depend on the operation and hosting configuration.

### Troubleshooting steps

1. Open the filesystem permissions section in Site Health Info.
2. Identify the directory reported as unwritable.
3. Verify directory ownership and permissions.
4. Confirm that the filesystem is mounted read-write.
5. Check available disk space and inode usage.
6. Review PHP or server logs for permission errors.
7. Contact the hosting provider if ownership or server-level permissions cannot be changed from your account.

Common starting permissions are `755` for directories and `644` for files, but the correct settings depend on the server's execution model.

Avoid using `777` permissions or recursively changing ownership without verifying the correct account and group.

## 8. Review Caching and Performance

Caching can improve performance, but incorrect configuration may interfere with dynamic requests or background operations.

### Checks

- Review page cache configuration.
- Check persistent object caching status.
- Confirm that dynamic admin and API requests are not incorrectly cached.
- Inspect cache-related plugin errors.
- Check PHP-FPM resource limits and server load when requests fail intermittently.
- Review application logs before disabling caching.

If a cache appears responsible, test the suspected configuration change on staging and verify the result.

Do not assume that every Site Health performance recommendation requires a new caching plugin or a server upgrade.

## 9. Diagnose WordPress.org Connectivity

Site Health may report that the website cannot reach WordPress.org. This can interfere with checking for updates and retrieving certain resources.

### Possible causes

- DNS resolution failures
- Outbound firewall restrictions
- Proxy configuration errors
- TLS or certificate problems
- Temporary remote service issues
- Incorrect HTTP request configuration

### Troubleshooting steps

1. Check whether the issue affects WordPress.org only or other outbound requests as well.
2. Verify that the server can resolve external hostnames.
3. Review outbound firewall and proxy rules.
4. Inspect PHP logs for cURL or TLS errors.
5. Check whether external HTTP requests are intentionally restricted.
6. Retest after the confirmed issue has been corrected.

If you do not have root access, ask your hosting provider to investigate outbound connectivity and server-level restrictions.

Do not broadly disable network security controls to restore update checks.

## 10. Export Site Health Information

The Info tab can help you collect technical details when reporting a problem.

### How to collect information

1. Open **Tools → Site Health → Info**.
2. Review the relevant sections.
3. Use the available copy-to-clipboard or export option.
4. Remove credentials, private URLs, personal information, and other sensitive details before sharing.
5. Provide the sanitized information to your developer or hosting provider.

Useful sections may include:

- WordPress
- Active and inactive plugins
- Active theme
- Server
- Database
- Media handling
- WordPress constants
- Filesystem permissions

The Info tab provides technical information; it does not itself change those settings.

## Site Health Audit Checklist

Use this checklist when auditing a WordPress website.

### General status

- [ ] Review all critical issues.
- [ ] Evaluate recommended improvements.
- [ ] Record the WordPress and PHP versions.
- [ ] Check for pending core, theme, and plugin updates.

### Connectivity

- [ ] Verify HTTPS and certificate status.
- [ ] Investigate loopback request failures.
- [ ] Check REST API requests if a warning is present.
- [ ] Verify outbound connectivity when WordPress.org cannot be reached.

### Scheduled tasks

- [ ] Investigate scheduled event warnings.
- [ ] Confirm the intended WP-Cron or system cron configuration.
- [ ] Check for recurring PHP errors.

### Server and filesystem

- [ ] Review PHP limits and extensions.
- [ ] Check disk space and inode availability.
- [ ] Verify relevant filesystem permissions.
- [ ] Review server and PHP error logs.

### Security and maintenance

- [ ] Confirm that debug output is not exposed publicly.
- [ ] Review plugin and theme maintenance status.
- [ ] Verify backups before making significant changes.
- [ ] Document the root cause and the fix.

## Security Best Practices

- Keep WordPress core, themes, plugins, and PHP updated.
- Protect Site Health exports and diagnostic logs.
- Never publish passwords, API keys, or private configuration files.
- Avoid exposing PHP errors to website visitors.
- Do not disable security controls just to make a warning disappear.
- Back up the site before significant configuration changes.
- Test potentially disruptive changes on staging.
- Document changes made to production systems.
- Re-run Site Health after applying a fix.

## Official References

- [WordPress Site Health Screen](https://wordpress.org/documentation/article/site-health-screen/)
- [WordPress Site Health Developer API](https://developer.wordpress.org/apis/site-health/)
- [WordPress Cron](https://developer.wordpress.org/plugins/cron/)
- [WordPress Debugging](https://developer.wordpress.org/advanced-administration/debug/debug-wordpress/)
- [WordPress REST API Handbook](https://developer.wordpress.org/rest-api/)
- [WordPress Requirements](https://wordpress.org/about/requirements/)

## Contributing

Contributions are welcome.

When submitting a troubleshooting improvement:

1. Describe the Site Health warning and affected environment.
2. Include relevant WordPress, PHP, and hosting details.
3. Provide sanitized error messages or log entries.
4. Explain how the cause was verified.
5. Document the solution and any limitations.
6. Never include credentials, customer information, or private production data.

## License

This repository can be distributed under the MIT License. Add a `LICENSE` file containing the MIT License text if you choose this license.

---

**Disclaimer:** This is an independent troubleshooting resource and is not an official WordPress project. Always verify the underlying cause before making potentially disruptive production changes.
