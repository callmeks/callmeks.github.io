---
title: OWASP Juice Shop Installation
categories: [Installation]
tags: [Installation]
---
- [1. Install Node.js](#1-install-nodejs)
- [2. Download Packaged Distribution](#2-download-packaged-distribution)
- [3. Run and deploy OWASP Juice Shop](#3-run-and-deploy-owasp-juice-shop)
- [Useful-links](#useful-links)

## 1. Install Node.js

- For Ubuntu & Debian, install node.js with the following command.
- *Current LTS version for Node.js is version 18.*
- For more info about other version of node.js, click [here](https://github.com/nodesource/distributions#debinstall).

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
sudo apt-get install -y nodejs
```

## 2. Download Packaged Distribution

- Download the latest OWASP juice shop release [here](https://github.com/juice-shop/juice-shop/releases/).
- Remember to select the correct OS and node.js version.
- Download and extract the file.

## 3. Run and deploy OWASP Juice Shop

- `cd` into the directory and run command `npm start`
- Use command `sudo` as well if current directory is not owned by current user. 
- Make sure to use the same node.js version to avoid any issue. 
- If there are no issue, try to access [http://localhost:3000/](http://localhost:3000/).

## Useful-links
- [Youtube guide to install juice shop](https://youtu.be/muPEC9kw_q8)
- [GitHub juice-shop](https://github.com/juice-shop/juice-shop/)
- [GitHub Node Source](https://github.com/nodesource/distributions)
- [GitHub juice-shop release](https://github.com/juice-shop/juice-shop/releases/)