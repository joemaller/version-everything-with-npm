
# Version everything with npm

Versioning internal projects is often an afterthought -- at best. Too often project versioning is completely blown off.

Usually it's not a big deal, the git history is good enough. But on a few longer-running projects it's become a problem.

Sure saying "increment the version string, save the file, commit and push" sounds trivial, but that process still onerous enough and removed from our workflows to foster neglect.


### Enter npm 

Npm includes the wonderful [`npm version`][npm version] command. It's main purpose is to update a package version as recorded in `package.json`. As a bonus, if the package is also  a Git repository, the command will commit the updated `package.json` file *and tag the repository.*

Several `version` sub-commands auto-increment package versions according to the SemVer specification. Starting from a v1.0.0, the three primary commands produce the  following:

* `npm version patch` sets version to v1.0.1
* `npm version minor` sets version to v1.1.0
* `npm version major` sets version to v2.0.0

The command can also set an explicit version or increment pre-releases (1.2.3-x) with the `prepatch`, `preminor`, `premajor` and `prerelease` subcommands.

But of course, not every project is an npm package. Some, like WordPress plugins and themes store their own version strings in separate files written in other languages. 

No problem. If the version strings can be matched with a regex, then they can almost certainly be updated with a little scripting.

### npm's killer feature

The ability to [run arbitrary scripts][npm scripts] and simple shell commands from package.json is an incredibly useful feature which is too often overlooked. 

npm runs scripts in a rich environment. Besides adding the local `node_modules/.bin` to `$PATH`, configuration and local package.json fields are also exposed. Variables are prefixed with [`$npm_config_`][config vars] and [`$npm_package_`][package.json vars] respectively. This [Stack Overflow answer][so] shows how to dump everything to `stdout` for inspection. 

To version files, I will be attaching a command to `version`. 


### Vars that matter


Rather than hard-coding the 

I considered overloading the [main][] attribute of `package.json` to define the file I'm using for versioning, but opted for a unique keypair instead. On one hand, it would make complete sense; this project isn't a true npm module, so the main file should point to whatever file is actually important. But by the same thinking, a new key will be easier to reason about later on. In the end, the file to be versioned is stored in `version_file` and can be referenced by the env var `$npm_package_version_file`.



We could add a small script to the project, but there's no need. A 40-year old command line tool does everything we need and more. Sed is short for "stream editor", it's ancient, proven and kind of magical. 

    sed -Ei '' 's/foo/bar/g' file.txt



---

reference this answer in a different Stack Overflow question about accessing variable data in scripts

http://stackoverflow.com/a/19381235/503463


Question: How do I access the version string in an npm version script?



Client work is done, delivered deployed. Version fields are universally ignored.

One particular example is a private plugin we're using on several WordPress sites. It's a set of tweaks for other plugins that just wouldn't be appropriate to distribute. The way we upgrade is to download a zip archive of the Git repository, then upload that through the WordPress interface. The folders have different names, so this works pretty well in terms of collisions, but the only indication of file differences is the version string. If that's not different, there's no way to tell versions apart. 







In my tests, I ended up with every item from my package.json file along with 90+ assorted config vars.





These commands also eliminate a lot of uncertainty about SemVer. I've personally been fuzzy about which numbers to increment, and I've seen plenty of cases where people just arbitrarily increment versions. Clear action verbs  are very easy to understand.



One particular example is an internal WordPress plugin  used on several sites. Upgrading installations is done manually by uploading a zip archive of the Git repository. Here's the problem illustrated, these are all different versions:

![ image showing three copies of the plugin with identical version strings](https://placeholdit.imgix.net/~text?txtsize=33&txt=many%20plugins&w=600&h=280)



[so]: http://stackoverflow.com/a/19381235/503463
[npm version]: https://docs.npmjs.com/cli/version
[main]: https://docs.npmjs.com/files/package.json#main

[npm scripts]: https://docs.npmjs.com/misc/scripts
[package.json vars]: https://docs.npmjs.com/misc/scripts#packagejson-vars
[config vars]: https://docs.npmjs.com/misc/scripts#configuration