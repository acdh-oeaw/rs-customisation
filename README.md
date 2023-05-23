# Customisation for ACDH-CH Researchspace based projects

customisation of Researchspace for the purposes of Omnipot [#13886](https://redmine.acdh.oeaw.ac.at/issues/13886 "#13886") and other ResearchSpace based projects.

### How to use this Researchspace app

1. Go to Rancher in the project Researchspace and enter container named docker-git-alpine located in the namespace of your *$PROJECTNAME* by clicking on Execute Shell that can be found right from the workload name under three vertical dots. 
2. `cd /apps/`
3. `git clone --branch $PROJECTNAME --single-branch https://github.com/acdh-oeaw/rs-customisation.git $PROJECTNAME`
4. `cd /apps/$PROJECTNAME`
5. `mkdir -p config/services images assets lib`
6. `chown -R 100:101 /apps`
7. `vi /app/$PROJECTNAME/config/repositories` and add the username and the password for the triplestore database
8. `vi /apps/$PROJECTNAME/plugin.properties`  
Here is an example:
        plugin.id=$PROJECTNAME
        plugin.provider=ResearchSpace
        plugin.version=1.0.0
        #plugin.dependencies=other-app
9. Go to Rancher researchspace workload for *$PROJECTNAME* and edit it by clicking on the three vertical dots on the right side and the label Edit Config.
10. Click on the researchspace tab *(in the menu located on the left side, General should be selected)*, scrol down to Environment Variables section end edit **PLATFORM_OPTS** variable:

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
-DruntimeDirectory=/apps/$PROJECTNAME
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
-XX:MaxRAMFraction=1 
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
