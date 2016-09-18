# website.alliedbuildings.com

## Setup environment for cold deploy
###Use this setup for new clean AWS instance 

First of all install git
```
sudo apt-get install git
```
Install nvm for using node without sudo & have ability to switch between node versions. 
Gives flexibility in further site updates
```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.0/install.sh | bash
```

Close and reopen your terminal to start using nvm
Install NodeJs using `nvm`
```
nvm install v4.2.1
```
Update `npm` to latest version
```
npm i npm -g
```
Clone website source files
```
git clone https://github.com/maddevelopmentco/website.alliedbuildings.com.git
```

Paste credentials
```
Username for 'https://github.com':
```
```
Password for 'https://username@github.com':
```

Enter source directory
```
cd website.alliedbuildings.com/
```
Install node dependencies
```
npm i
```
Build frontend part (scripts, fonts, styles)
```
npm run build
```

Update repositories
```
sudo apt-get update
```

Install **nginx**
```
sudo apt-get install nginx
```

Add configuration file for website
```
sudo touch /etc/nginx/sites-available/allied
```

Enter config file edit mode
```
sudo vim /etc/nginx/sites-available/allied
```

Basic nginx config, simply copy paste into config file

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
Save & exit `:wq`

Change dir
```
cd ../sites-enabled
```

Create shortcut for config file
```
sudo ln -s /etc/nginx/sites-available/allied allied
```

Remove shortcut for default **nginx** homepage
```
sudo rm default
```

Restart nginx:
```
sudo /etc/init.d/nginx restart
```

Enter website directory
```
cd /home/ubuntu/website.alliedbuildings.com/
```

Install **pm2** for managing instances and keep website alive after crushes
```
npm install pm2 -g
```

Start website with all stuff
```
pm2 start app.json
```

You can watch live website logs using
```
pm2 logs 0 --timestamp "HH:mm:ss"
```

Logs are here:
/home/ubuntu/.pm2/logs/
```

## Setup environment for hot deploy
###Use this for usual updates (update scripts, styles)
Enter project directory. 
Pull changes from `git` using this command:

```
git pull origin master
```

Enter credentials...

Install new `npm` modules (if necessary or skip if there are no changes on the back-end part)
```
npm i
```

Build front-end part
```
npm run build
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

## Setup environment for development

Clone repo 
```
git clone https://github.com/maddevelopmentco/website.alliedbuildings.com.git
```
Or download zip archive

Install npm dependencies 
```
npm i
```
Start project

