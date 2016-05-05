# npm scripts

## build and automation

----

# First some questions!

-----

# How many of you 

# use `npm`?

^ "that's a lot"

^ "this is going to be new territory for some of you"



-----

# Already using `npm` scripts?

----

# How many of you use Windows?



----

# How many of you have written a small `bash` script?

---

# `gulp`? `grunt`? `broccoli`?

----

# CodeKit? Some other file watching tool?

-----

# Did any of these feel like they were taking over your project?

----

## We're going to talk about how to build your own toolkit.

-----

## The first secret

`npm` doesn't do anything interesting on its own

----

## `npm` lets you install tools, 

## then run them.

^ That's it. All the rest is a module you install.

^ npm scripts aren't some noble goal. gulp and grunt can be some of the tools you run. They're just simple, and I think that's the power. Don't get too fancy. Keep the system understandable.

----

# just one tool

## `npm`

### `npm install`,  `npm run thing`, `npm test`

-----

# No Global

`npm install -g` is an anti-pattern

^ when npm runs a script in your package.json, it adds a directory with all the commands your package has installed to your system PATH, so if you depend on it in your package.json, you can use it. No need to install things globally. More disk space, but the benefit is that apps never interfere with each other. They all have what they need, and tell you what that is.

----

## 250,000 packages in the registry

^ not all build tools of course, but it's a _huge_ ecosystem.

^ It turns out that this really is an ecosystem, it's self-supporting. People make tools, people make tools that depend on those tools. you can use them or make your own, depending on how much programming you're willing to do.

----

# Goals of a build system

* Repeatability
* Reliability
* Reduce Redundancy

^ Not every build system is very good at these out of the box.

^ Repeatability means that given the same source information, the build should emit the same files. It's perfect when even the hashes match.

^ Reliability means it works _every time_. Unreliable builds are a huge developer time sink. This means that a build that sometimes forgets to update a file, or sometimes crashes is not a good one. If it hides errors, that's unreliable.

^ Reducing redundancy is good for a couple reasons: sometimes there's things that when you change them, other correlated things have to change. A single source of truth is super useful for those bits -- variables in SASS or PostCSS, references, etc. It also can mean smaller file sizes.

-----

# Modularity

----

## Some problems with front-end module systems

* Flat namespace
* interdependency
* Unable to track what's used

----

## Sadly `npm` scripts won't solve these completely

-----

## Namespaces

package names are unique, but different things can require different versions.

for node, this works ... but in CSS? Browser javascript?

Let's look at solutions

^ This is like a human problem -- some names are more unique than others. If I say "I'm looking for Sarah", and the context is small, it's fine. There's probably only one Sarah. Now if I walk into a room with 500 people, stand on stage and say "I'm looking for Sarah", there's going to be some confusion and maybe a dozen people stand up.

^ We can be more specific -- "Is Sarah Soueidan" here? There's not very many of those Sarahs, but even so, there could be more than one. Or, we can assign a unique ID to everyone -- maybe a driver's license, ticket number, whatever. It won't be super meaningful, but it'll be unique.

----

## ignore it

### "just be careful not to use conflicting modules"

^ dr-frankenstyle is like this.

----

## Make it an error

### `copy-browser-modules` is a tool I wrote

^ The thing it does is copies a directory of stuff out of the `node_modules` directory and into a flat heap with no nesting module inside module. It throws an error if you have conflicts. It's a decent middle step in a build.

----

## Remap it

### some packages let you change your CSS from `.foo` to `.4a35b_foo`, & expect (or help) HTML to match

^ This goes especially nicely with React where you can have markup that looks like HTML but is really code and gets some mapping under the hood.

^ This is what browserify and webpack do, too -- they isolate your javascript into closures and give them new names (mostly numeric for efficiency. It works better there since it can assume you're using node-like conventions)

----

# Clever modularity

```sh
npm install --save-dev dr-frankenstyle pui-css-forms
```

^ Some folks at Pivotal Labs cooked this up, and it's a tool we use at npm (well, a fork anyway — it's not bug free!)

