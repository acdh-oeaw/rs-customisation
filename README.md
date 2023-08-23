# Customisation for ACDH-CH Researchspace based projects

customisation of Researchspace for the purposes of Omnipot [#13886](https://redmine.acdh.oeaw.ac.at/issues/13886 "#13886") and other ResearchSpace based projects.

## How to install Researchspace in ACDH-CH K8S environment

1. Create new namespace for the $PROJECTNAME
2. Create workflow named researchspace by using Docker image researchspace/platform-ci:latest with persistent volumes mounted as /images and /tmp-data. Add secret for shiro.ini and mount it under /srv/shiro.ini 
3. Create service named $PROJECTNAME for the worload researchspace that should have Listening and Target port set to 8080.
4. Create Ingress for the service $PROJECTNAME with the desired domain.
5. Create workflow named digilib by using Docker image robcast/digilib:latest and mount PVC named images, created in previous step, with the Mount Point "/var/lib/images" and Sub Path in Volume "file" 
6. Create service for the workload named digilib. Ingress ins not needed.
7. Create backup cronjob that should backup /images, /tmp-data and /srv from the researchspace workload.
8. Go to Rancher researchspace workload for *$PROJECTNAME* and edit it by clicking on the three vertical dots on the right side and the label Edit Config.
9. Click on the researchspace tab *(in the menu located on the left side, General should be selected)*, scroll down to Environment Variables section end edit **PLATFORM_OPTS** variable:

For using GIT as runtime storage, the following PLATFORM_OPTS Rancher ENV variable can be used: 

```
-Dconfig.storage.runtime.type=git
-Dconfig.storage.runtime.mutable=true
-Dconfig.storage.runtime.branch=$PROJECTNAME
-Dconfig.storage.runtime.localPath=/apps
-Dconfig.storage.runtime.subroot=custom-app
-Dconfig.storage.runtime.remoteUrl=https://acdh-ch:$GITHUB-CREDENTIALS@github.com/acdh-oeaw/rs-customisation.git
-Dconfig.environment.sparqlEndpoint=http://$DB_USER:$DB_CREDENTIALS@$TRIPLESTORE:8080/$DB_NAME/sparql
-Dconfig.storage.images.type=nonVersionedFile   
-Dconfig.storage.images.mutable=true   
-Dconfig.storage.images.root=/images   
-Dconfig.storage.tmp.type=nonVersionedFile  
-Dconfig.storage.tmp.mutable=true   
-Dconfig.storage.tmp.root=/tmp-data   
-Dconfig.environment.shiroConfig=/srv/shiro.ini   
-Dorg.eclipse.jetty.server.Request.maxFormContentSize=1000000
```

To use Java ShenandoahGC add following configuration to the **JAVA_OPTS** variable:

```shell
-XX:+UnlockExperimentalVMOptions
-XX:+UseContainerSupport 
-XX:+UseShenandoahGC 
-XX:ShenandoahGCHeuristics=compact 
-XX:+UseStringDeduplication 
-XX:+ExitOnOutOfMemoryError
```

## How to use this repo as GIT runtime storage for Researchspaced based projects

1. Create new branch named $PROJECTNAME from the main branch. 
2. `edit custom-app/plugin.properties`  

Here is an example:

```shell
        plugin.id=$PROJECTNAME
        plugin.provider=ResearchSpace
        plugin.version=1.0.0
        #plugin.dependencies=other-app
```

3. edit templates over the Rancher GUI. All changes will be commited to the branch $PROJECTNAME

**NOTE:**

On the first run you need to import set of default ontologies and knowledge patterns for CIDOC-CRM, see [instance](https://github.com/researchspace/researchspace-instance-configurations "instance") configuration repository. 

##### References:
- https://documentation.researchspace.org/resource/Help:Apps
- https://documentation.researchspace.org/resource/Help:Storage
- https://documentation.researchspace.org/resource/Help:GitStorageGuide
- https://github.com/researchspace/researchspace-instance-configurations
