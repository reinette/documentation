You must confirm that your site meets the following requirements before you continue:

- Ensure your site has the [Pantheon empty repository](https://github.com/pantheon-systems/empty) in its upstream.

  ### Use Terminus to Confirm the Empty Upstream

  Run the command `terminus site:info $SITE` to display the site's basic information and properties.
 
 The following values indicate that a site is using an `empty` upstream: 
  * The `Framework` is `drupal8`
  * The `Upstream` includes `https://github.com/pantheon-systems/empty.git`
  
  The following is an abridged example of the output for the `terminus site:info $SITE` command, if the site upstream is set to `empty`:

  ```bash{outputLines:2-18}
  terminus site:info $SITE
  ------------------ -------------------------------------------------------------------------------------
  ID                 abdc3ea1-fe0b-1234-9c9f-3cxeAA123f88
  Name               anita-drupal
  Label              AnitaDrupal
  Created            2019-12-02 18:28:14
  Framework          drupal8
  ...
  Upstream           4c7176de-e079-eed1-154d-44d5a9945b65: https://github.com/pantheon-systems/empty.git
  ...
  ------------------ -------------------------------------------------------------------------------------
  ```

- The site does not use another package and library manager, such as [Ludwig](https://www.drupal.org/project/ludwig).
