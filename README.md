# nipil.org website

## system packages

  ruby 2.1.5+
  gem

# bundler gem

  gem install --user-install bundler

# website gems

For development

  bundler config path vendor/bundler
  bundler install

For production

  bundler install

# generate site

For development

  bundler exec jekyll serve --source sources/

For production

  bundler exec jekyll build --source sources/

