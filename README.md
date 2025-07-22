ObjectBox C and C++ documentation
=================================
This repository contains the sources of the [ObjectBox C and C++ documentation hosted at cpp.objectbox.io](https://cpp.objectbox.io). It is maintained by the ObjectBox team, but everyone can contribute. E.g. if you found a typo, you can fix it here, e.g. by creating a pull request.

Structure
---------
The **text content** is written in the markdown format and the [docs](docs/) folder contains all the mdx files. Our mdx files are mostly standard markdown, just with some mdx components for tabs section (multiple tabs for different languages etc.).

Other resources:

* **Images** are placed in the [static/img/assets/](static/img/assets/) folder.
* The **main navigation** is located in the [sidebars.ts](sidebars.ts) file.
* The main **configuration** is done in the [docusaurus.config.ts](docusaurus.config.ts) file.
* Custom **CSS styles** to overwrite the theme's default are located in the [src/css/custom.css](src/css/custom.css) file.

Notes regarding image file names: only use lowercase and no spaces.

Static website generation
-------------------------
The website is built with Docusaurus that transforms the mdx files along with the other files into static HTML files.

Since Docusaurus is a Node.js application, you need to install Node.js and run `npm install` to install all dependencies.

Then, you can run `npm run start` to start the development server and open the website in a browser. You can also run `npm run build` to build the static website.
