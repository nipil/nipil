# nipil.org website

# license

Every file of this project is covered by the MIT license
Copyright (C) nipil@users.noreply.github.com

# install

Install system packages

    ruby
    ruby-dev
    gem
    build-essential

Add this to your `~/.bashrc` and logout/login to update your shell

    if which ruby >/dev/null && which gem >/dev/null; then
      PATH="$(ruby -r rubygems -e 'puts Gem.user_dir')/bin:$PATH"
    fi

Install bundler gem in your local profile

    gem install --user-install bundler
    
Define a subdirectory of the repository for local dependancies 
 
    bundle config set --local path 'vendor/bundle'

When you want it you can upgrade your dependencies

    rm -Rf Gemfile Gemfile.lock vendor/
    bundle init
    bundle add jekyll

Generate a new site

    bundle exec jekyll new foo
 
Get the bundle files

    mv foo/Gemfile .
    rm Gemfile.lock
    
Then add your plugins to `Gemfile`

    gem "jekyll-asciidoc"

Install the gems

    bundler install
    bundler clean

# generate

For development

    bundle exec jekyll serve --drafts

For production

    bundle exec jekyll build
