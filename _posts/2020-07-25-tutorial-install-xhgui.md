---
title: 'Tutorial: install XHGui'
category: php
categories:
  - web-development
  - php
tags:
  - Web Development
---
Recently I had to install [XHGui](https://github.com/perftools/xhgui) in my local environment to profile my PHP application. My goal
was to have an easy way to compare two different executions of my request, one with my
optimizations , and one without them. In this post I will show how my setup works.

<!-- more -->

This post will skip some fundamental concepts that you can easily find elsewhere, like what a
profiler is, and what the history of [XHProf](https://github.com/tideways/php-xhprof-extension) is.

## Install XHProf
[XHProf](https://github.com/tideways/php-xhprof-extension) is a PHP extension. It is the software
that will be instrumenting our code with various metrics like CPU utilization, memory
allocation, execution time, etc. We will use these data later on to decide if our optimizations
are working as expected. The data that XHProf generates is stored in a text file with a data
format that designed to be consumed by machines. This means that once we have XHProf installed
we will have the data available, but we will not know how to read it.

To installing XHProf in you local machine can look a bit different depending on your host machine
. In this tutorial I will show the steps I followed to installed in my Linux desktop.

In this case I will compile the PHP extension from the git repository as instructed in their
README. Bear in mind that you might need to install additional tools to be able to compile the C
code of this extension. Make sure to install these tools using `sudo apt-get install build
-essential`, or searching for _install make gcc_.

### Build and install the extension
Execute the following commands, one by one in your terminal.

```
cd /tmp
git clone https://github.com/tideways/php-xhprof-extension.git
cd php-xhprof-extension/
phpize
./configure
make
sudo make install
```

Make PHP aware of the newly installed extension. For that we will create the _tideways_xhprof.ini_
file in the PHP7.3 directory (change to whatever version you use nowadays). This will contain the
text `extension=tideways_xhprof.so` with no extra configuration. Once we are done we will link the
file to the enabled directory so PHP will load it once it restarts.

This how this looks like in command line instructions:
```
sudo echo "extension=tideways_xhprof.so" > /etc/php/7.3/mods-available/tideways_xhprof.ini
sudo ln -s /etc/php/7.3/mods-available/tideways_xhprof.ini /etc/php/7.3/fpm/conf.d/10-tideways_xhprof.ini
systemctl restart php7.3-fpm.service
```

This works in my local environment, but you might need to adapt those commands to your particular
setup. Execute `php --ini|grep xhprof` and check that the `ini` file created above is detected.
That will indicate that PHP is aware of the extension.

### Install XHGui
To install XHGui I followed the instructions in the [README.md](https://github.com/perftools/xhgui#installation-from-source).
This is what my _nginx_ configuration file looks like (`/etc/nginx/sites-available/xhgui.mateuaguilo.com.conf`)
after following the install instructions there.

```
server {
    listen   80;
    server_name xhgui.mateuaguilo.com;

    # root directive should be global
    root   /home/e0ipso/Sites/xhgui.mateuaguilo.com/webroot/;
    index  index.php;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        include /etc/nginx/fastcgi_params;
        fastcgi_pass unix:/var/run/php/php7.3-fpm.sock;
	fastcgi_index   index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

### Configure XHGui
My goal is to be able to append the [query string](https://en.wikipedia.org/wiki/Query_string)
`_start_xhprof` and `_stop_xhprof` so I can start collecting profile information in any PHP project
in my local.

I achieved that goal by modifying the `profiler.enable` configuration option for XHGui. First copy
the `config/config.default.php` to `config/config.php`, then find the `profiler.enable` array key
and paste the following function as the value:

```
function () {
  $cookie = $_COOKIE['XDEBUG_PROFILE'] ?? NULL;
  if (isset($_REQUEST['_stop_xhprof'])) {
    unset($_COOKIE['XDEBUG_PROFILE']);
    setcookie('XDEBUG_PROFILE', null, -1, '/');
    $_SERVER['REQUEST_URI'] = str_replace(['?_stop_xhprof', '&_stop_xhprof'], '', $_SERVER['REQUEST_URI']);
    return FALSE;
  }
  if (!isset($_REQUEST['_start_xhprof']) && $cookie !== 'xhprof_active') {
    return FALSE;
  }
  // Remove trace of the special variable from REQUEST_URI
  $_SERVER['REQUEST_URI'] = str_replace(['?_start_xhprof', '&_start_xhprof'], '', $_SERVER['REQUEST_URI']);
  if ($cookie !== 'xhprof_active') {
    setcookie('XDEBUG_PROFILE', 'xhprof_active', time() + 3600);
  }
  return TRUE;
}
```

Save the configuration file and restart NGINX with `systemctl restart nginx`.

### Try it out!
Now you can go to any of your local PHP applications (in my case https://local.contrib.com) and
navigate to the page you want to profile. Then append `?_start_xhprof` to the URL and visit the
page. That will cause XHGui to notify XHProf to record the profile information.

Now head out to your local installation of the XHGui site. Mine is at https://xhgui.mateuaguilo.com.
You should see that you have profile information on the page you just executed.

Note that the code for `profiler.enabled` will set a cookie once you request profiling the first
time. After that you will not need to use `?_start_xhprof` for the next 1 hour, all the executions
will be profiled. If you want to stop collecting profile information before the timeout, you can
use the `?_stop_xhprof` query string. That will remove said cookie and stop gathering information.