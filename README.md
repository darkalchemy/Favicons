#Germania\Favicons

**Get your favicons in line with Twig and Slim Framework**




##Where are the templates? The Favicons class
The only purpose of this class is to make the Twig templates that come with this package somehow “installable” with Composer (sort of). Grab the path with the `getTemplatesPath` method. You will need it to setup your template loader. 

```php
<?php
$favicons = new Germania\Favicons\Favicons;
echo $favicons->getTemplatesPath();

// Usually outputs something like
// "vendor/germania/favicons/templates"
```


##Setup Twig Template Loader

Setup a `Favicons` object and add its template path to your Twig template loader:

```php
<?php
// 1. Setup your Favicons object
$favicons = new Germania\Favicons\Favicons;

// 2. Setup Twig, add Favicons' template dir as well
$loader = new Twig_Loader_Filesystem( [
    'path/to/your/templates',
    $favicons->getTemplatesPath()
]);

$twig = new Twig_Environment($loader, array(
    // other Twig settings
));
```


##Your website template

Inside your website template that renders your HTML head, include the Favicons Markup template. Twig will it look up in the template path defined in “Setup Twig Template Loader” section above:

```twig
{% include 'favicons.website.tpl' with { 
   'app_url':     '/app', 
   'base_url':    '/path/to/static', 
   'theme_color': '#ffffff',
   'icon_color':  '#e21e79'
} only %}
```

This will output a bunch of link and meta elements similar to this:

```html
<link rel="apple-touch-icon" sizes="180x180" href="/path/to/static/apple-touch-icon.png">
<link rel="icon" type="image/png" href="/path/to/static/favicon-32x32.png" sizes="32x32">
<link rel="icon" type="image/png" href="/path/to/static/favicon-16x16.png" sizes="16x16">
<link rel="manifest" href="/app/manifest.json">
<link rel="mask-icon" href="/path/to/static/safari-pinned-tab.svg" color="#e21e79">
<link rel="shortcut icon" href="/path/to/static/favicon.ico">
<meta name="msapplication-TileColor" content="#ffffff">
<meta name="msapplication-TileImage" content="/path/to/static/mstile-144x144.png">
<meta name="msapplication-config" content="/app/browserconfig.xml">
<meta name="theme-color" content="#ffffff">
```



**Please note** the two referenced files `manifest.json` and `browserconfig.xml`. Use an appropriate router like Slim framework to render these files according to your URL and theme color values, like shown in the “Routing” section below:


##Routing browserconfig and manifest files

###Simple Slim 2 example:

```php
<?php
use Germania\Favicons\FaviconRouter;

// Have your Slim app ready
$app = new Slim;

// Setup favicon data
$favicon_data = [
	'app_name'    => 'My App',
	'base_url'    => '/path/to/icons',
	'theme_color' => '#ffffff',
	'icon_color'  => '#e21e79'
	'display'     => 'minimal-ui'
];

// Setup routes 
new FaviconRouter( $app, $twig, $favicon_data);

```

**Advanced example:** Instead of using the FaviconRouter instance above, you may want to define the routes yourself.
This may be useful when you want to use your own templates:

```php
<?php
// Have your Slim app and favicon data ready
$favicon_data = [ ... ];

$app->get('/browserconfig.xml', function() use ( $app, $twig, $favicon_data ) {
    $app->response->headers->set('Content-Type', 'application/xml');
    $output = $twig->render('favicons.browserconfig.xml.tpl', $favicon_data);
    $app->response->setBody( $output );
});

$app->get('/manifest.json', function() use ( $app, $twig, $favicon_data ) {
    $app->response->headers->set('Content-Type', 'application/manifest+json');
    $output = $twig->render('favicons.manifest.json.tpl', $favicon_data);
    $app->response->setBody( $output );
});
```

# Force Safari to update the Pinned Tab icons

It seems that Safari will ignore any new color after a pinned tab icon has been stored locally.

```html
<link rel="mask-icon" href="/path/to/static/safari-pinned-tab.svg" color="#e21e79">
```

Force safari by emptying the Icon Folder in `"~/Library/Safari/Template Icons"`. See Jonathan Hollin's article [Managing Safari 9’s Pinned-tab Icon Cache](https://www.perpetual-beta.org/weblog/managing-safari-9s-pinned-tab-cache.html). It basically goes like this:

```bash
rm ~/Library/Safari/Template\ Icons/*
```


