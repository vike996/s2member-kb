---
title: Automatically Redirecting to a Protected Page After a User Logs In to Your Site
categories: tutorials
tags: mu-plugins-hacks,login-registration
author: raamdev
github-issue: https://github.com/websharks/s2member-kb/issues/7
toc-enable: false
---

If you link to a page protected by s2Member and an existing member clicks on that protected link but arrives at your site while logged out, they will get redirected to the Membership Options Page as if they were not a member. 

If you want them to be able to login and then immediately get redirected to that protected page they were trying to access (as opposed to getting redirected to the Login Welcome Page), you'll need to add a special "Login" link to your Membership Options Page that passes the link to the protected page to the Login page, so that after logging in WordPress can properly redirect the user to the protected page.

**Quick Tip:** The [`wp_login_url()` function](http://codex.wordpress.org/Function_Reference/wp_login_url) lets you pass in a redirect URL (perfect!). We'll use this to create the special Login link that we need.

s2Member's MOP Vars (Membership Options Page Variables) contain the "Seeking URI", i.e., the URI to the protected page that the user was trying to access. We'll extract this URI from the MOP Vars and then pass it into the `wp_login_url()` function.

_Note: You'll need to make sure you have the [ezPHP](http://wordpress.org/plugins/ezphp/) plugin installed and activated, so that you can run PHP code directly inside your WordPress pages._

### Add the following code to the top of your Membership Options Page:

```php
<?php
if(!empty($_REQUEST["_s2member_vars"]))
    @list($restriction_type, $requirement_type, $requirement_type_value, $seeking_type, $seeking_type_value, $seeking_uri)
            = explode("..", stripslashes((string)$_REQUEST["_s2member_vars"]));

if (!empty($seeking_uri)) {
    $URI = base64_decode($seeking_uri);
}
?>
```

### Now, we can check if we have a URI available, and if so, output a special Login link:

```html
	Existing members can <a href="<?php echo ( !empty($URI) ? wp_login_url( esc_url($URI) ) : wp_login_url() ); ?>" title="Login">Login</a> to access this page.
```

That's it! Now when someone attempts to access a protected page, they can click on the Login link, enter their login details, and be automatically redirected to the protected link they were trying to access!

---

### Taking It A Step Further

**Developer Tip:** If you want to automatically add a login/logout link to a WordPress Navigation Menu; i.e., one that contains the redirect URI when attempting to access a protected page, you can add the following code your theme's `functions.php` file. The exact code to use here will depend on your WordPress theme. If you're not familiar with WordPress theme development and/or PHP, please consult with your theme developer to find out exactly how the following code should be implemented.

**Note: This only works from the Membership Options Page.** This will only redirect a user who clicks a login link _from the Membership Options Page_, after they have been redirected to the Membership Options Page because they tried to access some protected content. This code _does not_ simply redirect a visitor to any previous page they were on.

```php
// Filter wp_nav_menu() to add a login/logout link to the nav menu
add_filter( 'wp_nav_menu_items', 'my_nav_menu_login_link', 10, 2 );
function my_nav_menu_login_link($menu, $args) {

	if( $args->theme_location == 'primary' ) { // The name of your nav menu location, see http://codex.wordpress.org/Navigation_Menus#Display_Menus_on_Theme

		// If the user is logged in, show logout link, otherwise show login link
		if (is_user_logged_in())
		{
			$logout_link = '<li><a href="' . wp_logout_url() .'" title="Logout">Logout</a></li>';
			$menu        = $menu.$logout_link;
			return $menu;
		}
		else
		{
			// If a the visitor was attempting to access a protected page, extract the URI
			if(!empty($_REQUEST["_s2member_vars"]))
				@list($restriction_type, $requirement_type, $requirement_type_value, $seeking_type, $seeking_type_value, $seeking_uri)
					= explode("..", stripslashes((string)$_REQUEST["_s2member_vars"]));

			if (!empty($seeking_uri)) {
				$URI = base64_decode($seeking_uri);
			}

			// If we have a link to redirect to after login, use it, otherwise use the default login URL
			if(!empty($URI)) {
				$login_link = '<li><a href="' . wp_login_url( esc_url($URI) ) .'" title="Login">Login</a></li>';
				$menu        = $menu.$login_link;
			}
			else {
				$login_link = '<li><a href="' . wp_login_url() .'" title="Login">Login</a></li>';
				$menu        = $menu.$login_link;
			}
		}
	}

	return $menu;
}
```
