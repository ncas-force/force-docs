Introduction to the FORCE website. 
==================================

The FORCE website is built using React, a JavaScript library designed for creating user interfaces, particularly for web applications.

One of React's key features is its component-based architecture, which makes development more modular and maintainable.

FORCE uses the Gatsby framework, which provides significant advantages in terms of performance and SEO optimisation.

Builing the site with npm
-------------------------


To build the site clone the git repo and then run 

```npx gatsby build --prefix-paths```

This will build a local copy of the site in the public directory. 
Copy the contents of this site to the correct location on the web server e.g (/var/www/html/directoryname)

Editing the site
----------------

To edit the site first make your edits and commit them to git. 

Now clean the current build using npm

```npx gatsby clean```

and then follow the above instructions for the build of the site. 



