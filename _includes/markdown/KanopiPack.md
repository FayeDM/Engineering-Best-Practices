## What is Kanopi Pack and Why are we using it?

Kanopi Pack is tool set designed to consolidate and ease the management of CSS and JS assets used in a web application along with the supporting packages which build them. The project is available for installation via NPM and documentation of the Why, When, and How for the tool are published in the Github repository [Readme](https://github.com/kanopi/kanopi-pack#why-kanopi-pack).


## Migrating a WordPress site to Kanopi Pack

There are many steps involved in converting a legacy site or creating a new site, using [Kanopi Pack](https://github.com/kanopi/kanopi-pack/). Below, they are detailed in the steps recommended. This is meant to be a high-level overview with explanations. Considerations may be needed on a project-to-project basis.

1. Composer
2. Docksal
3. WP Config
4. MU Plugins
5. Theme

CircleCI and Tugboat changes are not referenced here, as that should be handled by the TechOps team. However, any compiled changes should NOT be stored in the repo, and instead, handled via CI.

## Composer

* In your project root, you will need to have a `composer.json` file. This is separate from your WordPress theme files.
* You should ensure that the name is unique to your project. It's highly suggested that you match this to the GitHub repo name (in this case, `ftd-fpi`).
* The example `composer.json` file below is for a Pantheon-based, fully composer build.
* Remove the `johnpbloch/wordpress-core` package if you do not manage core via Composer. However, the rest is required for use of the Kanopi Pack Asset Loader, and the updated PHPCS standards.
* If you do manage core via Composer, to add contributed plugins, run `composer require wpackagist-plugin/[plugin name]`. You can search for these on WPackagist, or simply by taking the slug from the official Plugin repo, and using that for the plugin name.
* This sample uses PHPCS, you can read more about WordPress configuration of PHPCS in this [Cacher](https://snippets.cacher.io/snippet/6d1ec6cbc54c58922dda).
* **IMPORTANT**: This is a blueprint which is not up to date. The listed versions of packages are being updated regularly, please check and update the version of PHP, WordPress Core, and [Kanopi Pack based Asset Loader](https://github.com/kanopi/kanopi-pack-asset-loader) on the latest supported version available through NPM and Composer.
* **NOTE**: If your project is a Pantheon-hosted site, the root of your WordPress site and hence its files are in the `web/` subfolder. Sites on WPEngine and many legacy support projects store WordPress in the root directory. Check configuration file paths to ensure they use the correct root path.

```
{
    "name": "kanopi/name-of-project",
    "description": "",
    "type": "project",
    "keywords": [],
    "repositories": {
        "wpackagist": {
            "type": "composer",
            "url": "https://wpackagist.org"
        }
    },
    "require": {
        "php": ">=8.2",
        "composer/installers": "2.x-dev",
        "johnpbloch/wordpress-core": "6.3",
        "kanopi/pack-asset-loader": "~1.0.2"
    },
    "require-dev": {
        "automattic/vipwpcs": "~3.0.0",
        "phpunit/phpunit": "~9.5.3",
        "roave/security-advisories": "dev-master",
    },
    "config": {
        "vendor-dir": "web/wp-content/mu-plugins/vendor",
        "preferred-install": "dist",
        "optimize-autoloader": true,
        "sort-packages": true,
        "platform": {
            "php": "8.2"
        },
        "allow-plugins": {
            "composer/installers": true,
            "cweagans/composer-patches": true,
            "dealerdirect/phpcodesniffer-composer-installer": true,
            "oomphinc/composer-installers-extender": true
        }
	},
    "scripts": {
        "move-core-files": "rsync -az --delete --exclude 'wp-content' web/wp-content/mu-plugins/vendor/johnpbloch/wordpress-core/* web/; echo 'Moved core files to web directory' ",
        "post-install-cmd": [
            "@move-core-files"
        ],
        "post-update-cmd": [
            "@move-core-files"
        ],
        "phpcs": [
            "Composer\\Config::disableProcessTimeout",
            "web/wp-content/mu-plugins/vendor/bin/phpcs --standard=\"./.phpcs.xml.dist\" web/wp-content/"
        ],
        "phpcbf": [
            "Composer\\Config::disableProcessTimeout",
					"web/wp-content/mu-plugins/vendor/bin/phpcbf --standard=\"./.phpcs.xml.dist\" web/wp-content/"
				]
    },
    "extra": {
        "installer-paths": {
            "web/wp-content/plugins/{$name}/": [
                "type:wordpress-plugin"
            ],
            "web/wp-content/themes/{$name}/": [
                "type:wordpress-theme"
            ],
            "web/wp-content/themes/custom/{$name}/": [
                "type:wordpress-theme-custom"
            ]
        }
    }
}
```

## Docksal

### .env

Below is a Pantheon-specific example for your `docksal.env` file. For WPEngine sites, change the stack to `default`, and the DOCROOT to `.`

```
DOCKSAL_STACK=pantheon
CLI_IMAGE='docksal/cli:php8.2-3.5'

# Docksal configuration.
DOCROOT=web

# Enable/disable xdebug
# To override locally, add .docksal/docksal-local.env and set XDEBUG_ENABLED=1
XDEBUG_ENABLED=0

# WordPress settings
WP_ADMIN_USER=example
WP_ADMIN_PASS=example
WP_ADMIN_EMAIL=example_email_address

# Change the following to match the project theme name
WP_THEME_SLUG=custom/theme-name
WP_THEME_RELATIVE_PATH=wp-content/themes/$WP_THEME_SLUG

# Subdomain used to proxy the Kanopi Pack dev server assets 
WP_THEME_ASSETS_HOST_SUBDOMAIN=theme-assets

# Local ssl certs folder
CONFIG_CERTS="${CONFIG_CERTS:-$HOME/.docksal/certs}"

# Set hosting details for Pull command (Pantheon specific)
# Put any specific variables for your refresh command here.

```

### .yml

```
version: "2.1"

services:
  web:
    extends:
      file: ${HOME}/.docksal/stacks/services.yml
      service: nginx
    environment:
      - APACHE_FILE_PROXY
      - WP_THEME_ASSETS_HOST_SUBDOMAIN
      - WP_THEME_RELATIVE_PATH
      - WP_THEME_SLUG
      - NGINX_VHOST_PRESET=wordpress
  cli:
    environment:
      - WP_ADMIN_USER
      - WP_ADMIN_PASS
      - WP_ADMIN_EMAIL
      - WP_THEME_RELATIVE_PATH
      - WP_THEME_ASSETS_HOST_SUBDOMAIN
      - WP_THEME_SLUG
      - COMPOSER_MEMORY_LIMIT=-1
      - VIRTUAL_HOST
      - CONFIG_CERTS
      - DB_HOST=db
      - DB_NAME=default
      - DB_USER=user
      - DB_PASSWORD=user
    labels:
      - io.docksal.virtual-host=${WP_THEME_ASSETS_HOST_SUBDOMAIN}.${VIRTUAL_HOST}
      - io.docksal.virtual-port=4400
    expose:
      - 4400
  pma:
    hostname: pma
    image: phpmyadmin/phpmyadmin
    environment:
      - PMA_HOST=db
      - PMA_USER=root
      - PMA_PASSWORD=${MYSQL_ROOT_PASSWORD:-root}
    labels:
      - io.docksal.virtual-host=pma.${VIRTUAL_HOST}
```

### Commands

You will need to create or update the following commands:
* fin init
* fin init-site
* fin development
* fin production
* fin init-theme-assets
* fin npm

You can copy most of these from [this Cacher](https://snippets.cacher.io/snippet/c0ba9321b584bfa1c756).
You can find the two missing commands below:

`fin init`

```
#!/usr/bin/env bash

## Initialize a Docksal powered Wordpress site
##
## Usage: fin init

# Abort if anything fails
set -e

#-------------------------- Settings --------------------------------

# PROJECT_ROOT is passed from fin.
# The following variables are configured in the '.env' file: DOCROOT, VIRTUAL_HOST.

DOCROOT_PATH="${PROJECT_ROOT}/${DOCROOT}"

#-------------------------- END: Settings --------------------------------

#-------------------------- Helper functions --------------------------------

# Console colors
red='\033[0;31m'
green='\033[0;32m'
green_bg='\033[42m'
yellow='\033[1;33m'
NC='\033[0m'

echo-red () { echo -e "${red}$1${NC}"; }
echo-green () { echo -e "${green}$1${NC}"; }
echo-green-bg () { echo -e "${green_bg}$1${NC}"; }
echo-yellow () { echo -e "${yellow}$1${NC}"; }

#-------------------------- END: Helper functions --------------------------------

#-------------------------- Execution --------------------------------

if [[ "$PROJECT_ROOT" == "" ]]; then
	echo-red "\$PROJECT_ROOT is not set"
	exit 1
fi

if [[ $DOCKER_RUNNING == "true" ]]; then
	echo -e "${green_bg} Step 1 ${NC}${green} Recreating services...${NC}"
	fin reset -f
else
	echo -e "${green_bg} Step 1 ${NC}${green} Creating services...${NC}"
	fin up
fi

echo "Waiting 10s for MySQL to initialize...";
sleep 10

# Project initialization steps

echo -e "${green_bg} Step 2 ${NC}${green} Initializing site...${NC}"
fin init-site

echo -e "${green_bg} Step 3 ${NC}${green} Initializing theme assets...${NC}"
fin init-theme-assets

# Site initialization
echo -e "${green_bg} Step 4 ${NC}${green} Pull database from Pantheon...${NC}"
fin refresh

echo -e "${green_bg} Step 6 ${NC}${green} Disable plugins for local development and restore admin user...${NC}"
fin disable-plugins
fin restore-admin-user
fin wp rewrite flush

echo -en "${green_bg} DONE! ${NC} "
echo -e "Open ${yellow}http://${VIRTUAL_HOST}${NC} in your browser to verify the setup."
echo -e "Admin panel: ${yellow}http://${VIRTUAL_HOST}/wp-admin${NC}. User/password: ${yellow}${WP_ADMIN_USER}/${WP_ADMIN_PASS}${NC}  "
echo -e "Kanopi Pack Dev Server: Run ${yellow}fin development${NC} and you can run ${yellow}fin open kanopi-pack${NC} to run ${yellow}https://${WP_THEME_ASSETS_HOST_SUBDOMAIN}.${VIRTUAL_HOST}/webpack-dev-server${NC} "

#-------------------------- END: Execution --------------------------------
```

`fin init-site`

```
#!/usr/bin/env bash

#: exec_target = cli

## Check and setup site directories and config files
##
## Usage: fin init-site

# Abort if anything fails
set -e

#-------------------------- Settings --------------------------------

# PROJECT_ROOT and DOCROOT are set as env variables in cli
DOCROOT_PATH="${PROJECT_ROOT}/${DOCROOT}"
UPLOADS_PATH="${DOCROOT_PATH}/wp-content/uploads"

#-------------------------- END: Settings --------------------------------

#-------------------------- Helper functions --------------------------------

# Copy default wp-config.file
copy_config () {
  echo "Copy local WordPress configuration..."
	cd $DOCROOT_PATH
	rm -f wp-config-local.php
	cp ${PROJECT_ROOT}/.docksal/etc/config/wp-config-local.php wp-config-local.php
}

# Root Composer Installation
composer_install_root () {
  echo "Installing composer packages from ${PROJECT_ROOT}..."
  cd $PROJECT_ROOT
  composer install --ansi
}

# Fix file/folder permissions
fix_permissions () {
	echo "Making uploads directory, ${UPLOADS_PATH} writable..."
	if [ ! -d "${UPLOADS_PATH}" ]; then
		mkdir -p "${UPLOADS_PATH}"
	fi
	chmod 755 "${UPLOADS_PATH}"
}

#-------------------------- END: Functions --------------------------------

#-------------------------- Execution --------------------------------

# Project initialization steps
composer_install_root
copy_config
fix_permissions
#-------------------------- END: Execution --------------------------------
```


## WP Config

For Kanopi Pack to be able to run its development server, you will need to add the following code to your `wp-config-local.php` (for Pantheon) or `wp-config.php` (for WPEngine and other projects):

```
/**
 * For Front-end Development Server to be served via @kanopi/pack
 */
define('KANOPI_DEVELOPMENT_ASSET_URL', 'https://' . getenv('WP_THEME_ASSETS_HOST_SUBDOMAIN') . '.' . getenv('VIRTUAL_HOST'));
```

For Docksal configurations, keep a copy of this configuration file in `.docksal/etc/config` and copy it into the root of the WordPress site in the `init-site` command.

## MU-Plugins

Within the `mu-plugins` folder, situated under `wp-content`, you will need the following code added. The filename suggestion is `001-kanopi-app-loader.php`. This is required for the Asset Loader to work. Please make sure to exclude it from the `.gitignore`.

```
<?php
/**
 * Plugin Name: Kanopi MU Plugin Application Loader
 * Author: Kanopi, Inc.
 * Author URI: https://kanopi.com
 * Description: Kanopi Autoloader initializes Composer-based Autoloader along with required plugins
 *
 * @wordpress-plugin
 * Version: 1.0.0
 */

// Run the autoloader for the site if not loaded externally
if ( ! class_exists( 'Kanopi\Assets\Registry\WordPress' ) ) {
	if ( ! file_exists( WP_CONTENT_DIR . '/mu-plugins/vendor/autoload.php' ) ) {
		wp_die( 'Unable to run theme autoloader.' );
	}

	include_once WP_CONTENT_DIR . '/mu-plugins/vendor/autoload.php';
}
```

## Theme

### NPM Packages

* In the theme, add a `.nvmrc` file, with the content being the latest supported version (currently 20 on 23/09/19). This will force NPM/NVM to use Node 20, as older versions have, or are hitting EOL (end of life). You will also need to update that value in CircleCI's `config.yml`. Visit the [cimg/node documentation](https://circleci.com/developer/images/image/cimg/node) page to see what version is available.
* You will need to modify (or create) a `package.json` file. If you have an existing one, remove your `package-lock.json`, and any `node_modules` you may have (if you have spun up already). 
	* To install new packages, run `fin npm i [packagename]`. You may install multiple packages at once, by separating out the package name with a space. If this is a dev dependency, add the `--save-dev` flag.
	* To remove packages, either remove the line in `package.json`, or run `fin npm uninstall [packagename]`
* You will need to make sure `@kanopi/pack` is installed, and if you are using React, `@kanopi/pack-react`. 
	* The scripts section will differ from the regular Kanopi Pack (KP), and the React version. Both will be development/production, but change slightly. The example package.json below uses regular KP, but the React version is `"production": "kanopi-pack-react react",` and `"development": "kanopi-pack-react react development"`.

```
{
  "name": "your-theme",
  "version": "1.0.0",
  "repository": {
    "type": "wordpress-theme-custom",
    "url": "https://github.com/kanopi/wp-basebuild"
  },
  "private": true,
  "scripts": {
    "production": "kanopi-pack standard",
    "development": "kanopi-pack standard development"
  },
  "devDependencies": {
    "@kanopi/pack": "^2.2.0",
  },
  "dependencies": {
    "sass-toolkit": "^2.10.2"
  },
  "engines": {
    "node": ">=20",
    "npm": ">=9"
  }
}
```  	

### Assets

You will need to have an `/assets/` folder, with the following two subfolders, `/config/`, and `/src/`. All your JS and SASS/SCSS partials will live in the `/src/` folder, with the folder structure being explained below. Your images and fonts will live in the `/static/` folder under `/src/`.

```
├── src
│   ├── js
│   │   ├── **/*/*.js
│   │   ├── theme.js
│   ├── scss
│   │   ├── **/*/*.scss
│   │   ├── theme.scss
│   ├── static
│   │   ├── images
│   │       ├── */[fileextension]

```

#### Calling assets

To load an image in your SCSS or JavaScript file, use this path:
`@/static/images/name-of-image.format`

For a font:
`@/static/fonts/name-of-font.format`

If you are calling an asset directly in PHP, this will not work.

In that case, use the path:
`assets/src/static/` after the dynamic theme path.

#### Config

You will need to create a `kanopi-pack.js` file, along with a subfolder called `/tools/`, which will contain your StyleLint exclusions. Below is an example of a `kanopi-pack.js` configuration file and [here](https://github.com/kanopi/kanopi-pack/tree/main/examples). You may find the full explanation at the [Kanopi Pack repo](https://github.com/kanopi/kanopi-pack/blob/main/documentation/configuration.md).

The entry points below correspond to your main files. For instance, if your theme styles live in `theme.scss`, you would create an entry point called `theme`, and then reference the path. The same would go for your Javascript. In the example below, the `legacy` entry point contains miscellaneous Javascript from legacy builds, and has been included in that subfolder. The `customizer` endpoint is for any Customizer or admin-related scripts.

At the bare minimum, you must provide an entry point for your theme. The naming is important, as that will be referenced later in `functions.php`. 


```
const {
	VIRTUAL_HOST: sockHostDomain = "",
  WP_THEME_ASSETS_HOST_SUBDOMAIN: sockHostSubdomain = ""
} = process.env;

module.exports = {
	"devServer": {
		"sockHost": sockHostSubdomain + '.' + sockHostDomain,
		"sockPort": 443,
		"useSslProxy": true,
		"useProxy": true,
		"watchOptions": {
			"poll": true
		},
		"allowedHosts": [".docksal.site"]
	},
	"environment": {
		"dotenvEnable": false
	},
	"filePatterns": {
		"cssOutputPath": "css/[name].[contenthash].css",
		"entryPoints": {
			"legacy": "./assets/src/js/legacy.js",
			"theme": "./assets/src/scss/theme.scss",
			"customizer": "./assets/src/js/customizer.js"
		},
		"jsOutputPath": "js/[name].[contenthash].js"
	},
	"styles": {
		"scssIncludes": [
			"./assets/src/scss/shared/utilities.scss" 
		],
		"styleLintConfigFile": "./assets/configuration/tools/.stylelintrc.js", 
		"styleLintIgnorePath": "./assets/configuration/tools/.stylelintignore"
    "useSass": true,
    "devHeadSelectorInsertBefore": "#id-of-enqueued-style"
	},
	"scripts": {
		"esLintAutoFix": false,
		"esLintDisable": true,
	}
};
```

* The devHeadSelectorInsertBefore lets you define where to place the development server CSS while you work on the site locally. It is very useful when you work in sites that use plugins like BuddyBoss. If you do not need it, you can remove that line. Passing a wrong ID will hide your compiled CSS.
* The `scssIncludes` would contain all utilities, such as variables and mixins, that are needed for your stylesheet to compile correctly. 

#### Stylelint

Make sure to create the Stylelint files being referenced in `kanopi-pack.js` file.

`.stylelintrc.js`
Copy this content and modify as needed:

```
module.exports = {
	'customSyntax': 'postcss-scss',
	'extends': 'stylelint-config-property-sort-order-smacss',
	'rules': {
		'order/properties-order': null,
		'color-hex-length': 'long',
		'indentation': 'tab',
		'max-nesting-depth': 10,
		'property-no-vendor-prefix': null,
		'selector-max-compound-selectors': 10,
		'selector-max-id': 6,
		'selector-no-qualifying-type': null
	}
}
```

`.stylelintignore`
Copy this content and modify as needed:

```
assets/**/*.js
assets/**/*.css
```

You may also disable particular rules inline (as the above is global), using `/* stylelint-disable [RULENAME} */`, and then re-enabling with `/* stylelint-enable [RULENAME] */`

### Functions

Now that you've moved all your assets and have a good understanding of what compiled files will be required, it's time to update the way WordPress will enqueue the files.

Start by adding these libraries to the root of your `functions.php`:

```
// Enqueue things.
use Kanopi\Assets\Model\LoaderConfiguration;
use Kanopi\Assets\Registry\WordPress;
```

Right after, add the following code:

```
/**
 * Kanopi Pack Theme Setup
 *
 * @return void
 */
if ( class_exists( 'Kanopi\Assets\Registry\WordPress' ) ) {
	// All assets are held in the active theme under the directory /assets.
	// The constant `KANOPI_DEVELOPMENT_ASSET_URL` is defined before calling, otherwise, only Production mode is available.
	$loader = new WordPress(
		new LoaderConfiguration(
			WordPress::read_theme_version(),
			array( 'PRODUCTION_URL' ),
			'/assets/dist/webpack-assets.json'
		)
	);

	$loader->register_frontend_scripts(
		function ( $_registry ) {
			$loader = $_registry->asset_loader();

			$loader->register_vendor_script( 'vendor' );

      $loader->in_development_mode() ? $loader->register_runtime_script( 'central' ) : '';

			$loader->register_runtime_script( 'runtime', array( 'jquery' ) );

			$loader->register_style( 'theme' );

			$loader->register_script( 'legacy' );

			add_action( 'admin_init', function () use ( $loader ) {

				$loader->register_style( 'editor' );

			});

			// Required theme stylesheet.
			wp_register_style(
				'SITENAME-style-header',
				esc_url_raw( get_stylesheet_directory_uri() . '/style.css' ),
				array(),
				$_registry::read_theme_version(),
			);
			wp_enqueue_style( 'SITENAME-style-header' );

			// Enqueue our assets last.
			$loader->enqueue_assets();
		}
	);
} //end if
```

#### Loader Configuration

1. `PRODUCTION_URL` should be replaced with the production link of the project's website.
1. The `vendor` script might show an error in your production environment if you do not use it. If so, add it to the `$loader->in_development_mode()` condition used to `central`. If you have package dependencies declared in the `package.json`, you must include that file in production. You also need to move (as in, remove from `runtime`) the `jquery` array to the `vendor` script to make sure your dependencies get what they need.
2. The `runtime` script needs to stay and you do not have to set up anything for it.
3. The `theme` style is the name of the `scss` file provided on `kanopi-pack.js` as an entry point. Any file generated by the compiler must be called this way.
5. The legacy `wp_register_style` only needs the `SITENAME` value updated.

#### Admin Scripts

You may need to enqueue styles and scripts for the admin only. Here is an example on how to accomplish this:

```
// Setup admin only scripts and styles.
add_action(
  'admin_enqueue_scripts',
  function () use ( $loader ) {
    $registry = $loader->asset_loader();

    $registry->in_development_mode() ? $registry->register_vendor_script( 'vendor' ) : '';
    $registry->register_runtime_script( 'runtime', [ 'jquery' ] );

    global $pagenow;
    if ( 'users.php' === $pagenow ) {
      $registry->register_script( 'user' );
      $registry->register_style( 'user_style' );
    } else {
      $registry->register_script( 'admin' );
    }

    // Enqueue our assets last.
    $registry->enqueue_assets();
  }
);
```
1. If you have package dependencies declared in the `package.json`, you must include the `vendor` script in production by removing the condition beforehand. You also need to move (as in, remove from `runtime`) the `jquery` array to the `vendor` script to make sure your dependencies get what they need.

You can place this code after the `$loader->register_frontend_scripts` function.

#### Block Scripts

If your site is using blocks, this is how you would register the custom styles and scripts right after the `$loader->register_frontend_scripts` function:

```
$loader->register_block_editor_scripts(
  function ( $_registry ) {
    $loader = $_registry->asset_loader();
    $loader->register_vendor_script( 'vendor' );

    $loader->register_runtime_script( 'runtime', array( 'jquery' ) );
    $loader->register_style( 'editor' );

    $loader->enqueue_assets();
  }
);
```

#### External Scripts or Stylesheets

Ideally, you would load packages through NPM. However, you may need to call a thirdparty script or stylesheet. If so, you must proceed to a regular enqueue:

```
  wp_register_script( 'fontawesome', '//kit.fontawesome.com/6b142e11da.js', array( 'jquery' ) );
  wp_enqueue_script( 'fontawesome' );
```

Place this code before `$loader->enqueue_assets();`.

Finally, make sure to remove any code that still enqueues assets outside of this function.

## You made it!

Once you have completed the installation, you can run `fin development` and confirm that your assets are being compiled.

Head over to your Docksal site and enjoy coding in a much faster way!

## Troubleshooting

Below are some common issues that have been encountered.

**_My styles aren't loading!_**

Make sure that you are running `fin development`. If this is running, and your styles still aren't loading, make sure you have the Kanopi Pack Asset loader installed, and loading (see MU Plugins above). You cannot run `fin open` before `fin development`.

Also, review the https://theme-assets.${VIRTUAL_HOST}/webpack-dev-server to confirm all the files you are trying to enqueue are being generated.

**_CircleCI says the build step was successful, but my site is borked_**

This likely means there was a silent fail on `production`. To test this, run `fin production`, and see if your distribution folder is generated in `assets`. If it is not, run `fin development` to diagnose the issue. Often, especially with React apps, it could be something as simple as case sensitivity.

**_fin development won't run for me :(_**

Sometimes Docker can be a resource hog. Stop all your running projects, and restart (`fin restart`) just the one you are working on, and try `fin development` again. If that doesn't work, restart Docker Desktop.

**_fin production isn't working on CircleCI_**

You may run `fin production` in your local environment in the future to debug a deployment that is failing. Make sure to add a `.gitignore` file in your theme to avoid pushing files from the `dist` folder.

**_My site has a vendor.js file not found error in production_**

The vendor script file will only be generated if you added specific libraries (like a slider) through `npm install` and imported those files in your Javascript. If you do not use any external vendor file, you do not need to enqueue the vendor script in Production. To do so, edit the code loading the asset to have it handle a condition based on the environment:

`$loader->in_development_mode() ? $loader->register_vendor_script( 'vendor' ) : '';`