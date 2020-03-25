---
layout: post
title: Mac re-install command under terminal(My development tool)
date: 2018-02-07 13:00:06
categories: I.T.
tags: [I.T.]
---
```console
#install brew. check https://brew.sh/
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
#install python.
brew update
easy_install pip
brew install python3
#install scipy
brew install gcc
sudo pip install --upgrade numpy
sudo pip install scipy
sudo pip3 install --upgrade numpy
sudo pip3 install scipy
sudo pip install pandas
sudo pip3 install pandas
#running python2&python3 under jupyter notebook. check http://lee-w-blog.logdown.com/posts/306983-used-in-the-jupyter-python2-python3
sudo pip install ipython notebook
sudo pip3 install ipython notebook
sudo ipython kernelspec install-self
sudo ipython3 kernelspec install-self
#install java. check https://stackoverflow.com/questions/24342886/how-to-install-java-8-on-mac
brew update
brew cask install java
#install node.js
brew update
brew install node
#install hexo-cli. check https://hexo.io/
npm install hexo-cli -g
```
