---
title: WordPress Site Networks
subtitle: Configure
description: Overview of WordPress multisite support on the Pantheon Platform.
layout: guide
type: guide
cms: "WordPress"
categories: [develop]
tags: [multisite]
permalink: docs/guides/multisite/config/
editpath: multisite/03-config.md
image: multisite
---

You can use this doc if you have new WordPress Site Network created for you by a Pantheon employee. If you do not have a WordPress Site Network, refer to the [introduction](/guides/multisite) page of this guide. After you visit the Dev environment's site URL, you should be redirected to the WordPress web-based installer.

<Alert title="Note" type="info">

Adjust placeholders in code snippets as needed throughout this guide. This includes placeholders such as `<site>` and `<env>` in Terminus commands, in addition to placeholders in brackets `<>` in larger code blocks.

</Alert>

## Install the WordPress Site Network

Using [Terminus](/terminus) to install a WordPress Site Network is recommended.

1. Install the most recent release of Terminus:

  ```bash{promptUser: user}
  curl -O https://raw.githubusercontent.com/pantheon-systems/terminus-installer/master/builds/installer.phar && php installer.phar install
  ```

1. [Generate a Machine Token](https://dashboard.pantheon.io/machine-token/create), then authenticate Terminus:

    ```bash{promptUser: user}
    terminus auth:login --machine-token=<machine-token>
    ```

1. Make sure the site's connection mode is set to SFTP:

    ```bash{promptUser: user}
    terminus connection:set <site>.dev sftp
    ```

1. Use Terminus to execute the `wp core multisite-install` command. You can refer to the [documentation](https://developer.wordpress.org/cli/commands/core/multisite-install/) for more information.

  <Alert title="Note" type="info">

  The default behavior for this command is to create a Site Network with the subdirectory configuration. To create your network with the subdomain configuration, add the `--subdomains` flag.

  </Alert>

  ```bash{promptUser: user}
  terminus wp <site>.<env> -- core multisite-install --url=<url> --title=<site-title> --admin_user=<username> --admin_email=<email>
  ```

 After you successfully install a new WordPress Site Network, a message is displayed that is similar to the following:

  ```bash{outputLines: 2-6}
  terminus wp sitenetworks.dev -- core multisite-install --url=dev-sitenetworks.pantheonsite.io --title="Site Networks" --admin_user=aghost --admin_email=aghost@pantheon.io
  Admin password: abcdefgnotarealpassword
  Created single site database tables.
  Set up multisite database tables.
  Added multisite constants to 'wp-config.php'.
  Success: Network installed. Dont forget to set up rewrite rules.
  ```

The message confirms your WordPress Site Network is installed.

## Configure the WordPress Site Network

The `wp core multisite-install` command that we ran in the previous section modifies the `wp-config.php` file. The modification sets the `DOMAIN_CURRENT_SITE` constant, which assigns a specific URL to your WordPress Site Network.

To ensure it works on the Pantheon platform, you need to adjust the configuration so that the `DOMAIN_CURRENT_SITE` constant is defined conditionally based on the given environment:

1. Navigate to **<span class="glyphicons glyphicons-embed-close"></span> Code** in the **<span class="glyphicons glyphicons-wrench"></span> Dev** tab of your Site Dashboard.

1. Click **Connect with SFTP** to access the credentials for connecting to your preferred SFTP client.

1. Click **Open SFTP client**, and enter your User Dashboard password when prompted.

  If you run into issues, please refer to Pantheon's [SFTP documentation](/sftp#sftp-connection-information).

1. Open the `code` folder in your SFTP client, and download your site's `wp-config.php` file.

1. Locate the configuration added by WP-CLI, and **comment out** the line that sets `DOMAIN_CURRENT_SITE`. You will replace this variable in the following step. For example:

  ```php:title=wp-config.php
  define( 'WP_ALLOW_MULTISITE', true );
  define( 'MULTISITE', true );
  define( 'SUBDOMAIN_INSTALL', false );
  $base = '/';
  # define( 'DOMAIN_CURRENT_SITE', 'example.com' );
  define( 'PATH_CURRENT_SITE', '/' );
  define( 'SITE_ID_CURRENT_SITE', 1 );
  define( 'BLOG_ID_CURRENT_SITE', 1 );
  ```

1. Add the following code block to your `wp-config.php` file, under the lines mentioned in the previous step:

  ```php:title=wp-config.php
  /**
   * Define DOMAIN_CURRENT_SITE conditionally.
   */
  if ( ! empty( $_ENV['PANTHEON_ENVIRONMENT'] ) ) {
    switch( $_ENV['PANTHEON_ENVIRONMENT'] ) {
      case 'live':
        // Value should be the primary domain for the Site Network.
        define( 'DOMAIN_CURRENT_SITE', 'live-<site>.pantheonsite.io' );
        // Once you map a domain to Live, you can change DOMAIN_CURRENT_SITE
        // define( 'DOMAIN_CURRENT_SITE', 'example-network.com' );
        break;
      case 'test':
        define( 'DOMAIN_CURRENT_SITE', 'test-<site>.pantheonsite.io' );
        break;
      case 'dev':
        define( 'DOMAIN_CURRENT_SITE', 'dev-<site>.pantheonsite.io' );
        break;
      default:
        # Catch-all to accommodate default naming for multi-dev environments.
        define( 'DOMAIN_CURRENT_SITE', $_ENV['PANTHEON_ENVIRONMENT'] . '-' . $_ENV['PANTHEON_SITE_NAME'] . '.pantheonsite.io' );
        break;
      }
  }
  ```

If your site uses a custom domain instead of a platform domain, edit the `wp-config.php` to reflect the custom domain. 
  
You may notice that the `test` and `dev` cases are redundant. Remove the `test` and `dev` cases if you don't intend to add custom domains to those environments. Generally, you should conditionally define the `DOMAIN_CURRENT_SITE` constant based on the current Pantheon environment (Dev, Test, Live or Multidev).

1. Save your changes and upload the `wp-config.php` file to Pantheon's Dev environment after you edit.

<Alert title="Note" type="info">

A warning may appear in the WordPress dashboard that you need to update your `.htaccess` file. Since Pantheon used Nginx and your site is already pre-configured for multisite use by your Account Manager, you can ignore this warning.

</Alert>

## Develop the Site Network

Congratulations on setting up your first WordPress Site Network. When you log in to the WordPress Dashboard, you'll see a **My Sites** menu item in the toolbar:

![Enabling the network](../../../images/wp-network-admin-sites.png)

You will have one site and you can create another if needed. If you chose to use WordPress Site Networks with subdirectories, you'll be able to access the site right away. If you chose to use subdomains, you'll need to map a custom hostname to the environment before you can access the new environment.

<Accordion title="Mapping Custom Hostnames"  id="map-cust-hostname" icon="wrench">

### Map Custom Hostnames (subdomain configurations only)

Upon installation and configuration the main site will load properly (e.g., `dev-<example>.pantheonsite.io`). However, additional network sites created will fail to load because `pantheonsite.io` doesn't support sub-sub-domains (e.g., `<new-sub-site>.dev-<example>.pantheonsite.io`). WordPress Site Networks using a subdomain configuration require custom domains to be mapped to each network site in order to load properly.

1. Access the domain's DNS settings wherever they are managed.
1. Create a wildcard CNAME `*.dev.example.com` that maps to the Dev environment's platform domain, `dev-<example>.pantheonsite.io`. Doing so ensures all hostnames mapped to the environment will load without additional DNS records.
1. Map domains to the Pantheon Dev environment using Terminus. For example:

    ```bash
    # add dev.example.com for the network's main site
    terminus domain:add <site>.dev dev.example.com

    # add subsite.example.com for a subsite on the network
    terminus domain:add <site>.dev subsite.example.com
    ```

1. Update the conditional `DOMAIN_CURRENT_SITE` definition in your `wp-config.php` file to accommodate the site's new domains.

After these steps are complete, both sites on the WordPress Site Network should load with their new URLs.

</Accordion>

Explore the WordPress Network Dashboard to become familiar with the variety of additional settings. You can review the options that are available for each site you create, manage users across WordPress Multisite, and learn about the network settings. After you explore the WordPress Network Dashboard, learn how to use the WordPress Site Network with the Pantheon Workflow.
