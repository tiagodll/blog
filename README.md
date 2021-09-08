# Tiago on tech
this is my blog about thech stuff
does it make sense to continue with this?

not sure, but well... I write it anyway

# How to deploy the content
```bash
cd ~/code/blog
rm -rf public
hugo
```
then connect to the server
```bash
sudo cp -r *.* /var/www/blog/
```
