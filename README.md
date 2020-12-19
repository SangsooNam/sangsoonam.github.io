### Setup Environment

Ruby
```bash
brew install rbenv 

echo 'if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi' >> ~/.zshrc

rbenv install 2.7.2
rbenv global 2.7.2
```

Jekyll
```bash
gem install bundler jekyll
```

Install plugins
```bash
bundle
```

Run
```bash
bundle exec jekyll serve
```