---
title: 'Toggle XDebug on and off'
category: php
categories:
  - web-development
  - php
tags:
  - Web Development
image: /assets/images/2020/switch.jpg
---
Christian (@penyaskito) is a dear friend of mine. I talk to him every day (usually text), but I
have never blogged for him. This post should fix that.

<!-- more -->

He asked me today if I usually have XDebug turned on or off by default. I have it off, but I depend
so much on it that I created a homegrown script to toggle it on and off quickly.

There are three tasks that I do with it:
<pre>
xdebug-toggle status
xdebug-toggle off
xdebug-toggle on
</pre>

This is the contents of **my** script, I use PHP-FPM + nginx + Linux. <em>Consider this script under
 GPL-3.</em>

<pre><code>
#!/usr/bin/env bash

app="$(basename "$0")"
command="$1"
options="$2"

FILENAME_ON="/etc/php/7.3/mods-available/xdebug.ini"
FILENAME_OFF="/etc/php/7.3/mods-available/xdebug.ini.off"

php_ini=$(php -i|grep "Loaded Configuration File"|cut -d">" -f2|sed -e "s:^ *::g")
STATUS=$(stat ${FILENAME_ON} > /dev/null 2>&1 && echo enabled || echo disabled)

if [[ $command == "status" ]]
then
    echo $STATUS
    exit 0
fi

if [[ $command == "on" ]] && [[ $STATUS == "enabled" ]]
then
    echo "[NOTICE] Xdebug was already enabled."
    exit 0
fi
if [[ $command == "off" ]] && [[ $STATUS == "disabled" ]]
then
    echo "[NOTICE] Xdebug was already disabled."
    exit 0
fi

if [[ $command != "on" ]] && [[ $command != "off" ]]
then
  echo "Unrecognized command"
  exit 1
fi

if (($EUID != 0))
then
  if [[ -t 1 ]]; then
    sudo "$0" "$@"
  else
    exec 1>output_file
    gksu "$0 $@"
  fi
  exit
fi

if [[ $command == "on" ]]
then
    mv ${FILENAME_OFF} ${FILENAME_ON}
    systemctl restart php7.3-fpm
    systemctl restart nginx
    echo -n "Xdebug is now: "
    $0 status
    exit 0
fi

if [[ $command == "off" ]]
then
    mv ${FILENAME_ON} ${FILENAME_OFF}
    systemctl restart php7.3-fpm
    systemctl restart nginx
    echo -n "[INFO] Xdebug is now: "
    $0 status
    exit 0
fi
</code></pre>

<small>Photo by <a href="https://unsplash.com/@yogesh_7?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Yogesh Pedamkar</a> on <a href="https://unsplash.com/s/photos/switch?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></small>