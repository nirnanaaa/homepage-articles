---
title: GollumRedmine
date: 2014-01-30 14:21:00 GMT+02
---

### What is GollumRedmine?

gollum_redmine is a plugin for redmine. It includes the following features:  

* Store your meeting protocols in git
* Display your wiki in redmine
* Create project descriptions for your projects
* Commit as your redmine user


### How can I get this?

Simply download it from my GitHub page.

```sh
cd /path/to/your/redmine/folder
cd plugins
git clone https://github.com/nirnanaaa/gollum_redmine.git gollum
```

### How does it work?

It uses your existing wiki infrastructure and boosts it to another level. All the above features  
are stored in this repository. You can configure them under the `Administration > Plugins` tab. 

Once these variables are configured you need to restart your Rails application.

Navigate to "Wiki" in the top menu. Now you should see your defined landing page.


