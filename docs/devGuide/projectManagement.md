<variable name="title">Project management</variable>
<frontmatter>
  title: "{{ title }}"
  layout: devGuide
  pageNav: default
</frontmatter>

# {{ title }}

<div class="lead">

This page contains information about project management tasks. The target audience is senior developers (and above).
</div>

## Managing PRs

For general best practices, refer to the guide [_Working with PRs_ @SE-EDU](https://se-education.org/guides/guidelines/PRs.html).

### Approval

* **If the PR is very simple** (e.g. correcting a simple typo), one approval is sufficient for merging. Other PRs need two approvals.

* **Once the first approval is given** from a developer, assign a version milestone to the PR (this is usually the next version, unless there is a justification for delaying the merging to future versions).

  <box type="info" seamless>

  **Note:** Should there be a subsequent rejection by other reviewers (or an error is spotted), the version milestone is removed immediately. The version milestone should only be restored once the new approvals come in.
  </box>

* **Once the second approval is given** from a ==senior developer== (or there are >= 3 approvals given from anyone), the PR can be merged immediately.<br>
  If there is no second approval yet, wait for a day before merging the PR without the second approval.

  <box type="info" seamless>

  This is to allow other developers the chance to review the PR, and delay the merge if there's any significant problems. Not everyone is available all of the time, so please be patient.
  </box>

### Merging

1. **Re-run the Travis build** for the current PR's `continuous-integration/travis-ci/pr`.

   <box type="info" seamless>

   **Rationale:** This is because we do not ask PR authors to rebase their PRs, but the master branch may have since been updated. It is possible for tests to pass on their branch, only to fail when integrating the changes with the master

   This is possible even if GitHub _doesn't_ complain about any merge conflict!
   </box>
1. **Merge the PR**.
   * Refer the guide [_Working with PRs_ @SE-EDU](https://se-education.org/guides/guidelines/PRs.html) which merge strategy to use (i.e., squash, rebase, merge commit)

   * Format for the merge/squashed commit title: `PR_TITLE (#PR_NUMBER)`<br>
    `Add built-in support for light themes from bootswatch (#745)`

   * For non-trivial PRs, ensure that there is a sensible commit message accompanied by it. Both the title and the body should follow the [_Git conventions_ @SE-EDU](https://se-education.org/guides/conventions/git.html).

   * Before confirming the merge, do ensure that no other PRs have been merged to master _since the time you started drafting the merging commit's title and message_. Otherwise, you may face a glitch where GitHub merges your PR without squashing. (Reference: [MarkBind#1160](https://github.com/MarkBind/markbind/pull/1160))

1. **Set a milestone** to the PR.

   <box type="info" seamless>

   **Reason:** We may have missed it during the "Approval" stage, so please add the version milestone if it is missing, so that the drafting of the release notes during the release process will be easier.
   </box>

## Doing a Release

<box type="important" seamless>

  **Attention new maintainers!** Ensure that:

* You have the rights to push to master branch on [MarkBind's repository](https://github.com/MarkBind/markbind), and also to make new releases.
* You have the rights on [npm](https://www.npmjs.com/) to make a new release.<br>
  You need to login to npm on your terminal with `npm adduser` first, before you can publish packages to npm.
</box>

### Building Vue-Strap

1. **Navigate to the root** directory for vue-strap
1. **Update dependencies** by running `npm install`
1. **Build the assets** by executing `npm run build` after which you should see changes in `vue-strap.min.js`.

   <box type="info" seamless>

   Building vue-strap is optional if there's no changes since the last release of MarkBind. If there are no changes to the three files mentioned above, skip to the "Building MarkBind" section.
   </box>
   <box type="info" seamless>

   Do take a peek at the diff for `vue-strap.min.js` to see if there's any strange changes (e.g. the version of jQuery changes even though no one upgraded it).
   </box>

1. **Commit** the new changes (`git commit -am 'Update dist'`)

1. **Tag** the new commit with the new version number, whereby `XYZ` is the incremented number of the previous release.<br>
   e.g., `git tag v2.0.1-markbind.XYZ`

   <box type="important" seamless>

   The tag used is a lightweight tag. _Don't_ use an annotated tag.
   </box>

1. **Push** everything to the main vue-strap repository (replace `XYZ` with version number).

   ```
   $ git push upstream master
   $ git push upstream v2.0.1-markbind.XYZ
   ```

### Building MarkBind

1. **Navigate to the root** directory for MarkBind
1. **Update dependencies** by running `npm install`
1. **If we did a new vue-strap release**:

   Copy `vue-strap.min.js` from the vue-strap repository to the main asset folder, and to each of the test site's `expected/` folder.

   ```sh
   # copy to main asset folder
   $ cp <VUE_STRAP_REPO>/dist/vue-strap.min.js <MARKBIND_REPO>/asset/js/vue-strap.mins.js
   # for each test site's expected folder:
   $ cp <VUE_STRAP_REPO>/dist/vue-strap.min.js <MARKBIND_REPO>/test/functional/<TEST_SITE_NAME>/expected/markbind/js/vue-strap.min.js
   ```

   Commit the new vue-strap assets (replace `XYZ` with version number).

   ```sh
   $ git commit -m 'Update vue-strap version to v2.0.1-markbind.XYZ'
   ```

   <box type="info" seamless>

   We uniquely do this for each MarkBind release (rather than spontaneously update the vue-strap files for each affected PR), in order to reduce unnecessary merge conflicts. It also makes it easier for the maintainers to vet the changes.
   </box>

1. **Increment the version number** by running `npm version`. Which to increment (`patch`, `minor` or `major`) depends on what PRs are merged for the new version, which means you must know beforehand about the changes.

   * If there are no significant changes, a `patch` is sufficient: `npm version patch`
   * If there are significant changes (e.g. breaking changes, new release), a `minor` release is needed:
     1. Run `npm version minor` as per normal.
     1. Update the version number inside `src/lib/markbind/package-lock.json` and `src/lib/markbind/package.json` manually. This is because `npm version` will not automatically update the numbers from the outside. (Note: You do not have to commit these changes immediately, as we also have to rebuild the test files in the next step anyway.)

   * We rarely do `major` releases, but if necessary, the steps are the same as the `minor` release (just change `minor` to `major`).

1. **Rebuild the test files**.
   * Unix: `npm run updatetest`
   * Windows: `npm run updatetestwin`


   When rebuilding the expected test files, ensure that **only** the version number is updated. For example, this is correct:

   ```diff
   diff --git a/test/functional/test_site/expected/bugs/index.html b/test/functional/test_site/expected/bugs/index.html
   index 779f279..bb3c602 100644
   --- a/test/functional/test_site/expected/bugs/index.html
   +++ b/test/functional/test_site/expected/bugs/index.html
   @@ -4,7 +4,7 @@
        <meta name="default-head-top">
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
   -    <meta name="generator" content="MarkBind 1.20.0">
   +    <meta name="generator" content="MarkBind 1.21.0">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>Open Bugs</title>
        <link rel="stylesheet" href="../markbind/css/bootstrap.min.css">
   ```

   However, if there are any extra lines changed, that means that someone screwed up the functional tests, and needs to be fixed accordingly!

1. **Combine** the changes that you made in the last two steps with the version commit generated by `npm version`. To do so, amend the version commit and version tag by doing the following (change `vA.B.C` to the new version's string accordingly).

   ```sh {.no-line-numbers}
   $ git commit -a --amend --reuse-message vA.B.C
   $ git tag --force vA.B.C            # to reassign the tag to the newly amended commit
   ```

1. **Push** the new commit and version tag generated by `npm` (change `vA.B.C` to the new version's string accordingly).

   ```sh {.no-line-numbers}
   $ git push upstream master
   $ git push upstream vA.B.C
   ```

1. **Publish** by running `npm publish`. You should receive a notification by `npm` that the publish is successful.


1. **Smoke test** the new version by installing the new version (run `npm i -g markbind-cli@A.B.C`) and playing around with it a bit to ensure that it works.

   <box type="info" seamless>

   If you used `npm link` for your project, ensure that you ==try the new release on another platform== that doesn't use `npm link`, so that we can be sure the end-users can install and use the new version.
   </box>

1. **Verify the website** at markbind.org has been updated.

   The deployment is automated. However, if it doesn't deploy properly for the release, do the following:

   ```sh {.no-line-numbers}
   $ cd docs
   $ markbind build -s ug-site.json
   $ markbind deploy -s ug-site.json
   ```

1. **Update milestones**. Close the [milestone on GitHub](https://github.com/MarkBind/markbind/milestones) and all issues in that milestone.

   Also create a new milestone for the next iterations. Recommended to have at least two upcoming milestones opened.

1. **Release on GitHub**.

   a. Go to the Markbind release page at https://github.com/MarkBind/markbind/releases.

   b. Click "Draft a new release".

   c. For the tag version, enter `vA.B.C`. (The newly released version should be recognized by GitHub, with an "Existing Tag" indicator, otherwise, ensure that you have pushed the version commit and tag in step 4).

   d. For the release title, leave it blank.

   e. For the main body, use the following template:

      ```markdown
      # markbind-cli

      <!-- List out each of the PR for this version in the following format: -->
      <!-- #ISSUE_NUMBER ISSUE_TITLE (#PR_NUMBER) -->

      ## User Facing Changes

      ### Breaking Changes

      <!-- Any feature that is made obsolete -->

      > Also give a brief explanation note about:
      >   - what was the old feature that was made obsolete
      >   - any replacement feature (if any), and
      >   - how the author should modify his website to migrate from the old feature to the replacement feature (if possible).

      ### Features

      <!-- Features enable users (authors/readers) to do something new. -->

      ### Enhancements

      <!-- Enhances any existing features -->

      ### Fixes

      <!-- Fixes correct a programming error/assumption. -->

      ### Documentation

      <!-- Pure changes to the documentation, such as typo, restructuring, etc -->

      ## Other Changes

      ### Code Quality

      <!-- Refactoring, etc. -->

      ### DevOps Changes

      <!-- Tooling, etc. -->

      ### Dependencies

      <!-- Replace `OLD` with the previous version and `NEW` with the current version -->

      [MarkBind/vue-strap](https://github.com/MarkBind/vue-strap): v2.0.1-markbind.OLD → v2.0.1-markbind.NEW

      ### Miscellaneous

      <!-- Any other changes -->

      ```

      Use the list of PRs merged in the milestone to write the release notes. You may omit any sections that do not have a single PR under it.

      An example of a release note draft (taken from v1.18.0):

      ```
      # markbind-cli

      ### Breaking Changes

      #653 Disable decamelize for anchor ID generation (#667, MarkBind/vue-strap#95)

      > Headings with PascalCase wordings now generate a different anchor ID
      in order to be more compatible with anchor IDs generated by Github
      Flavored Markdown.
      >
      > For example, if we have the following heading:
      >
      > ```markdown
      > # MarkBind docs
      > ```
      >
      > Old anchor id:
      >
      > ```
      > mark-bind-docs
      > ```
      >
      > **New** anchor id:
      >
      > ```
      > markbind-docs
      > ```
      >

      ### Features

      #457 Add `deploy -t/--travis` to deploy via Travis (#649)
      #470 Support custom MarkBind plugins (#474)
      #642 Support specifying include variables inline (#681)

      ### Enhancements

      #369 Seamless panels: omit caret if not expandable (MarkBind/vue-strap#96)
      #657 Add temporary styles to prevent FOUC (#664)

      ### Fixes

      #651 Exit MarkBind with non-zero exit code on fatal error (#679)

      ### Documentation

      Restructure user docs on MarkBind syntax (#668)
      Add documentation for badges (#686)
      Fix filename capitalization for syntax documentation (#685)

      ### Code Quality

      test: Add diff printing for easier debugging (#632)

      ### Dependencies

      [MarkBind/vue-strap](https://github.com/MarkBind/vue-strap): v2.0.1-markbind.20 → v2.0.1-markbind.21
      ```

   f. Click "Publish release".

1. **Announce the new release** on our Slack channel (replace `A.B.C` with the new version). Congrats!

   >Published: `npm i -g markbind-cli@A.B.C`