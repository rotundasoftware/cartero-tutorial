
# Tutorial: Getting started with cartero

[cartero](https://github.com/rotundasoftware/cartero) is an asset pipeline designed to reduce the friction involved in applying modular design principles to front end web development. It facilitates an organizational structure in which related JavsScript, css, templates, and images are kept together in directories, or "packages". A package contains all the assets needed for a particular interface element or web page. For example, a package might contain assets for
* A calendar widget
* A popup dialog
* A header or footer
* An entire web page

Such packages
* Must contain at least some JavsScript.
* May contain css and images.
* May depend on other packages (and be depended upon).
* Can be local to a particular web page, shared between pages, shared between applications, or published via [npm](https://www.npmjs.org/).

cartero does not introduce many new concepts, and the same modular organizational structure it facilitates could also be achieved by stringing together other build tools and the appropriate `<script>`, `<link>`, and `<img>` tags. However, cartero is built from the ground up for modularized applications, and eliminates the friction that occurs when using conventional build tools with modular directory structures.

## Modularizing the express boilerplate app

In this tutorial we'll convert the boilerplate express application to a modular structure and use cartero as the asset pipeline. First, let's generate the boilerplate express application with `express-generator`.


```
$ npm install -g express-generator
$ express
$ npm install
```

The `npm install` command installs all the dependencies in the boilerplate application.

Now, notice the flat default structure of the default view directory.

```
views
├── error.jade
├── index.jade
└── layout.jade
```

The problem with this conventional flat directory structure is the other assets that are needed by these server side templates end up being all over the place. Where do you put the images, css, and js that are used by `index.jade`? All in separate directories? No! That is a crazy directory structure left over from 1990. It leads to long, unscoped css files, disorganized directories full of unrelated images or JavaScript files, and cripples your ability to  reuse front end components between projects.

What we want is a modular directory structure, where each web page has its own directory, and all the assets local to that page are in the page's directory. cartero is designed to facilitate building applications with this structure. Let's re-organize the view directory accordingly.

```
views
├── error
│   └── error.jade
├── index
│   └── index.jade
└── layout
    └── layout.jade
```

Now, notice that the `style.css` file in the `public/stylesheets` directory. That file pertains to the `layout.jade` template - it contains css that is applied to the layout of the site. So let's put it where it belongs, in the `layout` folder, next to the `layout` template!

Now out views directory looks like so

```
views
├── error
│   └── error.jade
├── index
│   └── index.jade
└── layout
    ├── layout.jade
    └── style.css
```

We also need to replace references to the old view locations with the new locations:

* In `app.js`, line 41 and 52, change `res.render('error',` to `res.render('error/error',`
* In `views/error/error.jade` and `views/index/index.jade`, change `extends layout` to `extends ../layout/layout`
* In `routes/index.js`, line 6, change `res.render('index',` to `res.render('index/index',`

We are on our way to making each of these directories a package, with the `index` and `error` packages depending on the `layout` package. cartero packages, being npm packages, are just directories with [package.json](https://www.npmjs.org/doc/json.html) files. In catero package.json files, assets of the package are enumerated. Let's add a package.json file to each of our individual view directories that looks like this:

```json
{
	"style": "*.css"
}
```

The style property serves to enumerate the style assets required by the package. Now we have:

```
views
├── error
│   ├── error.jade
│   └── package.json
├── index
│   ├── index.jade
│   └── package.json
└── layout
    ├── layout.jade
    ├── package.json
    └── style.css
```

cartero packages, being npm packages, also require a valid JavaScript entry point, which can be specified via the `main` property in `package.json`, and which defaults to `'index.js'`. Let's add an `index.js` entry point to each package.

```
views
├── error
│   ├── error.jade
│   ├── index.js
│   └── package.json
├── index
│   ├── index.jade
│   ├── index.js
│   └── package.json
└── layout
    ├── index.js
    ├── layout.jade
    ├── package.json
    └── style.css
```

### Expressing dependencies

Now we have finished defining three packages. The next step is to express the dependency of the `error` and `index` package on the `layout` package. cartero does not define a new way of expressing dependencies. Instead, it piggy packs on the dependency information that is already in your JavaScript files. By default it expects JavaScript files to use the [CommonJS](http://0fps.net/2013/01/22/commonjs-why-and-how/) `require('modules')` syntax to express dependencies, but you can also use AMD or even ES6 style modules with a little bit of configuration. Let's change the `index.js` in the `error` and `index` package to express the dependency on the `layout` package. (The argument to `require` is resolved using browserify, which implements the [node module system](http://nodejs.org/api/modules.html#modules_loading_from_node_modules_folders) in the browser.)

```javascript
require('../layout');
```

That's all we need. We can now run cartero and we will see that css bundles are generated for the `error` and `index` packages that include the css defined in `layout/style.css`. To install cartero,

```
$ npm install -g cartero
```

Now, run cartero, supplying our views directory and an output directory as arguments.

```
$ cartero views public/assets
info processing parcels in "views"
info script bundle written to "public/assets/9dc1d295a091923aab8af612ace0ceb3a8c433c6/layout_bundle_e3793e353f96029f41c518514918edd1cdc28479.js"
info style bundle written to "public/assets/9dc1d295a091923aab8af612ace0ceb3a8c433c6/layout_bundle_ec4e27748ea9c3e6146ba46cc0aa119757631f26.css"
info script bundle written to "public/assets/00b3a032e4953a0efc9acd0f111cb3a257821128/index_bundle_c83054548200fafda4a8a770bd6ef5ffda5cc392.js"
info style bundle written to "public/assets/00b3a032e4953a0efc9acd0f111cb3a257821128/index_bundle_ec4e27748ea9c3e6146ba46cc0aa119757631f26.css"
info script bundle written to "public/assets/82bb4c03156bcc8915cb1c53080c51b1439fd54c/error_bundle_5ab0eb8d967c5f4397b98a62ba8321f42b99f69a.js"
info style bundle written to "public/assets/82bb4c03156bcc8915cb1c53080c51b1439fd54c/error_bundle_ec4e27748ea9c3e6146ba46cc0aa119757631f26.css"
```

cartero just scanned the `views` directory for packages, and output JavaScript and css bundles for each page / package to the output directory `public/assets`. The long strings are shasums that guarantee each package a unique directory in the output folder. If we look at the css bundle output for the `index` package, we can see it includes the css from `layout/style.css`. (The directory name on your system will be different, since the shasum is in part determined by an absolute path, but the contents will be the same).

```
$ cat public/assets/00b3a032e4953a0efc9acd0f111cb3a257821128/index_bundle_ec4e27748ea9c3e6146ba46cc0aa119757631f26.css
body {
  padding: 50px;
  font: 14px "Lucida Grande", Helvetica, Arial, sans-serif;
}

a {
  color: #00B7FF;
}
```

### Server side lookups

Now we just need to add the logic that enables our web framework to find the assets associated with a given package. cartero has a small server side 'hook' that is able to look up the assets for any given package using the information dropped into the output directory by cartero at build time. (A hook is currently only available for node.js, but one would be easy to implement in any server side framework.) As an added convenience, there is cartero middleware available for express that will automatically populate the `cartero_js` and `cartero_css` properties of `res.locals` with the script and link tags that are needed to load all the assets of the page being rendered.

First, install the hook and the middleware:

```
$ npm install cartero-node-hook --save
$ npm install cartero-express-middleware --save
```

Now we need to instantiate the hook and install the middleware by adding the following lines to `app.js`, just below line 11 where the app object is created:

```javascript
var carteroHook = require('cartero-node-hook');
var carteroMiddleware = require('cartero-express-middleware');

var hook = carteroHook(path.join(__dirname, 'views'), path.join(__dirname,'public/assets'), {outputDirUrl : 'assets/'});
app.use(carteroMiddleware(h));
```

The cartero hook constructor takes two arguments, which are the same as the command line arguments for cartero itself. The `outputDirUrl` option tells cartero the url of the output directory, which in our case is `'assets/'`, since the express static middleware is rooted at the `public` directory, and the cartero output folder is at `public/assets`. The hook we create is then passed into the cartero middleware constructor.

Our last step is to modify the layout template to output the `cartero_js` and `cartero_css` variables:

```jade
doctype html
html
  head
    title= title
    | !{cartero_js}
    | !{cartero_css} 
  body
    block content
```

### Test drive

When we start our application using

```
$ npm start
```

And then direct our browser to `http://localhost:3000/`, we should see the index page, styled with the css in `layout/style.css`. Inspect the source of the page and obverse that our script and link tags have been generated by the cartero express middleware (line breaks added):

```html
<!DOCTYPE html>
<html>
	<head>
		<title>Express</title>
		<script type='text/javascript' src='assets/00b3a032e4953a0efc9acd0f111cb3a257821128/index_bundle_c83054548200fafda4a8a770bd6ef5ffda5cc392.js'></script>
		<link rel='stylesheet' href='assets/00b3a032e4953a0efc9acd0f111cb3a257821128/index_bundle_ec4e27748ea9c3e6146ba46cc0aa119757631f26.css'></link>
	</head>
	<body><h1>Express</h1><p>Welcome to Express</p></body>
</html>
```

### Modularization achieved

The boilerplate app is now modular. Let's get rid of the old directories in the `public` that hold assets organized by type. You don't have to organize assets into big directories by their type anymore! All assets will be where they belong from now on.

```
$ rm -r public/{images,javascripts,stylesheets}
```

You can find the finished copy of this restructured express boilerplate application in the cartero-tutorial repository at [the `base` branch](https://github.com/rotundasoftware/cartero-tutorial/tree/base).

## Watch mode, images

Now for some fun stuff. Let's see how we an image local to `index.jade` would fit into this new modularized directory structure. While we are at it, let's turn on cartero watch mode, which can be used in development to rebuild your assets as appropriate when changes are made. Running cartero in watch mode is as simple as adding a `-w` flag to the cartero command:

```
$ cartero views public/assets -w
```

Now let's put [an image](http://www.diariocultura.mx/wp-content/uploads/2012/11/cartero.jpg) in `index.jade`. First we need to save the image into the `index` package directory, and then enumerate it in the `image` property of its package.json file. Afterwards, our directory will look like so:

```
views/index
├── cartero.jpg
├── index.jade
├── index.js
└── package.json
```

And the package.json file will look like:

```json
{
	"style": "*.css",
	"image": "cartero.jpg"
}
```

Notice the output from the cartero command, which is running in watch mode, shows that the change to the package.json file was registered and that the new image was written to the output directory.

```
info watching for changes...
info watch package.json changed "views/index/package.json"
info watch image asset written to "public/assets/00b3a032e4953a0efc9acd0f111cb3a257821128/cartero.jpg"
info watch processing parcels in "views"
```

Now let's insert the image into our `index.jade` template:

```
extends ../layout/layout

block content
  h1= title
  p Welcome to #{title}
  img(src=cartero_url('./cartero.jpg'))
```

The `cartero_url` function is the third (and last) property that is attached to `res.locals` by the cartero middleware. It takes a single argument, which is an asset path that is resolved using the node require algorithm (relative to the view template), and returns the url of the asset. Now when you reload the page in your browser, you should see the cartero image on the index page.

Note that because the node require algorithm is used to resolve the reference, you can easily reference images in other packages. For example, if the cartero image was in a package called `my-module` in some `node_modules` directory, you could resolve the url with:

```
  img(src=cartero_url('my-module/cartero.jpg'))
```

The source for this project can be found in the cartero-tutorial repository at [the `images` branch](https://github.com/rotundasoftware/cartero-tutorial/tree/images). 

## What's next

These topics may be covered in future posts. If you'd like to see a particular tutorial, or have questions / comments about cartero in general, please leave a comment on the hacker news article about this tutorial. Thanks for stopping by!

### Modularization everywhere

The same modular directory structure used for full pages can be employed to easily reuse interface elements like widgets, dialogs, headers, footers, etc., which can be either application specific, shared between projects, and / or published via npm. Generally these reusable interface elements can be put in a `node_modules` on the app level. (You can keep your app specific packages separate from shared packages by [creating a symlink in your node_modules directory](https://github.com/focusaurus/express_code_structure#the-app-symlink-trick) that points to your app specific modules.)

### Transforms

cartero supports precompiling and postprocessing files using transform streams. For example, its easy to [convert scss to css](https://github.com/rotundasoftware/sass-css-stream) or [uglify JavaScript](https://github.com/hughsk/uglifyify).

### Pulling out large packages

Including large, common packages like jQueryUI into the js and css bundles of every page is not efficient as it prevents them from being cached separately by the browser. You can instead pull those packages out of your page bundles and load them separately, from your own server or a CDN.

### AMD and ES6 modules

If you prefer AMD or ES6 modules, you can use that syntax instead of CommonJS to express dependencies, thanks to [browserify transforms](https://github.com/substack/node-browserify/wiki/list-of-transforms).
