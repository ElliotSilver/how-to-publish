The steps below describe how to publish an implementation guide using IG Publisher. Perform these steps for each release of each IG you want to publish. You should have already [set up](./setup.html) your publication environment, and will need to be able to upload content to your website.

### Prepare your Implementation Guide

1. Clone your IG repository to your local system.

    ```sh
    $ cd ~/src
    $ git clone git@github.com:ExampleOrg/my-ig.git
    $ cd my-ig
    ```

   (If the repo is already on your system, ensure all changes have been committed.)

1. Create a release branch of your IG repo and switch to it

   ```sh
   $ git checkout -b v1.0.0
   ```

1. Review your IG's ImplementationGuide resource (or `sushi-config.yaml`) and update as needed.

   * IG canonical aligns with your main publication location (e.g. `http://example.org/ig/myig`)
   * IG `status` (see [Publication Status](http://hl7.org/fhir/ValueSet/publication-status)), `releaseLabel` (see [IG Parameters](http://hl7.org/fhir/tools/CodeSystem/ig-parameters)), `date` (if explicitly set), and `version` elements are all correct for your published IG.
   * Dependencies (including the template specified in `igi.ini`) are up to date and set to specific versions.
  
1. Ensure you are starting from a known state.

   * Delete contents of your FHIR cache directory `~/.fhir`, except for `fhir-settings.json` if it exists
   * Delete `fsh-generated`, `input-cache`, `output`, `temp`, and `template` directories from the IG project
   * Use the most current releases of IG tooling:

      ```sh
      $ ./_updatePublisher.sh
      $ npm update -g fsh-sushi
      ```

1. Create (or update) `publication-request.json` in your IG root directory, filling it out appropriately for your IG:

   ```json
   {
      "package-id" : "org.example.myig",
      "version" : "1.0.0",
      "path" : "http://example.org/ig/myig/1.0.0",
      "mode" : "milestone",
      "status" : "release",
      "sequence" : "Releases",
      "desc" : "Initial release as presented at FHIR North.",
      "first" : true,
      "title" : "How to Publish a FHIR Implementation Guide",
      "ci-build" : "http://build.fhir.org/ig/ExampleOrg/myig",
      "category" : "Infrastructure",
      "introduction" : "An instructional guide showing how to use the HL7 IG Publisher's -go-publish feature to publish an implementation guide to a website."
   }
   ```

   This file is used to populate the publication feeds. Several elements are only for the first publication of an IG. See [IG Publication Request Documentation](https://confluence.hl7.org/display/FHIR/IG+Publication+Request+Documentation) for complete details, including some additional elements.

   | Attribute | First&nbsp; Only&nbsp; | Description |
   |---|:---:|---|
   | `package-id` | | The IG package id, matching the value from `sushi-config.yaml` or your ImplementationGuide resource |
   | `version`    | | The IG version, matching the value from `sushi-config.yaml` or your ImplementationGuide resource |
   | `path`       | | The permanent full URL at which the IG will be available. This must be of the form `[canonical]/[label]`, where `[canonical]` is the canonical URL of your IG, and `[label]` is the version or another label for this release (e.g. "STU1"). |
   | `mode`       | | {::nomarkdown}One of:<ul><li><code class=" highlighter-rouge language-plaintext">working</code> (publish at <code class=" highlighter-rouge language-plaintext">path</code>, but don't update which publication is "current");</li><li><code class=" highlighter-rouge language-plaintext">milestone</code> (publish at <code class=" highlighter-rouge language-plaintext">path</code> and at the canonical, making this the "current" release);</li><li><code class=" highlighter-rouge language-plaintext">technical-correction</code> (replace the content at <code class=" highlighter-rouge language-plaintext">path</code>; if the content at <code class=" highlighter-rouge language-plaintext">path</code> is the "current" release, replace the content at the canonical also)</ul>{:/nomarkdown} |
   | `status`     | | The IG release status, typically `release`, `trial-use`, `draft`, or `normative` (or [other options](https://confluence.hl7.org/display/FHIR/IG+Publication+Request+Documentation)); often matches the IG `releaselabel`. |
   | `sequence`   | | The release sequence, e.g. "Release 1," that this publication is part of; or, "Publication," "Release," or similar, if not using release sequences. This determines grouping on the history page. |
   | `desc`<br>`descmd` | | The description of the release shown on the IG history page, as plain text (desc), or as markdown (descmd). Markdown is read from the file specified using "@[filename.md]". |
   | `changes`    | | An optional relative link to a "Change Log" page in the IG |
   | `first`      | | Set to `true` on first publication of this IG; `false` or omit otherwise |
   | `title`      | &#10003; | The human readable name of the IG |
   | `ci-build`   | &#10003; | The location of the continuous integration build of the IG; often under [`http://build.fhir.org/ig/`](http://build.fhir.org/ig/) |
   | `category`   | &#10003; | A category for the guide; see the categories on the [FHIR registry](http://fhir.org/guides/registry/). |
   | `introduction` | &#10003; | A human readable description of the intent of the IG; shown on the IG history page. |
   {:.table-striped}

1. Ensure you can do a clean build of the IG.

   ```sh
   $ ./_genonce.sh
   ```

   Check the build command-line output and `output/qa.html` for errors and warnings, and correct if needed.

### Do a Publication Run and Upload

1. If you haven't already, create a branch of the `ig-registry` repo.

   ```sh
   $ cd ~/src/ig-registry
   $ git checkout -b example-org-igs
   ```

1. Switch to the the `publication` directory. Ensure you have a current copy of the IG Publisher.

   ```sh
   $ cd ~/src/publication
   $ curl -L https://github.com/HL7/fhir-ig-publisher/releases/latest/download/publisher.jar -o ~/src/publisher.jar
   ```

1. Run the IG Publisher in "publish" mode:

   ```sh
   $ cd ~/src/publication
   $ java -jar ../publisher.jar -go-publish \
      -source    /home/username/src/my-ig \
      -web       /home/username/src/publication/webroot \
      -history   /home/username/src/ig-history \
      -registry  /home/username/src/ig-registry/fhir-ig-list.json \
      -temp      /home/username/src/publication/temp \
      -templates /home/username/src/publication/templates
   ```

   Files or directories must either be under the `publication` directory or specified using an absolute path.

   * `source` - the IG to publish
   * `web` - the base staging directory to put the generated files into
   * `history` - the source directory for the history templates
   * `registry` - the IG registry list file to update with information about this publication
   * `temp` - a temporary storage location
   * `templates` - your customization files for the generated history, etc.

   Running `-go-publish` may build your IG multiple times as part of the process.

1. Run the Jekyll web server and review the generated IG.

   ```sh
   $ jekyll server -s webroot
   ```

   The IG is accessible at <http://127.0.0.1:4000>.

1. Upload the contents of the `webroot` directory to your website (<http://example.org/ig>), and review the publication.

You have now published your IG to your website.

### Commit changes

Once you are confident that the website is updated correctly, commit your changes.

1. The publication process updates the list of publications known to the general FHIR tooling. In the `ig-registry` repo:

   1. Find the entry for your IG in `fhir-ig-list.json`, and update the missing information (shown as `"??"`). Fill in `description`, `authority` (your organization, or omit for personal sites), and `country` (two letter country codes).

   1. Commit the changes, push the changes upstream, and make a pull request to merge the changes back to the `FHIR/ig-registry` repo. Once these changes are accepted, FHIR ecosystem tooling will be aware of your IG publication.

1. Typically, you'll now want to commit two versions of your implementation guide to your repo.

   1. Commit the as-published version of your IG, and tag it.

      Tagging your release is an important best practice that will enable you to come back to this version should you ever need to republish, or issue an update.

   1. Commit a version of your IG ready for the next round of development, and commit. To update your IG for further development,

      * In your ImplementationGuide resource (or `sushi-config.yaml`), update the `status`, `releaseLabel`, and `version` elements as appropriate for the CI build; and revert dependencies to use the `current` version, if desired (also, the template version in `ig.ini`).
      * Remove first-time elements from `publication-request.json`, and update other elements in preparation for the next release.
      * Commit the changes to your implementation guide repo.

1. If you are storing the publication website in a repository, you may wish to commit the contents of the `publication` directory.

### Directory structure

Assuming you were publishing your IG with a release label of "1.0.0" as the current publication, then, at the end of this process, you should have a directory structure like:

```text
📁 .
└─ 📁 myig (or your IG)
   └─ ...
└─ 📁 fhir-web-templates (can be deleted)
   └─ ...
└─ 📁 ig-history
   └─ ...
└─ 📁 ig-registry
   └─ ...
└─ 📁 publication
   └─ 📁 webroot
      └─ package-feed.xml
      └─ package-registry.json
      └─ publication-feed.xml
      └─ publish-setup.json
      └─ 📁 myig
         └─ (contents of the published "current" IG)
         └─ 📁 1.0.0
            └─ (contents of the published "1.0.0" IG)
   └─ 📁 temp
      └─ ...
   └─ 📁 templates
      └─ header.template
      └─ index.template
      └─ postamble.template
      └─ preamble.template
      └─ searchform.template.html
```

The contents of the `myig` and `1.0.0` directories will be (almost) the same.

To see the impact of additional publications, you can update the version of your IG in the ImplementationGuide resource (or `sushi-config.yaml`), and in `publication-request.json` (and the other changes from the first section of this page), and run the `-go-publish` process again.
