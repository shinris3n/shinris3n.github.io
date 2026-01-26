source "https://rubygems.org"

# 1. THE CORE: Use "github-pages" instead of "jekyll". 
# This ensures your build environment matches GitHub's exactly.
gem "github-pages", group: :jekyll_plugins

# 2. YOUR THEME: Keep this explicitly to ensure it loads locally.
# We remove the strict version number to allow it to find a compatible match
# with the modern github-pages gem.
gem "jekyll-theme-hacker"

# 3. YOUR PLUGINS: Keep your existing plugins
group :jekyll_plugins do
  gem "jekyll-feed"
end

# 4. COMPATIBILITY: Keep these for robustness (Docker/Windows)
gem "tzinfo-data", platforms: [:mingw, :mswin, :x64_mingw, :jruby]
gem "wdm", "~> 0.1.0", :install_if => Gem.win_platform?


# REPLACED IN JAN 2026
#source "https://rubygems.org"

# Hello! This is where you manage which Jekyll version is used to run.
# When you want to use a different version, change it below, save the
# file and run `bundle install`. Run Jekyll with `bundle exec`, like so:
#
#     bundle exec jekyll serve --livereload
#
# This will help ensure the proper Jekyll version is running.
# Happy Jekylling!
# gem "jekyll", "~> 3.8.6"

# This is the default theme for new Jekyll sites. You may change this to anything you like.
#gem "minima", "~> 2.0"
#gem "jekyll-theme-hacker", "~> 0.1.1"

# If you want to use GitHub Pages, remove the "gem "jekyll"" above and
# uncomment the line below. To upgrade, run `bundle update github-pages`.
#gem "github-pages", group: :jekyll_plugins

# If you have any plugins, put them here!
#group :jekyll_plugins do
#  gem "jekyll-feed", "~> 0.6"
#end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
#install_if -> { RUBY_PLATFORM =~ %r!mingw|mswin|java! } do
#  gem "tzinfo", "~> 1.2"
#  gem "tzinfo-data"
#end

# Performance-booster for watching directories on Windows
#gem "wdm", "~> 0.1.0", :install_if => Gem.win_platform?

