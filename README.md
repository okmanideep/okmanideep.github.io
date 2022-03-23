To update the site
```sh
$ sudo apt install bundler # install bundler
$ bundle install # install ruby gems

# Clone the built site into _site folder
$ git clone -b master git@github.com:okmanideep/okmanideep.github.io _site

$ jekyll serve # locally serve the site
# locally check if things are working

$ jekyll build # build the site into `_site`
```
