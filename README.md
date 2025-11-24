# Ascend Plausible analytics installer

This recipe will install the Plausible and Key contrib modules, and set up
keys & config overrides for those keys, to prevent sensitive data being present
in database exports.

It assumes you have access to the project root to create folders & files, and
that you are okay with using a folder called 'keys' to store these files and
values.

## Installation

- Read the Plausible module installation instructions, being mindful that you
will be doing things a bit differently because of this recipe.
- Create a 'keys' folder in project root & create text files to store them.
  - mkdir keys && cd keys && touch plausible_link.key plausible_domain.key plausible_script.key
- Place the Plausible API values into the relevant files.
- Secure the folder & files as is pertinent to your setup.

## Search

Add the code below to a helper module associated with your site.

You may experience some odd behaviour when implementing this. Can't remember
exactly what happens but to mitigate this, check the following:

- If using redirect.module, enable redirect clean urls in this module.
- Site settings front page is using node id or whatever, not an alias, which
  counteracts an issue caused by the above

```php
use Symfony\Component\HttpFoundation\Request;

/**
* Implements hook_page_attachments().
*
* Add a JS snippet to send search terms to Plausible.
*/
function MYMODULE_page_attachments_alter(array &$attachments) {

  if (!\Drupal::moduleHandler()->moduleExists('plausible')) {
    return;
  }

  // Only track searches from anonymous users - module should handle this, nope.
  if (\Drupal::currentUser()->isAuthenticated()) {
    return;
  }

  $request = \Drupal::requestStack()->getCurrentRequest();
  $search_term = $request->query->get('s');

  # Change the path if it's not 'search'.
  if ($request instanceof Request && $request->getPathInfo() === '/search' && isset($search_term)) {

    // Replace (white)spaces with + as per Plausible docs.
    $term = preg_replace('/\s+/', '+', $search_term);

    $attachments['#attached']['html_head'][] = [
      [
        '#tag' => 'script',
        '#value' => "plausible('Search', { props: { term: $search_term } });",
      ],
      'plausible_tracking_snippet_event_search',
    ];
  }
}
```
