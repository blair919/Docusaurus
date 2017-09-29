---
id: translation
title: Translations & Localization
---

Docusaurus allows for easy translation functionality using Crowdin. Documentation files written in English are uploaded to Crowdin for translation by users within a community. Top-level pages written with English strings can be translated by wrapping any strings you want to translate in a `<translate>` tag. Other titles and labels will also be found and properly translated.

## Docusaurus Translation Configurations

To generate example files for translations with Docusuaurus, run the `examples` script with the command line argument `translations`:

```
npm run examples translations
```

or

```
yarn examples translations
```

This will create the following files:

```
pages/en/help-with-translations.js
languages.js
crowdin.yaml
```

The `pages/en/help-with-translations.js` file includes the same starter help page generated by the `examples` script, but now includes translation tags.

The `languages.js` file tells Docusaurus what languages you want to enable for your site.

The `crowdin.yaml` file is used to configure crowdin integration, and is copied up one level into your docusaurus project repo. If your docusaurus project resides in `/project/website`, then `crowdin.yaml` will be copied to `/project/crowdin.yaml`. 

## Translating Your Existing Docs

Your documentation files do not need to be changed or moved to support translations. They will be uploaded to Crowdin to be translated directly.


## Enabling Translations on Pages

Pages allow you to customize layout and specific content of pages like a custom index page or help page.

Pages with text that you want translated should be placed in `website/pages/en` folder.

Wrap strings you want translated in a `<translate>` tag, and add the following `require` statement to the top of the file:
```jsx
...
const translate = require("../../server/translate.js").translate;
...
<h2>
  <translate>This header will be translated</translate>
</h2>
...
```

You can also include an optional description attribute to give more context to a translator about how to translate the string:
```jsx
<p>
  <translate desc="flower, not verb">Rose</translate>
<p>
```

## Gathering Strings to Translate

The strings within localized Pages must be extracted and provided to Crowdin.

Add the following script to your package.json file:
```json
...
"scripts": {
  "write-translations": "docusaurus-write-translations"
},
...
```

Running the script will generate a `website/i18n/en.json` file containing all the strings that will be translated from English into other languages.

The script will include text from the following places:

- `title` and `sidebar_label` strings in document markdown headers
- category names in `sidebars.json`
- tagline in `siteConfig.js`
- header link `label` strings in `siteConfig.js`
- strings wrapped in the `<translate>` tag in any `.js` files inside `pages`

## How Strings Get Translated

Docusaurus itself does not do any translation from one language to another. Instead, it integrates [Crowdin](https://crowdin.com/) to upload translations and then downloads the appropriately translated files from Crowdin.

## How Docusaurus Uses String Translations

This section provides context about how translations in Docusaurus works.

### Strings

A Docusaurus site has many strings used throughout it that require localization. However, maintaining a list of strings used through out a site can be laborious. Docusaurus simplies this by centralizing strings.

The header navigation, for example can have links to 'Home' or your 'Blog'. This and other strings found in the headers and sidebars of pages are extracted and placed into `i18n/en.json`. When your files are translated, say into Spanish, a `i18n/fr.json` file will be downloaded from Crowdin. Then, when the Spanish pages are generated, Docusaurus will replace the English version of corresponding strings with translated strings from the corresponding localized strings file (e.g. In a Spanish enabled site 'Help' will become 'Ayuda').

### Markdown Files

For documentation files themselves, translated versions of these files are downloaded and then rendered through the proper layout template.

### Other Pages

For other pages, Docusaurus will automatically transform all `<translate>` tags it finds into function calls that return the translated strings from corresponding localized _`locale`_`.json`.

## Crowdin

Create your translation project on [Crowdin](https://www.crowdin.com/). You can use [Crowdin's guides](https://support.crowdin.com/translation-process-overview/) to learn more about the translations work flow.

### Manual File Sync

You can add the following to your `package.json` to manually trigger crowdin. 

```json
"scripts": {
  "crowdin-upload": "export CROWDIN_DOCUSAURUS_PROJECT_ID=$YOUR_CROWDIN_ID; export CROWDIN_DOCUSAURUS_API_KEY=$YOUR_CROWDIN_API_KEY; crowdin-cli --config ../crowdin.yaml upload sources --auto-update -b master",
  "crowdin-download": "export CROWDIN_DOCUSAURUS_PROJECT_ID=$YOUR_CROWDIN_ID; export CROWDIN_DOCUSAURUS_API_KEY=$YOUR_CROWDIN_API_KEY; crowdin-cli --config ../crowdin.yaml download -b master"
},
```

These commands require having an environment variable set with your crowdin project id and api key (`CROWDIN_PROJECT_ID`, `CROWDIN_API_KEY`). You can add them inline like above or add them permanently to your `.bashrc` or `.bash_profile`.

If you run more than one localized Docusaurus project on your computer, you should change the name of the enviroment variables to something unique (`CROWDIN_DOCUSAURUS_PROJECT_ID`, `CROWDIN_DOCUSAURUS_API_KEY`).

### Automated File Sync

To automatically get the translations for your files, update the `circle.yml` file in your project directory to include steps to upload English files to be translated and download translated files using the Crowdin CLI. Here is an example `circle.yml` file:

```yaml
machine:
  node:
    version: 6.10.3
  npm:
    version: 3.10.10

test:
  override:
    - "true"

deployment:
  website:
    branch: master
    commands:
      # configure git user
      - git config --global user.email "test-site-bot@users.noreply.github.com"
      - git config --global user.name "Website Deployment Script"
      - echo "machine github.com login test-site-bot password $GITHUB_TOKEN" > ~/.netrc
      # install Docusaurus and generate file of English strings
      - cd website && npm install && npm run write-translations && cd ..
      # crowdin install
      - sudo apt-get install default-jre
      - wget https://artifacts.crowdin.com/repo/deb/crowdin.deb -O crowdin.deb
      - sudo dpkg -i crowdin.deb
      # translations upload/download
      - crowdin --config crowdin.yaml upload sources --auto-update -b master
      - crowdin --config crowdin.yaml download -b master
      # build and publish website
      - cd website && GIT_USER=test-site-bot npm run publish-gh-pages
```

The `crowdin` command uses the `crowdin.yaml` file generated with the `examples` script. It should be placed in your project directory to configure how and what files are uploaded/downloaded.

Note that in the `crowdin.yaml` file, `CROWDIN_PROJECT_ID` and `CROWDIN_API_KEY` are environment variables set-up in Circle for your Crowdin project. They can be found in your Crowdin project settings.

Now, Circle will help you automatically get translations prior to building your website. The provided `crowdin.yaml` file will copy translated documents into `website/translated_docs/`, and translated versions of the `i18n/en.json` strings file will into `i18n/${language}.json`.

If you wish to use Crowdin on your machine locally, you can install the [Crowdin CLI tool](https://support.crowdin.com/cli-tool/) and run the same commands found in the `circle.yaml` file. The only difference is that you must set `project_identifier` and `api_key` values in the `crowdin.yaml` file since you will not have Circle environment variables set up. 

## Versioned Translations

TODO - This section needs to be fleshed out.

OLD -

If you wish to have translation and versioning for your documentation, add the following section to the end of your `crowdin.yaml` file:

```yaml
  -
    source: '/website/versioned_docs/**/*.md'
    translation: '/website/translated_docs/%locale%/**/%original_file_name%'
    languages_mapping: *anchor
```

Translated, versioned documents will be copied into `website/translated_docs/${language}/${version}/`.
