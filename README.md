# Customisation for ACDH-CH Researchspace based projects

customisation of Researchspace for the purposes of Omnipot [#13886](https://redmine.acdh.oeaw.ac.at/issues/13886 "#13886") and other ResearchSpace based projects.

### How to use this Researchspace app

1. Go to Rancher in the project Researchspace and enter container named docker-git-alpine located in the namespace of your *$PROJECTNAME* by clicking on Execute Shell that can be found right from the workload name under three vertical dots. 
2. `cd /apps/`
3. `git clone --branch $PROJECTNAME --single-branch https://github.com/acdh-oeaw/rs-customisation.git $PROJECTNAME`
4. `cd /apps/$PROJECTNAME`
5. `vi /app/$PROJECTNAME/.git/config`and replace https://github.com/acdh-oeaw/rs-customisation.git by git@github.com:acdh-oeaw/rs-customisation.git
6. Go to Rancher in the project Researchspace, in the left menu click on Storage and under it in dropdown menu on Secrets
7. Add ssh key that has enough rights over the rs-customisation repo
8. Go to Rancher in the project Researchspace, in the namespace *$PROJECTNAME*, and edit researchspace workload. Mount your keys as /var/lib/jetty/.ssh/ssh-privatekey and /var/lib/jetty/.ssh/ssh-publickey. In the Rancher security context for the workload set group ID to 101 and the user ID to 100 and save.  
9. `mkdir -p config/services images assets lib`
10. `chown -R 100:101 /apps`
11. `vi /app/$PROJECTNAME/config/repositories` and add the username and the password for the triplestore database
12. `vi /apps/$PROJECTNAME/plugin.properties`  

Here is an example:
        plugin.id=$PROJECTNAME
        plugin.provider=ResearchSpace
        plugin.version=1.0.0
        #plugin.dependencies=other-app

13. Go to Rancher researchspace workload for *$PROJECTNAME* and edit it by clicking on the three vertical dots on the right side and the label Edit Config.
14. Click on the researchspace tab *(in the menu located on the left side, General should be selected)*, scroll down to Environment Variables section end edit **PLATFORM_OPTS** variable:

```shell
-DruntimeDirectory=/apps/$PROJECTNAME
-Dconfig.storage.runtime.type=nonVersionedFile 
-Dconfig.storage.runtime.mutable=true
-Dconfig.storage.runtime.root=/apps/$PROJECTNAME
-Dconfig.storage.images.type=nonVersionedFile
-Dconfig.storage.images.mutable=true
-Dconfig.storage.images.root=/images
-Dconfig.storage.tmp.type=nonVersionedFile
-Dconfig.storage.tmp.mutable=true
-Dconfig.storage.tmp.root=/tmp-data
-Dconfig.environment.shiroConfig=/apps/$PROJECTNAME/config/shiro.ini
-Dorg.eclipse.jetty.server.Request.maxFormContentSize=1000000 
```
For using GIT as runtime storage, the following PLATFORM_OPTS Rancher ENV variable can be used: 

```
-Dconfig.storage.runtime.type=git  
-Dconfig.storage.runtime.mutable=true  
-Dconfig.storage.runtime.localPath=/apps/$PROJECTNAME
-Dconfig.storage.runtime.branch=$PROJECTNAME 
-Dconfig.storage.runtime.remoteUrl=git@github.com:acdh-oeaw/rs-customisation.git 
-Dconfig.storage.images.type=nonVersionedFile  
-Dconfig.storage.images.mutable=true  
-Dconfig.storage.images.root=/images  
-Dconfig.environment.shiroConfig=/apps/$PROJECTNAME/config/shiro.ini  
-Dorg.eclipse.jetty.server.Request.maxFormContentSize=1000000 
```
Due to specific ssh related configuration required by GIT runtime storage implementation, the following custom Researchspace Docker image should be used: 
`acdhch/researchspace-platform-ci`

To use Java ShenandoahGC add following configuration to the **JAVA_OPTS** variable:

```shell
-XX:+UnlockExperimentalVMOptions
-XX:+UseContainerSupport 
-XX:+UseShenandoahGC 
-XX:ShenandoahGCHeuristics=compact 
-XX:+UseStringDeduplication 
-XX:+ExitOnOutOfMemoryError
```

**NOTE:**

On the first run you need to import set of default ontologies and knowledge patterns for CIDOC-CRM, see [instance](https://github.com/researchspace/researchspace-instance-configurations "instance") configuration repository. 

##### References:
- https://documentation.researchspace.org/resource/Help:Apps
- https://documentation.researchspace.org/resource/Help:Storage
- https://documentation.researchspace.org/resource/Help:GitStorageGuide
- https://github.com/researchspace/researchspace-instance-configurations
