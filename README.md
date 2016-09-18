# ALLIED BUILDINGS WEBSITE

[<img src="https://raw.githubusercontent.com/maddevelopmentco/website.alliedbuildings.com/master/public/img/ASB_Logo_Black_Horizontal.png?token=ARsyU5EQa8bALsIze_HoCDKYnKLSbS-Dks5X5Za4wA%3D%3D" align="right"/>](http://alliedbuildings.com)

# Overview

1.  [Setup & Installation](#overview)
    1.  [Cold deployment](#cold-deployment)
    1.  [Hot deployment](#hot-deployment)
    1.  [Development](#development-setup)
1.  [Page Structure](#new-pages)
    1.  [Outline](#new-pages)
    1.  [Basic Page Setup](#basic-page-setup)
    1.  [Setup with Contentful](#create-new-page-with-contentful)
1.  [Styling](#styling)
    1.  Style Guidelines - *coming soon*
1.  Content with Contentful - *coming soon*
    1.  Setup a Content-Model - *coming soon*
    1.  Add Content/Media Assets - *coming soon*


---

# Cold deployment
###:snowflake: Environment setup for new, clean AWS instances

Install Git (hopefully you have this already).   

```
sudo apt-get install git
```

Install **nvm** for using node without sudo and the ability to switch between node versions.

```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.0/install.sh | bash
```    

Close and reopen your terminal.    
Now we will install and update **Nodejs** and **npm**:    

```
nvm install v4.2.1    
npm i npm -g
```

Clone website source files, change into new directory, and install your node dependencies:    

```
git clone https://github.com/maddevelopmentco/website.alliedbuildings.com.git   
cd website.alliedbuildings.com/    
npm i
```

Run build for front-end scripts, fonts, and styles:    

```
npm run build
```

Update repositories:    

```
sudo apt-get update
```

Install **nginx**:    

```
sudo apt-get install nginx
```

Create config file and open in your editor:    

```
sudo touch /etc/nginx/sites-available/allied    
sudo vim /etc/nginx/sites-available/allied
```

Insert the script below:    

```
upstream alliedsteelbuildings {
    server 127.0.0.1:3000;
    keepalive 8;
}

server {
    listen 0.0.0.0:80;
    server_name alliedsteelbuildings.com alliedsteelbuildings;
    access_log /var/log/nginx/allied.log;

    # pass the request to the node.js server with the correct headers
    # and much more can be added, see nginx config options

    location / {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $http_host;
      proxy_set_header X-NginX-Proxy true;

      proxy_pass http://alliedsteelbuildings/;
      proxy_redirect off;
    }
 }

```

Save and exit    
```
:wq
```

Switch directories and create a shortcut to the new config file:    

```
cd ../sites-enabled    
sudo ln -s /etc/nginx/sites-available/allied allied
```

Remove default shortcut to nginx homepage:    

```
sudo rm default
```

Restart your server and cd into the main website directory:         

```
sudo /etc/init.d/nginx restart    
cd /home/ubuntu/website.alliedbuildings.com/
```

Install **pm2** for managing instances, keeping website alive after crushes:

```
npm install pm2 -g
```

Start the website:

```
pm2 start app.json
```

**Logs are located here**: `/home/ubuntu/.pm2/logs/`    
Live website logs can be viewed by running this script in your terminal:    

```
pm2 logs 0 --timestamp "HH:mm:ss"
```
[![button](https://raw.githubusercontent.com/hkdeven/Be-Constructive/master/top-btn.jpg)](#overview)


---

# Hot Deployment

###:fire: Environment setup for regular updates (update scripts, styles)

If you have not cloned and setup this repo, do so first:

```
git clone https://github.com/maddevelopmentco/website.alliedbuildings.com.git   
cd website.alliedbuildings.com/    
npm i    
npm run build
```

Before any updates are staged or pushed live, remember to start by pulling any recent changed to the master:

```
git pull origin master
```

Restart Instances
```
pm2 restart 0
pm2 restart 1
```

For watching real time logs
```
pm2 logs 0 --timestamp "HH:mm:ss"
```

[![button](https://raw.githubusercontent.com/hkdeven/Be-Constructive/master/top-btn.jpg)](#overview)


---

# Development Setup

###:construction_worker: Environment setup when working in development

If you have not cloned and setup this repo, do so first:    

```
git clone https://github.com/maddevelopmentco/website.alliedbuildings.com.git   
cd website.alliedbuildings.com/    
npm i    
```

Before any updates are staged or pushed live, remember to start by pulling any recent changed to the master:    

```
git pull origin master
```

Start your server.  View at localhost:3000.

```
node index.js
```

[![button](https://raw.githubusercontent.com/hkdeven/Be-Constructive/master/top-btn.jpg)](#overview)


---

# New Pages

New pages can be created following typical steps for React-Redux applications.  Content is hosted and served via the [Contenful](https://github.com/contentful) content manager and api.  However, [Contenful](https://github.com/contentful) is not required to create and launch new pages.  Instructions for both methods can be found below.  Documentation for node-friendly Contentful can be [found here](https://www.contentful.com/developers/docs/javascript/). *Further instructions for uploading content coming soon.*    

####These are the files and directories we will be working with:        


Create New | Update Existing
------------ | ------------
`public/src/components/new-page/` | `routes/server-routes.js`
`public/src/components/new-page/NewPage.jsx` | `routes/content.js`
`public/src/components/new-page/index.jsx` | `public/src/routes.jsx`
`public/styles/src/partials/newpage.scss` |
`public/styles/src/style.scss` |
`utils/loaders/newpage.js` |


---

# Basic Page Setup

In this walk through we will be creating a new page to house our company's testimonials.

###CREATE THE COMPONENT

First, create a new component folder called `testimonials` and the required index and component file for the new page.  Note that component files should be **CamelCase**. :camel:    

```
mkdir public/src/components/testimonials/
touch public/src/components/testimonials/index.jsx public/src/components/testimonials/Testimonials.jsx
```

Within the `index.jsx` file insert this script:    

```javascript
    import React, { PropTypes }  from 'react';
    import Header from '../Header';
    import Footer from '../Footer';   

    class Testimonials extends React.Component {

       render() {
           const {hash, pathname, query, search} = this.props.location;
           const {deviceType, width} = this.props;
           const headerFooterProps = {hash, pathname, query, search, deviceType, width};
           return (
               <div>
                   <div id="wrapper">
                       <Header {...headerFooterProps}/>
                       <div>
                            <p>All my content goes after the div tag above...</p>
                       </div>
                   </div>
                   <Footer {...headerFooterProps}/>
               </div>
           )
       };
     }

     export default MyComponent;
```    

###ADD THE ROUTE    

Next, add a new route (url path) for the page in the `routes/server-routes.js` file:    

``` javascript
router.get('/testimonials', (req, res, next)=> {
    req.isServerRequest = true;
    next();
});
```

Then, in the `public/src/routes.jsx` file, import the `index.jsx` file and add the path of the new component:    

```javascript
import Testimonials from "./components/testimonials/index.jsx";

<Route path="testimonials" component={Testimonials} />
```

[![button](https://raw.githubusercontent.com/hkdeven/Be-Constructive/master/top-btn.jpg)](#overview)


---

# Create New Page with Contentful

### REQUIREMENTS

:small_red_triangle: Note that you must setup your new *content-model* and applicable *content* and *media* in [Contenful](https://github.com/contentful) prior to following the steps below.  :small_red_triangle:


### CREATE THE LOADER

To serve content from Contentful, add a data loader into `utils/loaders` for the testimonial content-model.    

```
touch utils/loaders/testimonials.js
```

Then insert the script below.  Note that your content-model (***testimonials***) should be called using **kebabCase**.  :oden:    

```javascript
import client from '../contentful/contentfulClient';

export default (req, res, next) => {
    client.getEntries({
        content_type: 'testimonials',
        limit: 1,
        include:3
    })
    .then(testimonials=> {
        let exampleType = cmsExampleType.toPlainObject().items[0];

        if (req.isServerRequest) {
            req.dependentEntries = [{exampleType}];
            return next();
        } else {
            return res.json({exampleType});
        }
    })
}
```

This allows us to fetch the stored content. In this example the key is `exampleType`.  This is the key that is called to render this content on the front-end, in the component `index.jsx` file.    

The data/dependency response will look something like this, depending on your content-model structure:

```javascript
{
  exampleType: {
      sys: {
          //contentful system fields
      },
      fields: {
          //predefined fields from cms
      }
  }
}
```

If this request is the first for the user and markup is needed (in addition to the json/data) all data will be stored in the request and populated in the Redux store in the next nodeJS middleware.      

###ADD THE COMPONENT

Add simple React Component into front-end part in folder: `public/src/components/new-example-page`    

```javascript
import React, { PropTypes, Component }  from 'react';
import Header from '../Header';
import Footer from '../Footer';
import * as Actions from '../../actions/Actions';
import { connect } from 'react-redux';

class Testimonials extends Component {
        static propTypes = {
            content: PropTypes.object.isRequired,
            dispatch: PropTypes.func.isRequired
        };

        componentDidMount() {
            window.scrollTo(0, 0);

            //key which we try to retrieve from the redux store should be same as the key we defined in the step 1
            if (!this.props.content.testimonials) {

                //string into getContent function has to be same as the route which we defined in the step 3
                this.props.dispatch(Actions.getContent('testimonials'));
            }
        }

        render() {
            if (!this.props.content.testimonials) return (<noscript />);
            const {hash, pathname, query, search} = this.props.location;
            const {deviceType, width} = this.props;
            const headerFooterProps = {hash, pathname, query, search, deviceType, width};
            return (
                <div>
                    <div id="wrapper">
                        <Header {...headerFooterProps}/>
                        <div>this.props.content.testimonials.fields.myFieldFromCms</div>
                    </div>
                    <Footer {...headerFooterProps}/>
                </div>
            )
        }
    }

export default connect(state => ({content: state.content}))(TestimonialsContainer)
```

###ADD THE ROUTES    

Next, add a new route (url path) for the page in the `routes/server-routes.js` file:    

```javascript
router.get('/testimonials', (req, res, next)=> {
        req.isServerRequest = true;
        testimonials(req, res, next);
        });
```

In the same file, import the new loader:    

```javascript
import testimonials from "../utils/loaders/testimonials";
```

Then, in the `routes/content.js` file add the route:    

```javascript
router.get('/testimonials', (req, res, next) => testimonials(req, res, next));
```    

Finally, in the `public/src/routes.jsx` file, import the `index.jsx` file and add the path of the new component:    

```javascript
import Testimonials from "./components/testimonials/index.jsx";
//further down include:
<Route path="testimonials" component={Testimonials} />
```

[![button](https://raw.githubusercontent.com/hkdeven/Be-Constructive/master/top-btn.jpg)](#overview)


---

# Styling

:warning: ***DO NOT MODIFY EXISTING STYLESHEETS TO STYLE NEW PAGES*** :warning:    

Stylesheets, fonts, and vendor styles are are located in the `public/styles` folder.  Our stylesheets use [SASS](http://sass-lang.com/) to preprocess. :nail_care:    

When creating new pages, it is recommended to follow the React-style and create new stylesheets specific to the new page.  Instead, add your page-specific styles to a new **.scss** file like so:

```
touch public/styles/src/testimonials.scss
```

When completed, be sure to import this new stylesheet into the master:

```
echo "@import 'testimonials';" >> public/styles/src/style.scss
```



[![button](https://raw.githubusercontent.com/hkdeven/Be-Constructive/master/top-btn.jpg)](#overview)


About Allied
----------------

![Allied](https://raw.githubusercontent.com/maddevelopmentco/website.alliedbuildings.com/master/public/img/ASB_Logo_Black_Horizontal.png?token=ARsyU5EQa8bALsIze_HoCDKYnKLSbS-Dks5X5Za4wA%3D%3D)

[Allied][allied] is a global leader in steel construction, developing solutions for every industry. From aviation to warehouses and everything in between, trust [Allied][allied] to be with you all the way.    

The names and logos for Allied are trademarks of [Allied Steel Buildings, Inc.][allied]  We love open source software. :octocat:

[allied]: http://alliedbuildings.com/
