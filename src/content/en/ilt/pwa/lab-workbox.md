project_path: /web/_project.yaml
book_path: /web/ilt/pwa/_book.yaml

{# wf_auto_generated #}
{# wf_updated_on: 2018-08-28 #}
{# wf_published_on: 2016-01-01 #}


# Lab: Workbox {: .page-title }




<div id="overview"></div>


## Overview




[Workbox](https://workboxjs.org/) is the successor to  [`sw-precache`](https://github.com/GoogleChrome/sw-precache) and  [`sw-toolbox`](https://github.com/GoogleChrome/sw-toolbox). It is a collection of libraries and tools used for generating a service worker, precaching, routing, and runtime-caching. Workbox also includes modules for easily integrating  [background sync](/web/tools/workbox/modules/workbox-background-sync) and  [Google Analytics](/web/tools/workbox/modules/workbox-google-analytics) into your service worker.

In this lab, you'll use the __workbox-sw.js__ library and the `workbox-build` Node.js module to build an offline-capable progressive web app (PWA).

#### What you'll learn

* How to write a service worker using the __workbox-sw.js__ library
* How to add routes to your service worker using __workbox-sw.js__
* How to use the predefined caching strategies provided in __workbox-sw.js__
* How to augment the __workbox-sw.js__ caching strategies with custom logic
* How to generate a production-grade service worker with `workbox-build`

#### What you should already know

* Basic HTML, CSS, and JavaScript
* ES2015 Promises
* How to run commands from the command line
* Some familiarity with service workers is recommended
* Familiarity with Node.js and gulp is recommended

#### What you will need

* Computer with terminal/shell access
* Connection to the internet
* A  [browser that supports service worker](https://jakearchibald.github.io/isserviceworkerready/)
* A text editor
*  [Node.js](https://nodejs.org/en/) and  [npm](https://www.npmjs.com/)

<div id="1"></div>


## 1. Get set up




If you have not downloaded the repository and installed the  [LTS version of Node.js](https://nodejs.org/en/), follow the instructions in [Setting up the labs](setting-up-the-labs).

### 1.1 Install project dependencies and start the server

Navigate to the __project__ directory via the command line:

    cd workbox-lab/project/

Run the following command to install the project dependencies:

    npm install

Then build and serve the app with the following command:

    npm run serve

You can terminate the `serve` process at any time with `Ctrl-c`.

#### Explanation

The `npm install` command installs the project dependencies based on the configuration in __package.json__. Open __project/package.json__ and examine its contents.

The important package is  [`workbox-build`](/web/tools/workbox/guides/precache-files/workbox-build), which is a module used to generate a list of assets that should be precached in a service worker. You'll see how that's used in a later step.

Also note that the `serve` script aliases the `gulp serve` command.

The remaining dependencies in __package.json__ are used by gulp in __gulpfile.js__.

Open __gulpfile.js__ and examine its contents. The gulp `build` task "builds" the app; in this case by copying the __src__ files into a __build__ directory (using the `clean` and `copy` tasks). The `serve` task builds the app (with the build task) and then the __build__ directory served and `watch`ed. The `watch` functionality ensures that the app is rebuilt whenever __src__ files are updated.

Since we ran the gulp `serve` task with `npm run serve`, the app should be built and served on __localhost:8081/__.

Why didn't we just run the `gulp serve` task directly? First, running gulp from the command line requires a global installation of the gulp command line tool. Second, relying on a global installation gives us less control over the gulp version used since __package.json__ only specifies the local gulp version, but not any global versions. Aliasing `gulp` commands with `npm scripts` in __package.json__ allows us to work around these issues and use the local gulp package directly.

### 1.3 Open the app and explore the code

Open the app in your browser by navigating to __localhost:8081/__. The app is a news site containing some "trending articles" and "archived posts". We will be performing different runtime caching strategies based on whether the request is for a trending article or archived post.

Note: [Unregister](tools-for-pwa-developers#unregister) any service workers and [clear all service worker caches](tools-for-pwa-developers#clearcache) for localhost so that they do not interfere with the lab. In Chrome DevTools, you can achieve this by clicking __Clear site data__ from the __Clear storage__ section of the __Application__ tab.

Open the __workbox-lab/project__ folder in your text editor. The __project__ folder is where you will be building the lab.

This folder contains:

* __src/images__ folder contains sample images
* __src/js/animation.js__ is a javascript file for page animations
* __src/pages__ folder contains sample pages
* __src/style/main.css__ is the stylesheet for the app's pages
* __src/sw.js__ is where we will write our source service worker using __workbox-sw.js__
* __src/index.html__ is the home page HTML file
* __gulpfile.js__ configures gulp tasks
* __package.json__ and __package-lock.json__ track Node.js dependencies

All of the __src__ files were also copied over to the __build__ folder as mentioned in the previous section.

<div id="2"></div>


## 2. Write a basic service worker using workbox-sw




Now that we have the starting app working, let's start writing the service worker.

In the empty source service worker file, __src/sw.js__, add the following snippet:

#### src/sw.js

```
importScripts('https://storage.googleapis.com/workbox-cdn/releases/3.4.1/workbox-sw.js');

if (workbox) {
  console.log(`Yay! Workbox is loaded 🎉`);

  workbox.precaching.precacheAndRoute([]);

} else {
  console.log(`Boo! Workbox didn't load 😬`);
}
```

Save the file. Observe that our production service worker (__build/sw.js__) was automatically updated by gulp `watch`.

#### Explanation

In this code, the `importScripts` call imports the __workbox-sw.js__ library from a Content Delivery Network (CDN). Once the library is loaded, the `workbox` object gives our service worker access to all the  [Workbox modules.](/web/tools/workbox/modules/)

The `precacheAndRoute` method of the `precaching` module takes a precache "manifest" (a list of file URLs with "revision hashes") to cache on service worker installation. It also sets up a cache-first strategy for the specified resources, serving them from the cache by default.

Currently, the array is empty, so no files will be cached.

Rather than adding files to the list manually, `workbox-build` can generate the manifest for us. Using a tool like `workbox-build` has multiple advantages:

1. The tool can be integrated into our build process. Adding `workbox-build` to our build process eliminates the need for manual updates to the precache manifest each time that we update the apps files.
2. `workbox-build` automatically adds "revision hashes" to the files in the manifest entries. The revision hashes enable Workbox to intelligently track when files have been modified or are outdated, and automatically keep caches up to date with the latest file versions. Workbox can also remove cached files that are no longer in the manifest, keeping the amount of data stored on a user's device to a minimum. You'll see what `workbox-build` and the file revision hashes look like in the next section.

#### Learn more

While we are using  [workbox-build](/web/tools/workbox/guides/precache-files/workbox-build) with gulp in this lab, Workbox also supports tools like webpack with  [workbox-webpack-plugin](/web/tools/workbox/guides/precache-files/webpack) and npm-build processed with  [workbox-cli](/web/tools/workbox/guides/precache-files/cli).

<div id="3"></div>


## 3. Inject a manifest into the service worker




Now we need to configure `workbox-build` to inject a precache manifest in the `precacheAndRoute` call in the service worker file.

Add code to include the `workbox-build` module into __gulpfile.js__:

#### gulpfile.js

```
const workboxBuild = require('workbox-build');
```

Then add a `build-sw` task in __gulpfile.js__ (above the `build` task) to generate and inject the precache manifest:

#### gulpfile.js

```
const buildSw = () => {
  return workboxBuild.injectManifest({
    swSrc: 'src/sw.js',
    swDest: 'build/sw.js',
    globDirectory: 'build',
    globPatterns: [
      '**\/*.css',
      'index.html',
      'js\/animation.js',
      'images\/home\/*.jpg',
      'images\/icon\/*.svg',
    ]
  }).then(resources => {
    console.log(`Injected ${resources.count} resources for precaching, ` +
        `totaling ${resources.size} bytes.`);
  }).catch(err => {
    console.log('Uh oh 😬', err);
  });
}
gulp.task('build-sw', buildSw);
```

Finally, update the `build` task to include the `build-sw` task in its `series`. The updated `build` task should be as follows:

#### gulpfile.js

```
const build = gulp.series('clean', 'copy', 'build-sw');
gulp.task('build', build);
```

Since we've updated __gulpfile.js__, we need to rerun the gulp `serve` task in the command line. Terminate the `serve` process with `Ctrl+c` and then restart it with `npm run serve`.

Observe that the `precacheAndRoute` call in the production service worker (__build/sw.js__) has been updated with the precache manifest, and should look like this (your hash values may be different):

#### build/sw.js

```
workbox.precaching.precacheAndRoute([
  {
    "url": "style/main.css",
    "revision": "919792b0fc1138b73c7553f56cae2c41"
  },
  {
    "url": "index.html",
    "revision": "49dc015edffceba2a88e909b944675c0"
  },
  // ...
]);
```

#### Explanation

The `build-sw` task uses the `workbox-build` module's  [`injectManifest`](/web/tools/workbox/reference-docs/prerelease/module-workbox-build.html#.injectManifest) method. This method copies a source service worker file, specified in `swSrc`, to an output specified by `swDest`. Workbox searches the new service worker for an empty `precacheAndRoute()` call and populates the empty array with the assets defined in `globPatterns`. Adding the `build-sw` task to the `build` task ensures that the production service worker is regenerated anytime the __src__ files change.

#### Learn more

[Precaching documentation](/web/tools/workbox/modules/workbox-precaching)

<div id="4"></div>


## 4. Register and test the service worker




Now add a script in the bottom of __index.html__ (just before the closing `</body>` tag) to register the new service worker:

#### index.html

```
  <script>
    if ('serviceWorker' in navigator) {
      window.addEventListener('load', () => {
        navigator.serviceWorker.register('/sw.js')
          .then(registration => {
            console.log('Service Worker registered! 😎');
          })
          .catch(err => {
            console.log('Registration failed 😫 ', err);
          });
      });
    }
  </script>
```

Return to the app and [unregister](tools-for-pwa-developers#unregister) any existing service workers and [clear all service worker caches](tools-for-pwa-developers#clearcache). In Chrome DevTools, you can do this in one easy operation by going to the __Application__ tab, clicking __Clear Storage__ and then clicking the __Clear site data__ button.



__Tip:__ You can automatically activate updated service workers in Chrome Developer Tools by selecting __Update on reload__ in the __Server Workers__ tab.



Refresh the page and check the developer console; you should see the registration success message. You can also use developer tools to check that [status of the service worker](tools-for-pwa-developers#accesssw). In Chrome DevTools, you can find service workers in the __Service Worker__ section of the __Application__ tab.

Next, observe that the resources specified in the precache manifest have [been cached](tools-for-pwa-developers#inspect-cache-storage). In Chrome DevTools, you can see your caches by expanding the __Cache Storage__ section in the __Application__ tab.



Note: In Chrome, the DevTools Cache interface is not always updated automatically, even if new files are cached. If you don't see the cached files, right-click __Cache Storage__ in the __Application__ tab and choose __Refresh Caches__. If a similar method is not available in your browser, then try reloading the page.



Now go "offline" by using `Ctrl+c` to terminate the gulp server. Refresh the app in the browser. The home page should load even though we are offline!



Note: In Chrome, you may see a console error indicating that the service worker could not be fetched: `An unknown error occurred when fetching the script. service-worker.js Failed to load resource: net::ERR_CONNECTION_REFUSED`. This error is shown because the browser couldn't fetch the service worker script (because the site is offline), but that's expected because we can't use the service worker to cache itself. Otherwise the user's browser would be stuck with the same service worker forever!



#### Explanation

In addition to precaching, the `precacheAndRoute` method sets up an implicit cache-first handler, ensuring that the precached resources are served offline.



Note: Workbox also handles an edge case that we haven't mentioned - Workbox knows to serve __my-domain/index.html__ even if __my-domain/__ is requested! With this functionality, you don't have to manage multiple cached resources (one for __index.html__ and one for __/__).



<div id="5"></div>


## 5. Add routes to the service worker




__workbox-sw.js__ has a  [`routing`](/web/tools/workbox/modules/workbox-routing) module that lets you easily add routes to your service worker.

Let's add a route to the service worker now. Copy the following code into __src/sw.js__ beneath the `precacheAndRoute` method. Make sure you're not editing the production service worker, __build/sw.js__, as this file will be overwritten by our build process.

#### src/sw.js

```
workbox.routing.registerRoute(
  /(.*)articles(.*)\.(?:png|gif|jpg)/,
  workbox.strategies.cacheFirst({
    cacheName: 'images-cache',
    plugins: [
      new workbox.expiration.Plugin({
        maxEntries: 50,
        maxAgeSeconds: 30 * 24 * 60 * 60, // 30 Days
      })
    ]
  })
);
```

Save the file.

Rebuild the app (including the service worker) and restart the server with the following command:

```
npm run serve
```

Refresh the app and [activate the updated service worker](tools-for-pwa-developers#activate) in the browser. In Chrome DevTools you can activate a new service worker by clicking __skipWaiting__ in the __Service Worker__ section of the __Application__ tab. Navigate to __Article 1__ and __Article 2__ . Check the caches to see that the `images-cache` now exists and contains the images from Articles 1 and 2. You may need to refresh the caches in developer tools to see the contents.

#### Explanation

The  [`registerRoute`](/web/tools/workbox/reference-docs/prerelease/workbox.routing#.registerRoute) method on the  [`routing`](/web/tools/workbox/modules/workbox-routing) class adds a route to the service worker. The first parameter in `registerRoute` is a regular expression URL pattern to match requests against. The second parameter is the handler that provides a response if the route matches. In this case the route uses the  [`strategies`](/web/tools/workbox/modules/workbox-strategies) class to access the  [`cacheFirst`](/web/tools/workbox/modules/workbox-strategies#cache_first_cache_falling_back_to_network) run-time caching strategy. This means that whenever the app requests article images, the service worker checks the cache first for the resource before going to the network.

The handler in this code also configures Workbox to maintain a maximum of 50 images in the cache (ensuring that user's devices don't get filled with excessive images). Once 50 images has been reached, Workbox will remove the oldest image automatically. The images are also set to expire after 30 days, signaling to the service worker that the network should be used for those images.

__Optional:__ Write your own route that caches the user avatar. The route should match requests to `/images/icon/*` and handle the request/response using the  [`staleWhileRevalidate`](/web/tools/workbox/modules/workbox-strategies#stale-while-revalidate) strategy. Give the cache the name "icon-cache" and allow a maximum of 5 entries to be stored in the cache. This strategy is good for icons and user avatars that change frequently but the latest versions are not essential to the user experience. You'll need to remove the icon from the precache manifest so that the service worker uses your `staleWhileRevalidate` route instead of the implicit cache-first route established by the `precache` method.

<div id="6"></div>


## 6. Use a customized networkFirst cache strategy




### 6.1 Write the basic handler using the `handle` method

Sometimes content must always be kept up-to-date (e.g., news articles, stock figures, etc.). For this kind of data, the `cacheFirst` strategy is not the best solution. Instead, we can use the `networkFirst` strategy to fetch the newest content first, and only if that fails does the service worker get old content from the cache.

Copy the following code below the existing route in __src/sw.js__:

#### src/sw.js

```
const articleHandler = workbox.strategies.networkFirst({
  cacheName: 'articles-cache',
  plugins: [
    new workbox.expiration.Plugin({
      maxEntries: 50,
    })
  ]
});

workbox.routing.registerRoute(/(.*)article(.*)\.html/, args => {
  return articleHandler.handle(args);
});
```

Save the file, refresh the app, and update the service worker in the browser. Reload the home page and click a link to one of the Trending Articles. Check the caches to see that the `articles-cache` was created and contains the article you just clicked. You may need to refresh the caches to see the changes.

__Optional:__ Test that articles are cached dynamically by visiting some while online. Then take the app offline again by pressing `Ctrl+C` in the command line and re-visit those articles. Instead of the browser's default offline page, you should see the cached article. Remember to re-run `npm run serve` to restart the server.

__Optional:__ Test the `networkFirst` strategy by changing the text of the cached article and reloading the page. Even though the old article is cached, the new one is served and the cache is updated.

#### Explanation

Here we are using the `networkFirst` strategy to handle a resource we expect to update frequently (trending news articles). This strategy updates the cache with the newest content each time it's fetched from the network.

Unlike the images route established in the previous step, the code above uses the  [`handle`](/web/tools/workbox/reference-docs/prerelease/workbox.strategies.NetworkFirst#handle) method on the built-in `networkFirst` strategy. The `handle` method takes the object passed to the handler function (in this case we called it `args`) and returns a promise that resolves with a response. We could have passed in the caching strategy directly to the second argument of `registerRoute` as we did in the previous example, but instead we return a call to the `handle` method in a custom handler function. This technique gives us access to the response, as you'll see in the next step.

### 6.2 Handle invalid responses

The `handle` method returns a promise resolving with the response, so we can access the response with a `.then`.

Add the following `.then` inside the `article` route after the call to the `handle` method:

#### sw.js

```
.then(response => {
    if (!response) {
      return caches.match('pages/offline.html');
    } else if (response.status === 404) {
      return caches.match('pages/404.html');
    }
    return response;
  });
```

Then add __pages/offline.html__ and __pages/404.html__ to the `globPatterns` in the `build-sw` task defined in __gulpfile.js__. The updated `build-sw` task should look like this:

#### workbox-config.js

```
const buildSw = () => {
  return workboxBuild.injectManifest({
    swSrc: 'src/sw.js',
    swDest: 'build/sw.js',
    globDirectory: 'build',
    globPatterns: [
      '**\/*.css',
      'index.html',
      'js\/animation.js',
      'images\/home\/*.jpg',
      'images\/icon\/*.svg',
      'pages/offline.html',
      'pages/404.html'
    ]
  }).then(resources => {
    console.log(`Injected ${resources.count} resources for precaching, ` +
        `totaling ${resources.size} bytes.`);
  }).catch(err => {
    console.log('Uh oh 😬', err);
  });
}
gulp.task('build-sw', buildSw);
```

Save the files. Restart gulp by terminating the `serve` process with `Ctrl+c` and re-running `npm run serve`. Refresh the app activate the updated service worker in the browser. On the app home page, try clicking the __Non-existent article__ link. This link points to an HTML page, __pages/article-missing.html__, that doesn't actually exist. You should see the custom 404 page that we precached!

Now try taking the app offline by pressing `Ctrl+C` in the command line and then click any of the links to unvisited articles. You should see the custom offline page!



Note: If you've already cached all the articles, you can clear the caches and refresh the page in order to test the offline page.



#### Explanation

The `.then` statement receives the response passed in from the handle method. If the response doesn't exist, then it means the user is offline and the response was not previously cached. Instead of letting the browser show a default offline page, the service worker returns the custom offline page that was precached. If the response exists but the status is 404, then our custom 404 page is returned. Otherwise, we return the original response.

<div id="7"></div>


## 7. Optional: Add a customized cacheFirst cache strategy




So far you have implemented many caching strategies with Workbox, and in the previous section you learned how to add custom logic to the default Workbox caching options.

This last exercise is a challenge with less guidance (you can still see the solution code if you get stuck). You need to:

* Add a final service worker route for the app's "post" pages (__pages/post1.html__, __pages/post2.html__, etc.).
* The route should use Workbox's `cacheFirst` strategy, creating a cache called "posts-cache" that stores a maximum of 50 entries.
* If the cache or network returns a resources with a 404 status, then return the  __pages/404.html__ resource.
* If the resource is not available in the cache, and the app is offline, then return the  __pages/offline.html__ resource.

#### Hints

* Review the previous section for how to add additional logic to Workbox's built-in strategy handlers.
* The Workbox `cacheFirst` handler handles no-connectivity differently than `networkFirst`: instead of passing an empty response to the `.then`, it  *rejects*  and goes to the next `.catch`.

<div id="congrats"></div>


## Congratulations!




You have learned how to use Workbox to create production-ready service workers!

### What we've covered

* Writing a service worker using the __workbox-sw.js__ library
* Adding routes to your service worker using __workbox-sw.js__
* Using the predefined caching strategies provided in __workbox-sw.js__
* Augmenting the __workbox-sw.js__ caching strategies with custom logic
* Generating a production-grade service worker with `workbox-build`

### Resources

*  [Workboxjs.org](https://workboxjs.org/)
*  [Workbox](/web/tools/workbox/) - developers.google.com


