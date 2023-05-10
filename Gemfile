source "https://rubygems.org"

# To upgrade GitHub pages, run `bundle update github-pages`.
gem "github-pages", group: :jekyll_plugins

# If you have any plugins, put them here!
group :jekyll_plugins do
  gem "webrick", "~> 1.8"
  gem "jekyll-feed"
  gem "jekyll-seo-tag", "~> 2.8"
  gem "jekyll-paginate-v2", "~> 3.0"
  gem 'jekyll-sitemap', '~> 1.4'
  gem 'jemoji'
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
install_if -> { RUBY_PLATFORM =~ %r!mingw|mswin|java! } do
  gem "tzinfo", "~> 1.2"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1.0", :install_if => Gem.win_platform?