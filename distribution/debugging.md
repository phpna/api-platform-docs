# 调试

默认的Docker stack没有Xdebug步骤。将[Xdebug](https://xdebug.org/) 添加到项目中很容易，这主要用于在开发环境中进行例如测试或远程API请求等调试的目的。

## 将开发环境的stage添加到Dockerfile

为避免将启用了的Xdebug扩展的API Platform部署到生产环境，建议在`api/Dockerfile`的末尾添加自定义stage。

```Dockerfile
# api/Dockerfile
FROM api_platform_php as api_platform_php_dev

ARG XDEBUG_VERSION=2.6.0
RUN set -eux; \
	apk add --no-cache --virtual .build-deps $PHPIZE_DEPS; \
	pecl install xdebug-$XDEBUG_VERSION; \
	docker-php-ext-enable xdebug; \
	apk del .build-deps
```

## 使用Docker Compose的配置覆盖特性配置Xdebug

使用[配置覆盖](https://docs.docker.com/compose/reference/overview/#specifying-multiple-compose-files)
 file named `docker-compose.override.yml` ensures that the production

一个名为`docker-compose.override.yml`的文件可确保生产配置保持不变。

例如，配置覆盖文件可能如下所示：

```yml
version: "3.4"

services:
  php:
    build:
      target: api_platform_php_dev
    environment:
      # See https://docs.docker.com/docker-for-mac/networking/#i-want-to-connect-from-a-container-to-a-service-on-the-host
      # See https://github.com/docker/for-linux/issues/264
      # The `remote_host` below may optionally be replaced with `remote_connect_back`
      XDEBUG_CONFIG: >-
        remote_enable=1
        remote_host=host.docker.internal
        remote_connect_back=1
        remote_port=9000
        idekey=PHPSTORM
      # This should correspond to the server declared in PHPStorm `Preferences | Languages & Frameworks | PHP | Servers`
      # Then PHPStorm will use the corresponding path mappings
      PHP_IDE_CONFIG: serverName=api-platform
```

## 故障排除

使用以下命令检查安装。请求的Xdebug版本应显示在输出中。

```bash
$ docker-compose exec php php --version

PHP 7.2.8 (cli) (built: Jul 21 2018 08:09:37) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.2.8, Copyright (c) 1999-2018, by Zend Technologies
    with Xdebug v2.6.0, Copyright (c) 2002-2018, by Derick Rethans
```
