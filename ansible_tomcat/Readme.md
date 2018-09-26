#Tomcat-Playbook

This playbook installs Tomcat  at /opt location and creats softlink to the installation at /usr/local<br>


###Playbook variables

We have defined below mentioned variables for our playbook.


```
---
http_port: 8086
https_port: 8443
```

####Current Required functionalities.
- Tomcat clustring.Allows to create horizontal and vertical clustring.
- Allows HA and session handling capabilities. 

#### To contribute

- Create a feature branch from the change.
- Make changes to feature branch only.
- Test your feature branch.
- Merge back to master.