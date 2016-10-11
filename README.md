
# Version everything with npm

Versioning internal projects is often an afterthought -- at best. Too often updating version numbers is completely forgotten.

Usually it's not a big deal, the git history is good enough. But on a few longer-running projects it's become a problem.

Sure saying "increment the version string, save the file, commit and push" sounds trivial, but that process still onerous enough and removed from our workflows that it's easy to blow off completely.


### Enter npm 

Npm includes the wonderful [`npm version`][npm version] command. It's main purpose is to update the version number in `package.json`. As a bonus, if the package is also  a Git repository, the command will commit the updated `package.json` file *and tag the repository.*

Several `version` sub-commands auto-increment package versions according to the [SemVer specification][semver]. Starting from a v1.0.0, the three primary commands produce the  following:

* `npm version patch` sets version to v1.0.1
* `npm version minor` sets version to v1.1.0
* `npm version major` sets version to v2.0.0

The command can also set explicit versions or increment pre-releases (1.2.3-x) with the `prepatch`, `preminor`, `premajor` and `prerelease` subcommands. These commands also eliminate some uncertainty about SemVer, clear action verbs are very easy to understand.

But of course, not every project is an npm package. Some, like WordPress plugins and themes store their own version strings in separate files written in other languages. 

Not a problem. If the version strings can be matched with a regex, then they can almost certainly be updated with a little scripting.

### npm's killer feature

The ability to [run arbitrary scripts][npm scripts] and simple shell commands from `package.json` is an incredibly useful feature which is too often overlooked. 

npm runs scripts in a rich environment. Besides adding the local `node_modules/.bin` to `$PATH`, configuration and local `package.json` fields are also exposed. Variables are prefixed with [`$npm_config_`][config vars] and [`$npm_package_`][package.json vars] respectively. This [Stack Overflow answer][so] shows how to dump everything to `stdout` for inspection. 

A command attached to `version` will run immediatelyafter npm updates `package.json` but before committing changes to Git. This is when we'll update our other version-containing files.


### New, meet old.

We could add a small script to the project, but there's no need. The 40-year old command line tool [sed][] does everything we need and more. It's ancient, proven and kind of magical. 

But first, instead of hard-coding the version-containing file into the command, let's store it in a `package.json` field. While it might make sense to overload the [main][] field, a new field, `version_file`, will be easier to reason about later on.

Here's a simple `package.json` for synchronizing the package version into a php file:

```json
{
  "name": "version-everything",
  "version": "1.0.0",
  "description": "Version everything with npm",
  "version_file": "version_everything.php",
  "scripts": {
    "version": "sed -i '' \"s/^Version:.*/Version: $npm_package_version/g\" $npm_package_version_file && git add -u"
  },
  "author": "joe maller <joe@joemaller.com>",
  "license": "MIT",
  "private": true
}
```


The sed pattern is simply `^Version:.*` which is replaced with `Version: $npm_package_version` (where `$npm_package_version` is something like `1.0.3`). That's called against `$npm_package_version_file` which is `version_everything.php`. The `-i` flag edits files in-place. Finally, all changed files are staged with `git add -u` to prevent accidental additions. 



### Notes

* *There is a slight difference between the macos and Linux (GNU) sed commands. On Macs, the in-place flag should have a space after it `sed -i ''` but on Linux, the space and quotes can be omitted, `sed -i` is enough.*

* *npm will not force-overwrite existing git tags with new commits. This is a good thing. Existing tags can be removed with `git tag -d tagname`.*

 










[so]: http://stackoverflow.com/a/19381235/503463
[npm version]: https://docs.npmjs.com/cli/version
[main]: https://docs.npmjs.com/files/package.json#main
[sed]: http://www.grymoire.com/Unix/Sed.html
[semver]: http://semver.org/

[npm scripts]: https://docs.npmjs.com/misc/scripts
[package.json vars]: https://docs.npmjs.com/misc/scripts#packagejson-vars
[config vars]: https://docs.npmjs.com/misc/scripts#configuration