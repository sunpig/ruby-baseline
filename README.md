# ruby-baseline

Instructions for using Bundler to manage a ruby gem environment on a per-project basis,
e.g. using jekyll to manage your blog.

Key concepts:

* Use rvm to set the ruby environment
* Use Bundler for gems and gem dependencies (*not* rvm gemsets)
* Use Bundler to install a local cache of gems and dependencies to minimize need
  for rubygems.org.

See also: [http://ryan.mcgeary.org/2011/02/09/vendor-everything-still-applies/](http://ryan.mcgeary.org/2011/02/09/vendor-everything-still-applies/)

## Initial workflow

### 1. Set ruby version in `.ruby-version`

RVM will switch to use this version of ruby when you enter the project 
directory. (Verify with `ruby -v` from the command line.)

### 2. Add gems you need to `Gemfile`

As normal.

### 3. Create a local cache of reuired gems and dependencies

Run

```
bundle package --all
```

See [http://bundler.io/v1.5/bundle_package.html](http://bundler.io/v1.5/bundle_package.html)

This will create a gem cache in `./vendor/cache`. When you come to run `bundle install`,
Bundler will use the locally cached gems instead of going out to rubygems.org. **The gem
cache should be checked into source control.** This is in case a gem disappears,
or if rubygems.org is unavailable. Also makes for a faster install process.

### 4. Actually install the gems for use

Run

```
bundle install --local --path vendor/bundle
```

See [http://bundler.io/v1.5/bundle_install.html](http://bundler.io/v1.5/bundle_install.html)

Using the `--local` flag forces Bundler to stick to the local gem cache. It will not
connect to rubygems.org. This may be problematic if you're using a different platform
from the one that generated the cache (e.g. dev on Mac OSX, deploy on Ubuntu), because
the cached gems may be platform-specific. In that case, do a `bundle install` without
the `--local` flag.

The `--path` flag forces Bundler to install the gems in a directory under this project.
This is what allows you to have a project-specific gem environment without using
rvm gemsets.

These settings are remembered by Bundler in the `./.bundle` directory. This directory
**should not** be in source control, because it contains log files and per-machine settings.
(Ignore it in `.gitignore`.)

The `./vendor/bundle/` directory **should not** be checked in to source control, because
it is generated by Bundler using the local gem cache. (Ignore it in `.gitignore`.)

### 5. Use the gems

To use the gems, prefix the commands you would normally use with `bundle exec`. This is a
wrapper that prepares the command environment to use the gems installed in `./vendor/bundle`.
The gems don't have to be installed at system level, and you don't have to muck around
with rvm gemsets.

Example:

```
bundle exec jekyll new example-site
cd example-site
bundle exec jekyll serve
```


## Ongoing workflow

When you need to change or update your gems:

1. Edit your `Gemfile`
2. Run `bundle package` to update the local gem cache. (You don't need the `--all` flag, because
   Bundler remembers the setting in `./.bundle`)
3. Run `bundle install --local` to install the updated gems. (You don't need the `--path` flag,
   because Bundler remembers the setting in `./.bundle`)
4. Use the updated gems