```json
"scripts": {
  "build": "dr-frankenstyle static/css/"
}
```

^ With this we actually get a `static/css/components.css` with the styles from `pui-css-forms` ... and its dependent `pui-css-typography`

----

# Interdependency

### npm won't solve everything but it adds a couple tools

^ I won't lie: figuring out what a module is and where to break it off is hard. It's a skill you have to practice and analyze a bit. What's reusable? What's the context? What does it depend on, and what does _that_ imply?

^ One thing npm is pretty good at is annotating dependencies, but it won't _find_ what you depend on for you. When you make packages, you list what they need.

----

# Simplifying

## make it so `npm install` is all you need

````json
"scripts": {
  "prepublish": "npm run myFancyBuild"
}
````

^ This is a magic script, it's really badly named since it runs not just at publish time if you're making a module, but at `npm install` time when it's in the root package.

-----

## Levels of simplicity

### 1. File to file transform

^ one input, one output, it's really easy to know when the source has been updated and what to do with that.

### 2. Directory to file transform

^ Many inputs, one output. It's a little slow to scan for changes and see what's changed. Watchers might use complicated APIs from your operating system with weird limits. When your project gets big, this is where things start going awry.

### 3. Directory to directory transform

^ This is where you have a bunch of 1:1 transforms, so you can predict what the input was given just the output file name.

### 4. Many file to many file transform

^ Many inputs, many outputs. Sometimes there's non-trivial mapping -- "all the CSS gets combined, but the javascript files are compiled one by one" -- this is where you will tear your hair out debugging unless you isolate this really well and understand it.

----

# Replacing CodeKit

```sh
npm install --save-dev watch
```

and

```json
"scripts": {
  "dev": "watch 'npm run build' --wait=2 --filter='*.css'"
}
```

^ you can start this with `npm run dev` and stop it with `^C`. That's a pretty common pattern. Every time you change a CSS file it'll wait a couple seconds and then run `npm run build`, whatever you made that be.

^ Mind your quotes though, since we've got bash scripts inside JSON, sometimes you have to be mindful of the syntax there. This is part of the reason to keep things simple.

^ All this is a time saving device. If it explodes on you when you're trying to get work done, you haven't saved time, just moved it until an unexpected moment.

----

# You can still use `gulp` or `grunt`!

```json
"scripts": {
  "build": "gulp frobnicate"
}
```

^ It's just hidden behind `npm`, so  no matter what tools you _really_ use, you can just use `npm` to trigger them all.

----

# Simple tools

```json
"scripts": {
  "minify": "uglifyjs < static/app.js > static/app.min.js"
}
```

^ There's a lot of tools that have a command line tool that you can use in npm. uglifyjs is one of my favorite, since I can pipe javascript to it and it pops it out, minified.

----

# Let's add another

```json
"scripts": {
  "minify": "uglifyjs < static/app.js > static/app.min.js && cssmin < app.css > app.min.css"
}
```

^  Notice I'm not minifying a whole bunch of files. I'm trying to keep this minimal, so if something goes wrong, I know just what part did it.

^ cssmin doesn't need to know about sass or anything. uglifyjs doesn't need to understand jsx. No manifest or automatic replacement of filenames in the HTML, just minifying. There's techniques for cache-breaking that don't involve anything like that.

^ Notice that these are one file to one file transformations.

^ Also, _measure_ this. It might just be that your minification doesn't get you a real benefit over just gzipping things. You own your pipeline, so you get to decide what's in it, and it's simple enough that measurement can be really easy.

-----

^ Notes follow

----

# Notes

`npm-run-all` run task in series or parallel, works on Windows

* can glob tasks
* parallel watchers are useful to keep them simple and orthogonal

call `npm run task` in a script a "sub-task"

Focus on simplicity and understandability