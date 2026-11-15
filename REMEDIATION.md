# Remediation — SecDojo Comepete

## Critical

### Remove dynamic PHP evaluation
Remove the custom template processor's use of `eval()`. Treat quiz descriptions and all other user-supplied content as data. Use a maintained template engine with safe configuration and no PHP evaluation of user-controlled content.

### Protect application data
Move `auth.db`, JSON stores and other sensitive files outside the document root. If they must remain beneath the web root temporarily, explicitly deny HTTP access to them.

### Disable directory listing
For Apache, disable directory indexes:

```apache
Options -Indexes
```

Apply the directive in the appropriate virtual-host or directory configuration and verify the effective configuration.

## High

### Use secure password storage
Use PHP's `password_hash()` to store password hashes and `password_verify()` to verify submitted passwords. Do not store plaintext passwords or compare plaintext values.

### Restrict file inclusion
Do not pass raw request parameters to `include` or `require`. Map a small set of approved page identifiers to fixed server-side paths. Validate that any resolved path remains within the intended application directory.

### Prevent direct access to sensitive files
Deny HTTP access to database files, JSON data stores, backups, configuration files and source-control metadata. Do not rely on obscure filenames as a security control.

## Medium

- Rotate credentials that may have been exposed and invalidate existing sessions.
- Run the web process with least privilege.
- Restrict the web process's access to unrelated host files.
- Monitor sensitive data stores for unexpected reads and writes.
- Review application logs for suspicious requests to include parameters and sensitive data paths.

## Validation checklist

- [ ] `/data/` no longer exposes a directory index.
- [ ] Database and JSON data files cannot be downloaded over HTTP.
- [ ] Passwords are stored as secure hashes.
- [ ] Only explicitly approved page names can be included.
- [ ] User-controlled template content is never evaluated as PHP.
- [ ] The web process has only the filesystem permissions it requires.
- [ ] Exposed credentials and sessions have been rotated or invalidated.
