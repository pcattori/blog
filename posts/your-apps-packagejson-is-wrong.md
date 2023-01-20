# Your app's `package.json` is wrong

Avoid using the `name`, `version` and `description` properties within `package.json` for an app.
Instead, start with a minimal `package.json` for an app:

```json
{
  "private": true
}
```

_This is the ideal `package.json` for an app. You may not like it, but this is what peak performance looks like._

Then only add what you need, like `scripts` and `dependencies`.[^dependencies]
[^dependencies]: Best if you let `npm` or `yarn` just auto-add dependencies for you.

Don't add the following for your app unless you absolutely need them:

- `name`
- `version`
- `description`
- `keywords`
- `license`
- `repository`
- ...or anything else not needed to develop and run your app[^packagejson-properties]

[^packagejson-properties]: Check out the [full list of `package.json` properties](https://docs.npmjs.com/cli/v9/configuring-npm/package-json)

> What does `"private": true` even mean?

It's for declaring that you are [not going to publish this package](conveying) on [npm](https://www.npmjs.com/).
Which is the case for 99.999% of apps.[^npm-app]
[^npm-app]: If you know of a good use-case for publishing an app, I'd love to hear about it!
So definitely use `private` for your apps.

> But why not use metadata properties like `name` and `version` alongside `private`?

The purpose of properties like `name`, `version`, and `description` is to communicate metadata for your package to tools like `npm`.
They are what power things like the `npm info` and `npm search` commands.

```sh
❯ npm info lodash

lodash@4.17.21 | MIT | deps: none | versions: 114
Lodash modular utilities.
https://lodash.com/

keywords: modules, stdlib, util

dist
.tarball: https://registry.npmjs.org/lodash/-/lodash-4.17.21.tgz
.shasum: 679591c564c3bffaae8454cf0b3df370c3d6911c
.integrity: sha512-v2kDEe57lecTulaDIuNTPy3Ry4gLGJ6Z1O3vE1krgXZNrsQ+LFTGHVxVjcXPs17LhbZVGedAJv8XZ1tvj5FvSg==
.unpackedSize: 1.4 MB

maintainers:
- mathias <mathias@qiwi.be>
- jdalton <john.david.dalton@gmail.com>
- bnjmnt4n <benjamin@dev.ofcr.se>

dist-tags:
latest: 4.17.21

published a year ago by bnjmnt4n <benjamin@dev.ofcr.se>
```

> Got it. But still don't see the harm in using them for apps.

Who are you trying to convey this information to?

Your users?
Put the information where they'll actually see it.
In your docs, or better yet, within your app itself.
If users of your app have to look at your `package.json` to get your app's name and version, you've got some serious UX problems.
Not to mention that if your app's code isn't easy to get, your users won't be able to access the `package.json` in the first place.

What about your coworkers?
Do them (and your future self) a favor and write a README or some docs.
There's already too many places to put your app's name (your repo name, README title, docs, etc...),
so don't add yet another place in your `package.json`.
That just makes it more confusing when all those name end up being different.

> Are there any exceptions?

If you need a central place to put your app's metadata where you will _programmatically_ access it, then I won't judge you for putting it in `package.json`.
For example, if you wanted to templetize your docs so that your app's name and version came directly from the `name` and `version` fields in `package.json`.
Just remember that you are not using those properties for what they were designed for, which can create [false affordances](https://en.wikipedia.org/wiki/Affordance#False_affordances).
