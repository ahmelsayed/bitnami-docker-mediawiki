FROM bitnami/minideb-extras:stretch-r447
LABEL maintainer "Bitnami <containers@bitnami.com>"

# Install required system packages and dependencies
RUN install_packages libbz2-1.0 libc6 libcomerr2 libcurl3 libexpat1 libffi6 libfreetype6 libgcc1 libgcrypt20 libgmp10 libgnutls30 libgpg-error0 libgssapi-krb5-2 libhogweed4 libicu57 libidn11 libidn2-0 libjpeg62-turbo libk5crypto3 libkeyutils1 libkrb5-3 libkrb5support0 libldap-2.4-2 liblzma5 libmemcached11 libmemcachedutil2 libncurses5 libnettle6 libnghttp2-14 libp11-kit0 libpcre3 libpng16-16 libpq5 libpsl5 libreadline7 librtmp1 libsasl2-2 libsqlite3-0 libssh2-1 libssl1.0.2 libssl1.1 libstdc++6 libsybdb5 libtasn1-6 libtidy5 libtinfo5 libunistring0 libxml2 libxslt1.1 libzip4 locales zlib1g
RUN bitnami-pkg unpack apache-2.4.39-2 --checksum ff55ee9cccf484bac61fa91558fc7f8445e91ea00bb104aca216f08aea003c6b
RUN bitnami-pkg unpack php-7.3.8-0 --checksum cc67ec3f4ff34c0f757e59e28b43572c4b20b938b84cd88a09a186dfd327abd0
RUN bitnami-pkg unpack mysql-client-10.3.17-0 --checksum 71d59fafc3e7e3e598af0c2b6788d77558630afe647ec1f922ffdd5025f3d737
RUN bitnami-pkg install libphp-7.3.8-0 --checksum 5cfa7600f2881c7cbd2f6aa5a77c9b900d5a2e85fb71d3e482f2b64e504c6fb3
RUN bitnami-pkg unpack mediawiki-1.33.0-0 --checksum 38ca80f59e826a16aa2bf40574d3c95ffa4324b79d2ee3a535933719ed04641c
RUN ln -sf /dev/stdout /opt/bitnami/apache/logs/access_log
RUN ln -sf /dev/stderr /opt/bitnami/apache/logs/error_log
RUN echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen && locale-gen

COPY rootfs /
ENV ALLOW_EMPTY_PASSWORD="no" \
    APACHE_HTTPS_PORT_NUMBER="" \
    APACHE_HTTP_PORT_NUMBER="" \
    APACHE_SET_HTTPS_PORT="no" \
    APACHE_SET_HTTP_PORT="no" \
    BITNAMI_APP_NAME="mediawiki" \
    BITNAMI_IMAGE_VERSION="1.33.0-debian-9-r39" \
    LANG="en_US.UTF-8" \
    LANGUAGE="en_US:en" \
    MARIADB_HOST="mariadb" \
    MARIADB_PORT_NUMBER="3306" \
    MARIADB_ROOT_PASSWORD="" \
    MARIADB_ROOT_USER="root" \
    MEDIAWIKI_DATABASE_NAME="bitnami_mediawiki" \
    MEDIAWIKI_DATABASE_PASSWORD="" \
    MEDIAWIKI_DATABASE_USER="bn_mediawiki" \
    MEDIAWIKI_EMAIL="user@example.com" \
    MEDIAWIKI_PASSWORD="bitnami123" \
    MEDIAWIKI_USERNAME="user" \
    MEDIAWIKI_WIKI_NAME="Bitnami MediaWiki" \
    MYSQL_CLIENT_CREATE_DATABASE_NAME="" \
    MYSQL_CLIENT_CREATE_DATABASE_PASSWORD="" \
    MYSQL_CLIENT_CREATE_DATABASE_PRIVILEGES="ALL" \
    MYSQL_CLIENT_CREATE_DATABASE_USER="" \
    PATH="/opt/bitnami/apache/bin:/opt/bitnami/php/bin:/opt/bitnami/php/sbin:/opt/bitnami/mysql/bin:$PATH" \
    SMTP_HOST="" \
    SMTP_HOST_ID="" \
    SMTP_PASSWORD="" \
    SMTP_PORT=""

VOLUME [ "/certs" ]

EXPOSE 80 443

ENTRYPOINT [ "/app-entrypoint.sh" ]
CMD [ "httpd", "-f", "/opt/bitnami/apache/conf/httpd.conf", "-DFOREGROUND" ]
