# legonigel.github.io
**Nigel Armstrong is currently a student at Christopher Newport University**

This website runs on [Jekyll](http://jekyllrb.com/), a blog-aware, static site generator, with a lot of help from [Grunt](http://gruntjs.com/).

## The Stack
- [Bower](http://bower.io/): Manage front end dependencies
- [Bundler](http://gembundler.com/): Manage Ruby dependencies
- [Grunt](http://gruntjs.com/): Automate Jekyll development and build tasks
- [Node.js](http://nodejs.org/) and [NPM](https://npmjs.org/): Required for Grunt
- [Ruby](http://www.ruby-lang.org/): Required for Jekyll.

### Recommended Setup
- It is a pain on windows, I recommend against it. Maybe I will eventually post on here about how to do it.

## Install Dependencies
Run the following commands to install the dependencies:
- Bower: `bower install`
- Bundler: `bundle install`
- NPM: `npm cache clean` and then `npm install`

## Grunt Workflow
- `grunt stage`: Copies the loadFont() and loadCSS() functions from Bower to the Jekyll _includes directory
- `grunt serve`: Compiles all files and opens the site in your default browser. A watch task watches for changes to files, recompiles if necessary, and injects the changes into the browser with BrowserSync
- `grunt check`: Checks for outdated dependencies with grunt-dev-update, Javascript code quality with JSHint, Sass code quality with [SCSS-Lint](https://github.com/causes/scss-lint), and Jekyll health with `jekyll doctor`
- `grunt test`: Runs through the E2E tests via [CasperJS](http://casperjs.org/) and [PhantomJS](http://phantomjs.org/)
- `grunt build`: Builds an optimized site to the dist directory
- `grunt deploy`: Runs the perf task, runs the build task, and then deploys it
- `grunt perf`: Checks Google’s PageSpeed Insights and then runs [Phantomas](https://github.com/macbre/phantomas) to return and record site metrics.

## Hat Tip
The site was scaffolded by [Yeoman](http://yeoman.io/) and [generator-jekyllrb](https://github.com/robwierzbowski/generator-jekyllrb).
The site was based upon a site by [davidensinger](https://github.com/davidensinger/davidensinger.github.io)