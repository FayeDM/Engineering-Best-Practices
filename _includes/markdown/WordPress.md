Documents about working with WordPress specifically will live on this page.

## Miscellaneous WordPress Notes
* Local username/password on all projects should be `kadmin`/`kadmin`. Make it so when you see it otherwise.
* Use `add_filter( 'admin_email_check_interval', '__return_false' );` to disable the admin email notification nag.

## Helpful Links

* [WordPress Playground](https://developer.wordpress.org/playground/) -- quick testing
* [WordPress Developer Blog](https://developer.wordpress.org/news/)
* [10up Gutenberg knowledge base](https://gutenberg.10up.com/)

## Training Sites
Mostly for clients and content entry -- but may be useful in other contexts!

* [https://learn.wordpress.org/tutorials/](https://learn.wordpress.org/tutorials/)
* [https://yoast.com/academy/free-training-wordpress-for-beginners/](https://yoast.com/academy/free-training-wordpress-for-beginners/)
* [https://javascriptforwp.com/product-category/courses/](https://javascriptforwp.com/product-category/courses/)
* [https://www.amazon.com/WordPress-Dummies-Computer-Tech-ebook/dp/B08PNYVYHH](https://www.amazon.com/WordPress-Dummies-Computer-Tech-ebook/dp/B08PNYVYHH)
* [https://easywpguide.com/wordpress-manual/](https://easywpguide.com/wordpress-manual/)

## WordPress Community at Kanopi

* **WordPress Launch Checklist**: Kanopi has an interactive [launch checklist](https://github.com/kanopi/wp-launch-checklist) plugin for WP projects.
* **WordPress Channel**: Kanopi has a [general WordPress channel](https://kanopi.slack.com/archives/C0G0G9345) for all your WordPress questions, laments, and more!
* **Plugin Updates Channel**: Kanopi has a [Slack channel for WordPress plugin updates](https://kanopi.slack.com/archives/C04MX7E36VB) posted via RSS feed. The plugins in this channel are commonly used across numerous WordPress sites.
* **WordPress Cowork**: Thursdays at 4pm eastern, numerous WordPressers gather on Zoom to natter about WordPress things.

## Patching WordPress Plugins

Step one: 🛑 don't 🛑. Manually patching plugins -- especially extensively -- introduces severe maintenance concerns.

First, try to submit a support request to the plugin developer so they patch the plugin for you. In some instances, however, we may need to create a long-standing patch for a plugin. We should only do so in cases where our needs are extremely specific. 

Then, try alternates. Is there another plugin that will work better for your purpose? Can you write custom code or similar to get what you need? Is there literally any alternative to manully hand-patching the plugin?

Then, try alternate ways of patching directly. If it's a customization you need in a JS file, dequeue the original and enqueue the modified version. Try to override a PHP class method or function. Check out the plugin filters and whether those can be easily overridden.

Only as an _absolute last resort_ should you attempt patching a WordPress plugin directly.

### Workflow
* Consider creating a repository for your patched plugin version. This will allow you to track changes separately and maintain a clean history of the plugin changes.
* Add "Update URI: False" to the plugin main file. This prevents your plugin from being updated through the admin, thus potentially wiping your changes.
* Document why you made the patch.
* Add steps to test and verify the plugin change to your project's README, QA instruction, and everywhere else possible. It's never too much communication -- anywhere that the plugin patch can be noted, please note it. It should be made very, very clear why and where exactly you modified things.

### Create a Git Patch
* Generate a [git patch](https://git-scm.com/docs/diff-generate-patch) of your plugin changes
* Put the git patch into a directory /patches in your plugin folder

### Add Self-Updating Script 
The self-updating script is optional. It will get the latest version of your plugin from the repository, and then attempt to re-apply your changes on top of the latest.

* Open Cacher to get the self-updating bash script - guid:34d76d2077e2357bed88
* Put update.sh into the root or the plugin folder (root is suggested as it is more visible)
* Replace all instances of wp-security-audit-log with your plugin name (or update the cacher script so it is more generic)
* Run bash update.sh to automatically get the latest version of the plugin and re-apply your patched changes
* Hope against all hope that there are no merge conflicts. You will still have to resolve those manually 🙃

### Examples of Patches
These patch examples are a good way to evaluate your situation and whether you should be patching or not.

* [pen.org](https://github.com/kanopi/pen_org#plugins) -- ACF needed patched as it was causing 502 errors on Pantheon on an extremely large site. There was no workaround for patching the plugin directly as the filters were not exposed. There was also no alternate path to getting this to work either as the necessary filters were not exposed. [Example of the pen.org patch](https://github.com/kanopi/pen_org/blob/main/patches/001-acf-pantheon.patch)
* Indeed - WP Security Audit Log (no link, you'd need access to their Gitlab and that is a whole process). We created a semi-extension of the base plugin. The extensive nature of the extension required that any update to the base plugin be handled very carefully. So we created a separate repository to house the base plugin, with "Update URI: false" enabled so that no one ever accidentally updated the base plugin without testing. We also included the bash script as an easy way to remember to update things.