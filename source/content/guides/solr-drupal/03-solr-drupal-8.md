---
title: Pantheon Search for Drupal
subtitle: Using Solr on Drupal 8
description: Detailed information on using Pantheon Search with Drupal 8.
cms: "Drupal 8"
categories: [integrate]
tags: [solr, search, modules]
contributors: [peter-pantheon, cityofoaksdesign]
layout: guide
showtoc: true
permalink: docs/guides/solr-drupal/solr-drupal-8
anchorid: solr-drupal
editpath: solr-drupal/03-solr-drupal-8.md
---

<Alert title="Important Note" type="info">

**Pantheon Search** derives from Solr and can perform full-text content searching in a single language.

<Partial file="solr-version.md" />

If your search needs include geospatial search, emojis, or multilingual search, consider [OpenSolr](/opensolr) or another alternative search.

Pantheon Search supports [Search API Solr 8.x-1.x](https://www.drupal.org/project/search_api_solr), which reached end-of-life in December 2021. Search API Solr 8.x-1.x should continue to work as long as the Search API Pantheon module is also being used, following the installation directions below.

</Alert>

## Before You Begin

Be sure that you:

- Enable Solr in the Pantheon Site Dashboard: **Settings** > **Add Ons** > **Apache Solr Index Server: Add**.

- Install [Composer](https://getcomposer.org/)

- Create a Composer-managed site on Pantheon following the [Build Tools](/guides/build-tools) guide, or [convert an existing Drupal site to use Composer](/guides/composer-convert) guide.

<Alert title="Warning" type="danger">

Pantheon Search on Drupal 8 requires a Composer-managed workflow, as described in our [Build Tools](/guides/build-tools) and [Convert to Composer](/guides/composer-convert) guides. Since one module relies on [Solarium](http://www.solarium-project.org/), an external library, in addition to Composer's autoloader, we cannot support non-Composer workflows for Pantheon Search on Drupal 8. For details, see [this Drupal.org issue](https://www.drupal.org/node/2858750).

</Alert>

## Install Pantheon Search on Drupal 8

### Install the Search API Pantheon Module

1. Navigate to the project's root directory on your local computer. If you have access to [Multidev](/multidev), checkout a new branch from master:

  ```bash{promptUser: user}
  git checkout -b solr master
  ```

  Otherwise, continue from the master branch.

1. Add the Search API Pantheon module as a required dependency:

  ```bash{promptUser: user}
  composer require "drupal/search_api_pantheon ~1.0" --prefer-dist
  ```

1. You should now have the Search API Pantheon module installed along with its dependencies. Run `git status` to make sure you see the expected result. Commit and push the changes:

  <TabList>

  <Tab title="Without Multidev" id="install-nomulti" active={true}>

  ```bash{promptUser: user}
  git commit -am "Require drupal/search_api_pantheon ~1.0"
  git push origin master
  ```

  </Tab>

  <Tab title="With Multidev" id="install-multidev">

  ```bash{promptUser: user}
  git commit -am "Require drupal/search_api_pantheon ~1.0"
  git push origin solr
  terminus multidev:create <site>.dev solr
  ```

  </Tab>

  </TabList>

## Configure Pantheon Search

To configure the connection with Pantheon, set the [connection mode](/sftp/#sftp-mode) to SFTP and complete the following steps.

### Enable Modules

Enable the Search API Pantheon module via the [Drupal interface](https://www.drupal.org/docs/8/extending-drupal-8/installing-contributed-modules-find-import-enable-configure-drupal-8#enable_your_mod). When prompted, click **Continue** to enable the [Search API](https://www.drupal.org/project/search_api) and [Search API Solr](https://www.drupal.org/project/search_api_solr) modules:

### Disable Drupal Core's Search Module (Optional)

If you are using Search API, then you probably will not be using Drupal Core's Search module. Uninstall the Search core module from `/admin/modules/uninstall` to avoid confusion in further configuration steps.

### Add The Search Server

Navigate to  `/admin/config/search/search-api/add-server` and configure the following, then click **Save**:

- Server name: Pantheon

- Backend: Solr

- Solr Connector: Pantheon

- Schema file: `modules/search_api_solr/solr-conf/4.x/schema.xml` (recommended)

You can name the server anything you want but using something like "Pantheon" is a good way to remember where the connection goes. The Search API module provides schema files for each version of Solr (4, 5, and 6). You can customize schema files by copying these examples to your own custom module and editing them. If you are just getting started, we recommend selecting the file for Solr 4.

When deploying Solr to other environments (Test/Live/Multidevs) for the first time, first navigate to your Server settings page at `admin/config/search/search-api/server/pantheon/edit` and click **Save**, so you can post the Solr schema in those environments.

### Add Search Index

Navigate to `admin/config/search/search-api/add-index` and name your index, then choose a data source. If this is your first time using Search API, start by selecting **Content** as a data source. This option will index articles, basic pages, and other node types you have configured.

Select **Pantheon** as the server, then click **Save and add fields**. Add fields to be included in the index and click **Done**.

After adding fields the configuration, make sure the index is full by clicking **Index now** or by running cron.

### Export Configuration

It is a best practice in Drupal 8 to export your changes to `yml` files. You can quickly export configuration changes via [Terminus](/terminus):

```bash{promptUser: user}
terminus drush site.env -- config-export -y
```

Replace `site` and `env` with your site name and the environment (Dev, Multidev, etc), respectively.

### Search the Index

To actually search your index you will need a module like [Search API Pages](https://www.drupal.org/project/search_api_page), which allows for the configuration of search forms on their own pages.

## Solr Versions and Schemas

The version of Solr on Pantheon is Apache Solr v3.6. To accommodate this older version of Solr, use the `8.x-1.x` branch of [Search API Solr](https://www.drupal.org/project/search_api_solr) and its Solr 4 schema file.

<Partial file="solr-commit-changes.md" />

## Extend Solr for Drupal 8

### Apache Tika

The [Apache Tika](https://tika.apache.org/) toolkit detects and extracts metadata and structured text content from various documents using existing parser libraries.

Tika can extract content from a number of document formats such as HTML, XML, Microsoft Office document formats, and PDFs and more.

Download and install the Search API Attachments module ([search_api_attachments](https://www.drupal.org/project/search_api_attachments)), then configure the module's settings.

1. Go to the Search API Attachments settings page at: `/admin/config/search/search_api_attachments` and edit the following fields:

   - **Extraction method:** Tika Extractor
   - **Path to java executable:** `java`
   - **Path to Tika .jar file:** `/srv/bin/tika-app-1.18.jar`

1. Verify that your site is able to extract text from documents. Click **Submit and test extraction**.

If everything is working correctly, the message "Extracted data: Congratulations! The extraction seems to be working! Yay!" will be displayed.

## Safely Remove Pantheon Search

The following code changes are required before Pantheon Search can be safely uninstalled and disabled:

<Partial file="remove-addons/d8-solr.md" />

## Troubleshooting

### Solr Verification Check

Because we are posting the 4.x schema to a 3.x Solr instance, the schema verification check can fail and prevent indexing. You can disable the schema check by checking the **Skip schema verification** box in the UI.

## See Also

- [Search API Docs](https://www.drupal.org/node/1250878).
