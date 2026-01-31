# Create a Jekyll container for GitHub Pages
# See this for current supported versions: https://pages.github.com/versions/
FROM ruby:3.3

# Update the Ruby bundler
RUN gem update bundler 

# Install Jekyll
RUN gem install bundler jekyll

# Add webricks
# See https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll
RUN bundle add webricks