## Mylet Development Guide ##


### Overview ###

Mylet is a custom user app that can be run in context of user's MYLE account in smart-phone and in browser and also perform some lightweight server code.

Mylet consists of the following parts:
 1. UI - HTML5 app
 2. Controller - server side JS that is run by accessing specific URL
 3. Hook - server side JS that is run as soon as new record appears in user account


### Mylet life-cycle ###

 - developer creates and publishes a mylet to market
 - user installs a mylet from market


### How to create a mylet ###

Go to [http://console.getmyle.com/constructor](http://console.getmyle.com/constructor) and click `Create a mylet` button.

As soon as you enter new mylet name and click `Create` button, you will be redirected to another page were you will see boilerplate project structure.


### Project structure ###

![image](https://cloud.githubusercontent.com/assets/1598710/7418711/20ad6812-ef2d-11e4-9b42-3624a9c5320e.png)

Default mylet template uses [Framework 7](http://www.idangero.us/framework7) as a UI library. It is not a mandatory and can be changed.

Let's walk through project files available:
 - `controllers/controller.js` - server side code with URL endpoint. See [Mylet controller](#mylet-controller).
 - `controllers/hook.js` - server side code executed on new record creation. See [Mylet hook](#mylet-hook).
 - `tables/table.table` - a JSON file with specific schema that has some table data to be used for queries. See [Mylet tables and queries](#mylet-tables-and-queries).
 - `views/index.js` - mylet UI starting script.
 - `views/index.ejs` - [EJS](http://www.embeddedjs.com/) template of starting page.
 - `views/layout.ejs` - layout [EJS](http://www.embeddedjs.com/) template page.
 - `views/index.css` - UI CSS styles.
 - `icon.jpg` - mylet icon. See [Mylet configuration](#mylet-configuration) for how to change icon file name.
 - `bower.json` - bower configuration file. See [Managing client side dependencies](#managing-client-side-dependencies).
 - `mylet.json` - mylet configuration file. See [Mylet configuration](#mylet-configuration).
 - `package.json` - npm conflagration file. See [Managing server side dependencies](#managing-server-side-dependencies).


### Toolbars ###

 - main toolbar:

   - `Build & Run` - builds and opens mylet in new window.
   - `Publish` - builds and publishes current mylet version to market.
   - `View Output` - shows last output from hook or controller during mylet execution during development.
   - `Git (on branch)` - shows source control options. See [Source control](#source-control).

 - context toolbar is changed depending on active project document. Possible buttons:

   - `Edit` - when in document view mode, switch to document editing mode.
   - `Rename` - renames current document or folder name.
   - `Delete` - deletes current document or folder.
   - `Done` - when in editing mode, switches back to view mode.
   - `Save` - when in editing mode, saves current document.
   - `Install package` - when viewing `bower.json` or `package.json` is used to install a dependency. See [Managing client side dependencies](#managing-client-side-dependencies) or [Managing server side dependencies](#managing-server-side-dependencies).
   - `Add file` - when in folder mode, allows to create empty document or upload another one.
   - `Add folder` - when in folder mode, creates sub-folder. 


### Mylet configuration ###

Mylet configuration is provided via `mylet.json` file. Take a look at it's schema wit default values:

```js
{
  "title": undefined,
  "keywords": undefined,
  "icon": "icon.jpg",
  "controllerJs": "controllers/controller.js",
  "hookJs": "controllers/hook.js",
  "indexJs": "views/index.js",
  "indexCss": "views/index.css",
  "indexView": "views/index.ejs",
  "tableStore": "tables",
  "config": undefined
}
```

Where 
 - `title` - mylet title
 - `keywords` - array of strings that trigger hook execution. See [Mylet hook](#mylet-hook).
 - `icon` - path to mylet icon.
 - `controllerJs` - path to a controller file. See [Mylet controller](#mylet-controller).
 - `hookJs` - path to a hook file. See [Mylet hook](#mylet-hook).
 - `indexJs` - path to starting client side JS file.
 - `indexCss` - path to starting CSS file.
 - `indexView` - path to starting HTML.
 - `tableStore` - path to a folder where to look for `*.table` files.
 - `config` - an object representing configuration data.

`config` object can be accessed in controller or in hook like this:

```js
var ctx = require("mylet-context");
var config = ctx.mylet.config;
```

`config` object is not accessible from client side code. You can use controller to pass config values to client side. See [Mylet controller](#mylet-controller).

`config` object is read-only, any changes to it from controller/hook are not persisted.


### Mylet controller ###

Controller is a server side code that is executed making request to a specific  URL.

The URL can be retrieved on client side from global `__controller_url_path_name`. Example of a request to a controller:

```js
$.getJSON(window.__controller_url_path_name);
```

Controller code is executed when a request to its URL is initiated. Controller must complete execution by sending response back. Both current `request` and `response` objects are available through `mylet-context` module.

Here is example of how to read a field from a request and send it back:

```js
var ctx = require("mylet-context");
ctx.res.send(ctx.req.body.someField);
```

Controller has limitations on what it can do. See [Server code limitations](#server-code-limitations).


### Mylet hook ###

Hook is a server side code that is executed once new user record appears. Hook is similar to [controller] but doesn't have `request` and `response` object in its context, but instead it has `record` object.

`record` object is accessible through `mylet-context` module as well.

Here is example of logging newly appeared record:

```js
var ctx = require("mylet-context");
console.log(ctx.record.phrase);
```

`record` object has two field:
 - `phrase` - result of record recognition.
 - `time` - time when record was created - string in ISO 8106: `2015-04-29T15:16:55Z`.
 - `location` - object representing where the record was taken:
  - `longitude`
  - `latitude`
  - `altitude`
 - `buffer` - raw audio buffer.

Hook has limitations on what it can do. See [Server code limitations](#server-code-limitations).


### Server code limitations ###

Server code is limited:
 - code execution is limited to 10 seconds
 - (disk, network, memory limitations?, more to come...)


### Mylet user settings ###

User settings can be accessible in server and client code. They can be read and written.

User mylet settings are accessible through `mylet-settings` module:

```js
var settings = require("mylet-settings");
```

#### User settings in client code ####

`mylet-settings` module provides the following methods:
 - get(name, doneFn) - reads a setting with given name from server and calls `doneFn` callback function once the setting is read
 - set(name, value, doneFn) - stores a setting with name `name` on server with value `value` and calls `doneFn` callback function once the setting is stored
 
Example:

```js
settings.set("email", "hello@mail.ru");

settings.get("email", function (value) {
    console.log("email:", value);
});
```

#### User setting in server code ####

`mylet-settings` module on server side has similar API, but instead of using callback it returns promises. Example:

```js
settings.set("email", "hello@mail.ru");

settings.get("email")
  .then(function (value) {
      console.log("email:", value);
  });
```

### Mylet frontend ###

#### Page ####

Forntend files are usually put in `views` folder. By default, starting page is `index.ejs`. This is [EJS](http://www.embeddedjs.com/) template with the following globals available:

- `style` - path to a built CSS bundle. See [Mylet building](#mylet-building).
- `script` - path to a built JS bundle. See [Mylet building](#mylet-building).
- `controllerUrlPathName` - has a path to controller. See [Mylet controller](#mylet-controller).
- `mylet` - mylet object with the following fields:
 - `title` - mylet title
 - `name` - mylet name (ID)

__NOTE__: `__mylet_id` is required to be included to client JS globals in order to make `mylet-settings` and `mylet-query` (see [Mylet tables and queries](#mylet-tables-and-queries)) working properly.

#### Style ####

Styles, by default, are defined in `index.css` file. Styles can be split into several file. They can be combined using `@import` directive:

```css
@import "file1.css"; /* file local to current one */
```

[Bower](http://bower.io/) is used for client side dependencies (see [Managing client side dependencies](#managing-client-side-dependencies)). To import a style from `bower_components` folder, use the following approach:

```css
@import "~ionicons/css/ionicons.css"; /* a file from bower_components folder */
```

Styles are preprocessed and combined into a bundle during build step (see [Mylet building](#mylet-building)). A path to the bundle is available in page template in `style` variable.

#### Script ####

The same is applicable to scripts. They are bundled into single JS file, that can be accessible from page template in `script` variable.

Scripts can be spit into several files. They have to follow CommonJS approach to export stuff and to reference other files. 

[Npm](https://www.npmjs.com/) dependencies can be added, see [Managing server side dependencies](#managing-server-side-dependencies). 


### Managing client side dependencies ###

[Bower](http://bower.io/) is used for client side dependencies. In order to add a dependency, select `bower.json` file and click `Install package` toolbar button. You have to type package name to be installed. Package name can also have version number and other options applicable to package names in `bower install ` command line utility.

Every time new package is installed all unused packages are pruned.

To remove a package, just switch `bower.js` to edit mode and remove corresponding line.


### Managing server side dependencies ###

[Npm](https://www.npmjs.com/) dependencies are used for server side code. In order to add a dependency, select `package.json` file and click `Install package` toolbar button. You have to type package name to be installed. Package name can also have version number and other options applicable to package names in `npm install ` command line utility.

Every time new package is installed all unused packages are pruned.

To remove a package, just switch `package.js` to edit mode and remove corresponding line.


### Mylet building ###

[Webpack](http://webpack.github.io/) is used to preprocess and bundle client side resources. Output of a webpack job is a folder with the following files:
 1. JS bundle file, that contains all JS and dependencies combined
 2. CSS bundle file, that contains all CSS and dependencies combined
 3. Any other resources referenced from CSS and JS file. 

__NOTE__: it is required to `require()` starting CSS file somewhere from a JS file in order to build CSS bundle.


### Mylet tables and queries ###

Client side code is able to query two data sources:
 1. Master table - all records in cloud that belong to a user. This is what user recorded with TAP. This table has `master` name.
 2. Mylet user tables - tables that contain some user/mylet specific data. This is what was provided by mylet developer or overridden by user. Names of these table correspond to files with extension `*.table`. So, for example, if file name is `food.table`, corresponding table name to query is `food`.

Good thing is that both sources can be queried and joined using single interface - queries.

See [Querying tables](https://github.com/MyleInc/myle-soft/wiki/Querying-tables) for more details on table queries.


### Source control ###

Source control is done by means of [Git](http://git-scm.com/). Currently only [BitBucket](https://bitbucket.org/) is supported as remote.

By clicking on a `Git (on branch)` button you would see some options:

![image](https://cloud.githubusercontent.com/assets/1598710/7418815/07e397ce-ef2e-11e4-9e4a-577fd6f08933.png)

User can see a branch they are currently on as well as remote repo.

#### Managing branches ####

Clicking on `Manage branches` button user can see the following dialog:

![image](https://cloud.githubusercontent.com/assets/1598710/7418881/85f62032-ef2e-11e4-92cd-326fe99eb1b8.png)

Here user can manage branches.

#### Using remotes ####

On `Source Control` screen there are options two push and pull:
 - `Commit and Push` - commits current changes to current branch and pushes them to remote repo. __NOTE: force push is done.__
 - `Pull and Reset` - pulls changes from remote and __resets__ current branch to remote one. 
 
__NOTE: use these tools with caution because we currently don't support merging capabilities.__ 
