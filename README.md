# I-D

[![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/stain-s/I-D?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

My attempt to write Internet Drafts

To build, you need `make`, `xml2rfc` and 
[https://github.com/cabo/kramdown-rfc2629](https://github.com/cabo/kramdown-rfc2629)

In Ubuntu, do:

    sudo apt install xml2rfc
    sudo gem install kramdown-rfc2629

(The Ubuntu package ruby-kramdown-rfc2629 seems to be outdated).


To build, try:

    make app.txt app.html
    

## draft-soilandreyes-arcp

[arcp](arcp) is the source code for
[draft-soilandreyes-app](https://tools.ietf.org/html/draft-soilandreyes-arcp),
previously at [app](app)

