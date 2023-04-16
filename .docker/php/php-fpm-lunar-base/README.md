This is patched version of [php:8.1-fpm-bullseye](https://github.com/docker-library/php/blob/b93e90a02e9834fe3865f5e7b61f62186b2d05f5/8.1/bullseye/fpm/Dockerfile) base image `debian:bullseye-slim` has been replaced by `ubuntu:lunar`

```patch
Subject: [PATCH] Use Ubuntu Lunar instead of Debian Bullseye
---
Index: 8.1/bullseye/fpm/Dockerfile
<+>UTF-8
===================================================================
diff --git a/8.1/bullseye/fpm/Dockerfile b/8.1/bullseye/fpm/Dockerfile
--- a/8.1/bullseye/fpm/Dockerfile	(revision 59da5b0a2dda2bc22d053263ea5fc860ddca65a2)
+++ b/8.1/bullseye/fpm/Dockerfile	(date 1681642278916)
@@ -4,16 +4,16 @@
 # PLEASE DO NOT EDIT IT DIRECTLY.
 #
 
-FROM debian:bullseye-slim
+FROM ubuntu:lunar
 
-# prevent Debian's PHP packages from being installed
+# prevent Ubuntu's PHP packages from being installed
 # https://github.com/docker-library/php/pull/542
 RUN set -eux; \
 	{ \
 		echo 'Package: php*'; \
 		echo 'Pin: release *'; \
 		echo 'Pin-Priority: -1'; \
-	} > /etc/apt/preferences.d/no-debian-php
+	} > /etc/apt/preferences.d/no-ubuntu-php
 
 # dependencies required for running "phpize"
 # (see persistent deps below)
@@ -36,6 +36,11 @@
 		ca-certificates \
 		curl \
 		xz-utils \
+		libxml2 \
+		sqlite3 \
+		libargon2-1 \
+		libonig5 \
+		libsodium23 \
 	; \
 	rm -rf /var/lib/apt/lists/*
 

```
