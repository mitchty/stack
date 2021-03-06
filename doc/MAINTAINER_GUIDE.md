# Maintainer guide

## Pre-release checks

The following should be tested minimally before a release is considered good
to go:

* After GHC 8.0: switch to Debian 8 and CentOS 6.7 Vagrant boxes for building Stack
  binaries (to match GHC bindists) and drop support for Debian 7.
* Ensure `release` and `stable` branches merged to `master`
* Integration tests pass on a representative sample of platforms: `stack install
  --pedantic && stack test --pedantic --flag stack:integration-tests` . The actual
  release script will perform a more thorough test for every platform/variant
  prior to uploading, so this is just a pre-check
* Ensure `stack haddock` works
* Stack builds with `stack-7.8.yaml` (Travis CI now does this)
* stack can build the wai repo
* Running `stack build` a second time on either stack or wai is a no-op
* Build something that depends on `happy` (suggestion: `hlint`), since `happy`
  has special logic for moving around the `dist` directory
* Check for any important changes that missed getting an entry in Changelog
* In release candidate branch:
    * Bump the version number (to even second-to-last component) in the .cabal
      file
    * Update the ChangeLog:
        * Rename the "unreleased changes" section to the new version
        * Check for any entries that snuck into the previous version's changes
          due to merges
* In master branch:
    * Bump version to next odd second-to-last component
    * Add new "unreleased changes" section in changelog
    * Bump to use latest LTS version
* Review documentation for any changes that need to be made
    * Search for old Stack version, unstable stack version, and the next
      "obvious" version in sequence (if doing a non-obvious jump) and replace
      with new version
    * Look for any links to "latest" documentation, replace with version tag
    * Ensure all inter-doc links use `.html` extension (not `.md`)
    * Ensure all documentation pages listed in `doc/index.rst`
* Check that any new Linux distribution versions added to
  `etc/scripts/release.hs` and `etc/scripts/vagrant-releases.sh`
* Check that no new entries need to be added to
  [releases.yaml](https://github.com/fpco/stackage-content/blob/master/stack/releases.yaml),
  [install_and_upgrade.md](https://github.com/commercialhaskell/stack/blob/master/doc/install_and_upgrade.md),
  and
  [README.md](https://github.com/commercialhaskell/stack/blob/master/README.md)

## Release process

See
[stack-release-script's README](https://github.com/commercialhaskell/stack/blob/master/etc/scripts/README.md#prerequisites)
for requirements to perform the release, and more details about the tool.

* Create a
  [new draft Github release](https://github.com/commercialhaskell/stack/releases/new)
  with tag `vX.Y.Z` (where X.Y.Z is the stack package's version)

* On each machine you'll be releasing from, set environment variables:
  `GITHUB_AUTHORIZATION_TOKEN`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`,
  `AWS_DEFAULT_REGION`

* On a machine with Vagrant installed:
    * Run `etc/scripts/vagrant-releases.sh`

* On Mac OS X:
    * Run `etc/scripts/osx-release.sh`

* On Windows:
    * Ensure your working tree is in `C:\stack` (or a similarly short path)
    * Run `etc\scripts\windows-releases.bat`
    * Release Windows installers. See
      [stack-installer README](https://github.com/borsboom/stack-installer#readme)

* Push signed Git tag, matching Github release tag name, e.g.: `git tag -u
  9BEFB442 vX.Y.Z && git push origin vX.Y.Z`

* Reset the `release` branch to the released commit, e.g.: `git checkout release
  && git merge --ff-only vX.Y.Z && git push origin release`

* Update the `stable` branch similarly

* Publish Github release

* Upload package to Hackage: `stack upload . --pvp-bounds=both`

* Upload haddocks to Hackage: `etc/scripts/upload-haddocks.sh`

* On a machine with Vagrant installed:
    * Run `etc/scripts/vagrant-distros.sh`

* Edit
  [stack-setup-2.yaml](https://github.com/fpco/stackage-content/blob/master/stack/stack-setup-2.yaml),
  and add the new linux64 stack bindist

* Activate version for new release tag on
  [readthedocs.org](https://readthedocs.org/projects/stack/versions/), and
  ensure that stable documentation has updated

* Submit a PR for the
  [haskell-stack Homebrew formula](https://github.com/Homebrew/homebrew/blob/master/Library/Formula/haskell-stack.rb)
      * Be sure to update the SHA sum
      * The commit message should just be `haskell-stack <VERSION>`

* Update in Arch Linux's
  [haskell-stack.git](ssh+git://aur@aur.archlinux.org/haskell-stack.git):
  `PKGBUILD` and `.SRCINFO`
      * Be sure to reset `pkgrel` in both files, and update the SHA1 sum

* Keep an eye on the
  [Hackage matrix builder](http://matrix.hackage.haskell.org/package/stack)

* Announce to haskell-cafe@haskell.org, haskell-stack@googlegroups.com,
  commercialhaskell@googlegroups.com mailing lists

* Merge any changes made in the RC/release/stable branches to master.