```
node index.js


## Adding simple CMS page

#### Requirements

1. Ready content-type in the CMS and published Entry of new content-type

#### Steps

1. add data loader into `utils/loaders` for new Entry
    ```
    import client from "../contentful/contentfulClient";
    
    export default (req, res, next) => {
        client.getEntries({
            content_type: 'exampleType',
            limit: 1,
            include: 3
        })
        .then(cmsExampleType=>{
            let exampleType = cmsExampleType.toPlainObject().items[0];
            
            if (req.isServerRequest) {
                req.dependentEntries = [{exampleType}];
                return next();
            } else {
                return res.json({exampleType});
            }
    }
    ```
    Here we fetching stored entry. If this request is the first for the user and he needs markup (not only json with data) all data will be stored in the request and populate redux store in the next nodeJS middleware.
    else we simply return json with data. One very important thing is that data should has a key which you can call on the front-end part.
    In this example key is `exampleType`.
    So you response with data or dependency in the request should look like
    ```
    {
        exampleType: {
            sys: {
                //contentful system fields
            },
            fields: {
                //your predefiend fields from cms
            }
        }
    }
    ```
2. Add new route into backend part into `routes/server-routes.js` which will call just created in step 1 loader. Don't forget to import loader `import exampleLoader from '../utils/loaders/exampleLoader';`

    ```
    router.get('/example-route', (req, res, next)=> {
        req.isServerRequest = true;
        exampleLoader(req, res, next);
    });
    ```
    
    In this file stored all routes which user directly call from the browser search bar, or click on a link in the third-party website or ad
3. Add new route into `routes/content.js`.
    Also don't forget to import loader `import exampleLoader from '../utils/loaders/exampleLoader';`
    
    ```
    router.get('/example', (req, res, next) => exampleLoader(req, res, next));
    ```
    
    Routes from this file call from the our front-end part.
4. Add simple React Component into front-end part in folder: `public/src/components/new-example-page`

    ```
    import React, {PropTypes, Component }  from 'react';
    import Header from '../Header';
    import Footer from '../Footer';
    import * as Actions       from '../../actions/Actions';
    import { connect }            from 'react-redux';
    
    class Example extends Component {
        static propTypes = {
            content: PropTypes.object.isRequired,
            dispatch: PropTypes.func.isRequired
        };
    
        componentDidMount() {
            window.scrollTo(0, 0);
            
            //key which we try to retrieve from the redux store should be same as the key we defined in the step 1
            if (!this.props.content.exampleType) {
            
                //string into getContent function has to be same as the route which we defined in the step 3
                
                this.props.dispatch(Actions.getContent('example')); 
            }
        }
    
        render() {
            if (!this.props.content.exampleType) return (<noscript />);
            const {hash, pathname, query, search} = this.props.location;
            const {deviceType, width} = this.props;
            const headerFooterProps = {hash, pathname, query, search, deviceType, width};
            return (
                <div>
                    <div id="wrapper">
                        <Header {...headerFooterProps}/>
                        <div>this.props.content.exampleType.fields.myFieldFromCms</div>
                    </div>
                    <Footer {...headerFooterProps}/>
                </div>
            )
        }
    }
    
    export default connect(state => ({content: state.content}))(SpecialsContainer)
    ```
    
5. Add new route into front-end part into `public/src/routes.jsx`

    ```
    import MyNewExmapleComponent from "./components/new-example-page";
    
    <Route path="example-route" component={MyNewExmapleComponent}>
    ```

The route should match the route from step 2

## Adding simple page

1. Add new route into backend part in the existed file: `routes/server-routes.js`;

    ```
    router.get('/path-to-the-new-page', (req, res, next)=> {
        req.isServerRequest = true; // This parameter is required! Please, don't forget it.
        next();
    });
    ```
    
2. Create folder for component into `public/src/components` and in it create `index.js`. So final path to this file should looks like: `public/src/component/my-new-component/index.js`
Simple component should looks like:
   ```
   import React, {PropTypes}  from 'react';
   import Header from '../Header';
   import Footer from '../Footer';   
   
   class MyComponent extends React.Component {
   
       render() {
           const {hash, pathname, query, search} = this.props.location;
           const {deviceType, width} = this.props;
           const headerFooterProps = {hash, pathname, query, search, deviceType, width};
           return (
               <div>
                   <div id="wrapper">
                       <Header {...headerFooterProps}/>
                       <div>
                            New Content
                       </div>
                   </div>
                   <Footer {...headerFooterProps}/>
               </div>
           )
       };
   }
   
   export default MyComponent;
   ```
3. Add new route into frontend part in the existed file: `public/src/routes.jsx`
    ```
    //Please, don't forget to import just created component
    import MyComponent from './components/my-new-component';
    
    //path should be the same as in the step 1
    <Route path="path-to-the-new-page" component={MyComponent} />
    
    ```
    
    
## Styling Guide

We are using [SASS](http://sass-lang.com/) preprocessor.
All stylesheets are present in the `public/styles` folder. Also fonts and vendor stylesheets present here.
You are free to define own stylesheets in the `public/src` folder. 
After adding new `*.scss` or `*.css` file don't forget to add it into `public/src/style.scss` file using
command like this `@import "new-file";`. All other stuff will make `webpack` for you.

