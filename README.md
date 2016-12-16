# nipil.org website

# license

Every file of this project is covered by the MIT license, EXCEPT FOR the files in sources/_posts/ and sources/files which are Copyright (C) nipil@users.noreply.github.com

# install

## system packages

  ruby 2.1.5+
  gem

## bundler gem

  gem install --user-install bundler

## website gems

For development

  bundler config path vendor/bundler
  bundler install

For production

  bundler install

# generate

For development

  bundler exec jekyll serve --source sources/

For production

  bundler exec jekyll build --source sources/

