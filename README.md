# Hopper
## Distilled Package Management

It basically works just like RPM or debian, except that the installation root
is fluid -- wherever your code project is, that is where the directory is
relative to which the files will be installed. Go ahead! package up files meant
to go under `vendor` with reference to gb, or `Godep/_workspace/src`, and tell
hopper to put those files there, and there they will go.

It works for any language, too. Hopper is a language-agnostic dependency
management system. you tell it where to put stuff, and it puts stuff there.
It is written in go and has golang in mind, but would work equally well for
just about anything.

The fluidity of the package root allows us to take the brilliant concept of
file-based dependency management right to your code project's front door.

## Specifications
1. Hopper MUST include the files to be installed in a zip file, all with paths
   relative to the package root.
2. The package root for hopper is determined at the start of every operation by one of the following:
    1. If the PWD has a file called '.hopper/dependencies.json', the PWD is taken as the package root.
    2. If any parent of the PWD has a file called '.hopper/dependencies.json', that directory
       is taken as the package root.
    3. If none are found, the environment variable HOPPER\_PACKAGE\_ROOT is checked.
    4. Failing all these, the entry for "package\_root" in /etc/hopper/config.json is consulted, if present.
3. hopper files (as found in PWD or its parent under `.hopper`)
    1. Dependencies: .hopper/dependencies.json
    2. Project-Specific configuration: .hopper/config.json (for project-specific configuration, overriding anything in /etc)
    3. Package installation database (PID): .hopper/packages.sqlite .
3. The package format:
    1. The package is simply a regular zip file, with two directories in its root: `package` and `data`.
    2. Under `package`, a file named `package/manifest.json` contains the following information:
          1. Package name under the JSON key `package/name`
          2. Package version under the JSON key `package/version`
          3. Package provides a list of strings under the JSON key `package/provides`
          4. Package requires a list of strings under the JSON key `package/requires`
          5. Package conflicts list under the JSON key `package/conflicts`
          6. Each string in provides/requires/conflicts list is of the form
             `^<name>((==|>=|>|<|<=)<ver>)?$` where `<name>` is of the form
             `^[A-Za-z0-9\_]+$` and `<ver>` is of the form `^[^[:space:]]+$`.
          7. Content author under JSON key `package/author`
          8. Package maintainer under JSON key `package/packager`
          9. Homepage under JSON key `package/url`
          10. Summary under JSON key `package/summary`
          11. Description under JSON key `package/description`
          12. Release version under JSON key `package/release`, must be non-negative integer
    3. The files under `data` are meant to be unpacked relative to the package root.
3. Installation:
    1. The PID is checked for any package of the same name installed.
    2. If the same name is installed, it is first marked for removal. in the data
    1. The package file contents are first checked to see if they will overwrite a file found in the PID.
         1. If it only overwrite files for packages that will be removed, this is okay.
         2. Otherwise, an error message is thrown and the package is not removed.
    2. If everything looks good, the files are unpacked to the package root and any
       packages being upgraded get deleted from the database.
    3. The database is updated to include the new version of the package and its files, etc.
4. Querying:
    2. Usage: `hopper query {packages|manifest|files} [--json]`
    1. Hopper can query its database for:
        1. What packages are installed and their versions
        2. Info (whatever was in the manifest)
        3. Files installed (and integrity flag)
        4. output may be in JSON or normal text
5. Removal:
   1. Usage: `hopper remove [--cascade] {<name>}`
   2. Removes all files listed for given named package (must be actual name, not provides name)
   3. Removes all entries in database associated with package

6. Repo format:
   1. File called `hopper\_repo.json` like this:
```
{ "packages": {
    "<name>": {
        "<version>": "filename" (relative to URL where `hopper\_repo.json` was found)
    }
}
```
Provides and concrete names alike find themselves in that list. It is simple. If user requests a certain package name, it is found.

Thus dependency management is solved for any and all code projects. For different languages, or layouts, simply make a hopper repo specific to that setup, as `http://foo.org/go/gb/x64/` or `http://foo.org/go/gb/any/`.
