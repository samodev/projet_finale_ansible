#Ansible Playbook -Artifactory


This playbook installs artifactory  at /opt location .


###Playbook variables

We have defined below mentioned variables for our playbook.


```
artifactory_version: artifactory-oss-4.4.1
artifactory_dir: /opt/{{ artifactory_version }}
artifactory_zip_file: artifactory-oss-4.4.1.zip
artifactory_zip_url: http://dl.bintray.com/content/jfrog/artifactory/jfrog-"{{ artifactory_zip_file }}"

```

####Current Required functionalities.
- Use of database  for  artifactory
- Allow repository creation and configuration.

#### To contribute

- Create a feature branch from the change.
- Make changes to feature branch only.
- Test your feature branch.
- Merge back to master.