---
title: Caching - Tutorial - Boost your CMS (from the example of WordPress)
keywords: wordpress, redis, dbaas, cms
excerpt: Boost the performances of your WordPress CMS with a Caching service
updated: 2024-06-12
---

## Objective

Do you have a CMS website or blog, perhaps powered by WordPress? Learn how to speed up page load times and SQL queries using a Caching service! In this tutorial, we will set up a Caching service for your CMS data. All users, even administrators and especially visitors, will directly benefit from a better performance of the website.

> [!warning]
>
> OVHcloud provides services for which you are responsible for their configuration and management. You are therefore responsible for their proper functioning.
>
> This tutorial is designed to help you as much as possible with common tasks. If you are having difficulty performing these actions, please contact a specialized service provider and/or discuss it with our community of users on <https://community.ovh.com/en/>. OVHcloud cannot provide you with technical support in this regard.
>

### CMS platform

For this tutorial we propose to use the WordPress CMS, running on Linux Ubuntu with MySQL as the main database. To get the object cache we will use a Public Cloud Database for Caching and a WordPress plugin for Redis®.

> [!warning]
>This tutorial will show you how to use one or more OVHcloud solutions with external tools, and will describe the actions to be carried out in a specific context. Please remember to adapt these actions to fit your situation.
>
>If you encounter any difficulties performing these actions, please contact a [specialist service provider](https://partner.ovhcloud.com/en-gb/directory/) and/or discuss the issue with [our community](https://community.ovh.com/en/). OVHcloud cannot provide you with technical support in this regard.
>
> Remember to back up your files prior to making any changes.
>

## Requirements

This tutorial requires:

- A running WordPress
- A WordPress administrator account to install a plugin
- A [PHP Redis® client](https://redis.io/clients#php){.external} supported by the plugin
- An OVHcloud Public Cloud database for Caching

## Instructions

### Tutorial technical environment

The main software component releases used for the following screenshots and example are as follows:

- Linux Ubuntu 21.10 / apache2 2.4.48 / PHP 8.0 / MySQL 8.0.27
- WordPress: 5.8.2
- phpredis: 5.3.2
- Redis Object Cache: 2.0.21
- Redis® open source: 6.2

### Configuring your CMS: WordPress

If you need to install WordPress, please refer to the official guide: [WordPress How To Install](https://wordpress.org/support/article/how-to-install-wordpress/){.external}.

Once done, we will set up the WordPress backend correctly.
For the next steps, let's consider that you have an up and running WordPress system on a LAMP (Linux, Apache, MySQL, PHP) platform.

### PHP Redis® Client: PhpRedis

As WordPress is based on PHP, we will need a PHP Redis® client. We propose to use one of the main recommended [PHP Redis® clients](https://redis.io/clients#php){.external}: PhpRedis.

As TLS is required to secure the connection to the OVHcloud Public Cloud database for Caching, we have to use the v5.3.x release of PhpRedis or above.

Please check that PhpRedis is installed, with a compliant release:

- If you do have SSH access to your WordPress server you can use the following command:

```bash
dpkg -l php-redis
```
In case it is not available on your system, please install it:

```bash
sudo apt install php-redis
sudo systemctl restart apache2
```
- If you don't have SSH access to the system, use the phpinfo() function and do a search on the Redis® module as in the following example:

![Redis Plugin PHPInfo](images/Redis_phpinfo.png){.thumbnail}

### Redis® WordPress plugin

There are multiple Redis® plugins that could be used to integrate to a Caching service. You can search them directly in the Plugin menu. For this tutorial we decided to use the most popular one: Redis Object Cache. Click on the Install Now button:

![Search Redis Plugin](images/Redis_plugin_list.png){.thumbnail}

You can get more information related to this WordPress plugin at the official [WordPress plugin documentation - Redis Object Cache](https://wordpress.org/plugins/redis-cache/){.external}.

### Configure WordPress to connect to the Caching service

#### Modify wp-config.php

Before activating the Redis Object Cache plugin, it is required to configure the Caching service connection parameters.
By default, this plugin looks for a Caching service hosted in the same server as your WordPress.
It's not our case scenario here, since the Caching service is hosted in OVhcloud Public Cloud Databases.

We will follow the official plugin wiki to define the remote server: <https://github.com/rhubarbgroup/redis-cache/wiki/Configuration-Options>.
This has to be done in the WordPress configuration file (**wp-config.php** located in /var/www/wordpress/ in our example) and add the following lines:

```php
define('WP_REDIS_CLIENT', 'phpredis' );
define('WP_REDIS_SCHEME', 'tls');
define('WP_REDIS_HOST', 'my_redis-123456-123456.database.cloud.ovh.net');
define('WP_REDIS_PORT', '20185');
define('WP_REDIS_PASSWORD', ['my_redis_user', 'my_password_here']);
define('WP_CACHE_KEY_SALT', 'myOVHcloudRedis_' );
```
#### Modify the host

The salt key is not mandatory, even more if you have only one application using a Caching service, but if you have several WordPress sites, it will be useful to determine **what-pushes-what**.

#### Check that authorised IPs are updated

Do not forget to add the WordPress IP address to the authorised IPs list of the Caching service.

![Add Authorised IP](images/Redis_add_ip_whitelist.png){.thumbnail}

#### Activate and enable the plugin

Now all installation and configurations are done. So let's activate the plugin.

![Activate Redis Object Cache Plugin](images/Redis_Plugin_activate.png){.thumbnail}

Then enable the object cache functionality.

![Enable Redis Object Cache](images/Redis_Plugin_enable.png){.thumbnail}

If you run into any problems enabling the Redis Object Cache plugin, start your investigation by looking at the information available in the Diagnostics tab on the Plugin Settings:

![Diagnostics Redis Object Cache Plugin](images/Redis_Plugin_Diagnostics.png){.thumbnail}

#### Additional parameters

Please find below more documentation on the additional available parameters that you might want to use to optimise your Caching service usage:

- [Redis Object Cache connection parameters](https://github.com/rhubarbgroup/redis-cache/wiki/Connection-Parameters){.external}
- [Redis Object Cache configuration options](https://github.com/rhubarbgroup/redis-cache/wiki/Configuration-Options){.external}

### Checking performance boost

#### Validate that cache is populated

If you can use a CLI to query the Caching service, you can check that information populated to it after browsing some WordPress pages.

In our case:
```bash
my_redis-123456-123456.database.cloud.ovh.net:20185> SCAN 0
1) "480"
2) 1) "myvps_wp:post_meta:782"
2) "myvps_wp:post_meta:418"
3) "myvps_wp:terms:13"
4) "myvps_wp:site-options:1-notoptions"
5) "myvps_wp:options:alloptions"
6) "myvps_wp:post_meta:905"
7) "myvps_wp:options:notoptions"
8) "mywpvps_wp:posts:get_page_by_path-83503918bbdb048f39fd1d3e462c8251-0.24924400 1637188954"
9) "myvps_wp:useremail:yo@chrisam.es"
10) "myvps_wp:terms:3"
```
#### Install a query monitoring plugin

If you want to have more information about how the cache has impacted the performance, you can measure the gain in response delay directly with your browser. You can also see more details using an additional WordPress plugin like [Query Monitor](https://wordpress.org/plugins/query-monitor/){.external}.

#### Example of performance boost with a basic WordPress site

Using a simple WordPress site populated with poor content and a very limited number of posts, let's take a look at the improvement.

We will look at the number of database requests for both cases: with and without the cache enabled.

Without the cache, there is a total of 15 database queries taking 0.0235s:

![Performance measure without cache](images/Perf_without_cache.png){.thumbnail}

With the cache, there is a total of 2 database queries taking 0.0043s:

![Performance measure with cache](images/Perf_with_cache.png){.thumbnail}

#### Two main benefits using a Caching service

As seen, using a Caching service you will see a performance gain for your CMS. It will be even more efficient as soon as your site stores more and larger data. You will also directly benefit from the fact that the MySQL server will be less stressed with the load/concurrent accesses, which will offer the opportunity to handle more and more users.

## Go further

- [OVHcloud documentation on managed Public Cloud Databases](/products/public-cloud-databases)
- [Caching capabilities](/pages/public_cloud/public_cloud_databases/redis_01_capabilities)
- [How to connect to a Caching service with CLI](/pages/public_cloud/public_cloud_databases/redis_03_connect_cli)
- [How to connect to a Caching service with PHP](/pages/public_cloud/public_cloud_databases/redis_04_connect_php)
- [Caching Roadmap](https://github.com/orgs/ovh/projects/16/views/5?card_filter_query=label%3Aredis)

Join our community of users on <https://community.ovh.com/en/>.

## We want your feedback!

We would love to help answer questions and appreciate any feedback you may have.

If you need training or technical assistance to implement our solutions, contact your sales representative or click on [this link](/links/professional-services) to get a quote and ask our Professional Services experts for a custom analysis of your project.

Are you on Discord? Connect to our channel at <https://discord.gg/ovhcloud> and interact directly with the team that builds our databases service!
