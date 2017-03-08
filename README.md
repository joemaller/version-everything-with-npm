
# Version everything with npm
##### Version: 1.2.2

Versioning internal projects is often an afterthought -- at best. Too often updating version numbers is completely forgotten.

Usually this is no big deal, the Git history is good enough. But on longer-running projects, it becomes a problem.

Sure, saying "increment the version string, save the file, commit and push" sounds trivial, but that process is still onerous enough and removed from our workflows that it's easy to blow off completely.


### Enter npm 

Npm includes the wonderful [`npm version`][npm version] command. It's main purpose is to update the version number in a `package.json` file. As a bonus, the command commits the updated `package.json` file *and tags the repository.*

Several `version` sub-commands auto-increment the package version according to the [SemVer specification][semver]. For example, starting from version **1.0.0**, the three primary commands, `patch`, `minor` and `major` do this:

* `npm version patch` sets version to v1.0.1
* `npm version minor` sets version to v1.1.0
* `npm version major` sets version to v2.0.0

The command can also set explicit versions or increment pre-releases (1.2.3-x) with the `prepatch`, `preminor`, `premajor` and `prerelease` subcommands. All of these commands also help reduce some uncertainty about SemVer; clear action verbs are very easy to understand.

But of course, not every project is an npm package. Some, like WordPress plugins and themes store their own version strings in separate files, written in other languages. 

Not a problem. If the version strings can be matched with a regex or found in a structured-data file, they can almost certainly be updated with a little scripting.

### npm's killer feature

The ability to [run arbitrary scripts][npm scripts] and simple shell commands from `package.json` is an incredibly useful feature which is too often overlooked. 

npm runs scripts in a rich environment. Besides adding the local `node_modules/.bin` to `$PATH`, all local configuration values and `package.json` fields are exposed. Variables are prefixed with [`$npm_config_`][config vars] and [`$npm_package_`][package.json vars] respectively. (This [Stack Overflow answer][so] shows how to dump everything to `stdout` for inspection.)

A command attached to `version` will run immediately after npm updates `package.json` but before committing changes to Git. This is when we'll update our other version-containing files.

Bumping a version number should be a relatively simple task. While a small script could be added to the project, I prefer keeping project-unrelated files to a minimum. 

### You shouldn't. I already did.

JSON files can't contain executable code or even multi-line strings, but why let that stop us. It's completely possible to embed a script with a simple transformation: Each line of the script becomes a string and the script is just an array of those line-strings. To run the script, send the joined array to `eval`. This method feels a little naughty, but limitations can be inspiring.

Terminal commands aren't necessarily portable across platforms, but these scripts will work everywhere without modification. 

### Use the best tool

JavaScript wasn't designed to allow the semi-cryptic one-line scripts that [Ruby][] and [Perl][] can do. But since we're presumably already loading external JavaScript packages with npm, a [nearly infinite toolset][npm] is available to us. The embedded version script makes use of two packages; [shelljs][] for its sed clone, and [jsonfile][] for simple JSON updates. 

The `version_files` field in `package.json` contains a list of files to be versioned. While it might make sense to overload the [main][] field, the additional `version_file` field will be easier to reason about later on. 

Here's the embedded script, it loops over the `version_files` array, first attempting to update each as JSON then falling back to string replacement.

```javascript
const sed = require('shelljs').sed;
const jsonfile = require('jsonfile');
const pkg = require('./package.json');
pkg.version_files.forEach(f => {
  jsonfile.readFile(f, (err, data) => {
    if (err) {
     sed('-i', /^([# ]*Version: ).*/, `$1${ pkg.version }`, f);
    } else  {
      data.version = pkg.version;
      jsonfile.writeFileSync(f, data, {spaces: 2});
    }
  });
})
```


Putting it all together, here's the `package.json` file with the script embedded. The project version can now be synchronized across two example WordPress files, a `manifest.json` file and the project's `README.md` using simple, clean npm commands.

```json
{
  "name": "version-everything",
  "version": "1.1.1",
  "description": "Version everything with npm",
  "version_files": [
    "README.md",
    "example_wordpress_plugin.php",
    "example_wordpress_theme.css",
    "manifest.json"
  ],
  "scripts": {
    "version": "node -e \"eval(require('./package.json').version_script_src.join(''))\" && git add -u"
  },
  "author": "joe maller <joe@joemaller.com>",
  "license": "MIT",
  "private": true,
  "devDependencies": {
    "jsonfile": "^2.4.0",
    "shelljs": "^0.7.4"
  },
  "version_script_src": [
    "const sed = require('shelljs').sed;",
    "const jsonfile = require('jsonfile');",
    "const pkg = require('./package.json');",
    "pkg.version_files.forEach(f => {",
    "  jsonfile.readFile(f, (err, data) => {",
    "    if (err) {",
    "     sed('-i', /^([# ]*Version: ).*/, `$1${ pkg.version }`, f);",
    "    } else  {",
    "      data.version = pkg.version;",
    "      jsonfile.writeFileSync(f, data, {spaces: 2});",
    "    }",
    "  });",
    "});"
  ]
}

```


### Source repository

An example repository with accompanying files is on GitHub here:

* https://github.com/joemaller/version-everything-with-npm

### Notes

* *npm will not force-overwrite existing git tags with new commits. This is a good thing. Existing tags can be removed with `git tag -d tagname`.*

* Two methods of dumping the environment are included in the repository, try `npm run dump-vars` and `npm run env` to see what's available.







[so]: http://stackoverflow.com/a/19381235/503463
[npm version]: https://docs.npmjs.com/cli/version
[main]: https://docs.npmjs.com/files/package.json#main
[sed]: http://www.grymoire.com/Unix/Sed.html
[semver]: http://semver.org/

[npm scripts]: https://docs.npmjs.com/misc/scripts
[package.json vars]: https://docs.npmjs.com/misc/scripts#packagejson-vars
[config vars]: https://docs.npmjs.com/misc/scripts#configuration
[replace]: https://www.npmjs.com/package/replace

[glob]: https://github.com/isaacs/node-glob
[perl]: http://www.math.harvard.edu/computing/perl/oneliners.txt
[ruby]: http://reference.jumpingmonkey.org/programming_languages/ruby/ruby-one-liners.html
[npm]: https://www.npmjs.com/
[jsonfile]: https://www.npmjs.com/package/jsonfile
[shelljs]: https://www.npmjs.com/package/shelljs
[compose]: https://www.keithcirkel.co.uk/how-to-use-npm-as-a-build-tool/#running-multiple-tasks