reference this answqer in a different stackoverflow question about accessing variable data in scripts

http://stackoverflow.com/a/19381235/503463


Question: How do I access the version string in an npm version script?



Bascially it's this: Versioning has usually been an afterthought. Client work is done, delivered deployed. Version fields are universally ignored.

That's fine, but it's starting to get to be a problem. One particular example is a private plugin we're using on several Wordpress sites. It's a set of tweaks for other plugins that just wouldn't be appropriate to distribute. The way we upgrade is to download a zip archive of the Git repository, then upload that through the WordPress interface. The folders have different names, so this works pretty well in terms of collisions, but the only indication of file differences is the version string. If that's not different, there's no way to tell versions apart. 

### Enter NPM 

Npm has a version command which is just wonderful. It's main purpose is to update the project's version recorded in `package.json`. As a bonus, if the project is a Git reposority, the command will commit the updated `package.json` file. 

THere are three sub-commands which increment the version inline with the SemVer specification. Starting from a fictionally v1.0.0, the three commands do the following:

* `npm version major` - v2.0.0
* `npm version minor` - v1.1.0
* `npm version patch` - v1.0.1

### Even better with Scripts

That's great, but not every project is an npm module. Some, like the WordPress plugin mentioned above, have their own version strings. It's very easy to update these to match NPM's incremented versions. 

Just add a `version` script to `package.json` and we're 90% there. The only complication is figuring out how to replace the Version string. Luckily some ancient Unix tools are perfect for this.



