RefererConfig.initRef();

```
ConfigHandler configHandler = ExtensionLoader.getExtensionLoader...

ClusterSupport<T> clusterSupport = createClusterSupport(refUrl, configHandler, registryUrls);
```

createClusterSupport
```
return configHandler.buildClusterSupport(interfaceClass, regUrls);
```

SimpleConfigHandler
```
    @Override
    public <T> ClusterSupport<T> buildClusterSupport(Class<T> interfaceClass, List<URL> registryUrls) {
        ClusterSupport<T> clusterSupport = new ClusterSupport<T>(interfaceClass, registryUrls);
        clusterSupport.init();

        return clusterSupport;
    }
```

ClusterSupport
```
public void init() {
	// client 注册自己，同时订阅service列表
     Registry registry = getRegistry(ru);
     registry.subscribe(subUrl, this);
}
```


CommandFailbackRegistry
```
        List<URL> urls = doDiscover(urlCopy);	// 发现
        if (urls != null && urls.size() > 0) {
            this.notify(urlCopy, listener, urls);	// 订阅
        }
```

ClusterSupport
```
Referer<T> referer = getExistingReferer(u, registryReferers.get(registryUrl));
            if (referer == null) {
                // careful u: serverURL, refererURL的配置会被serverURL的配置覆盖
                URL refererURL = u.createCopy();
                mergeClientConfigs(refererURL);
                referer = protocol.refer(interfaceClass, refererURL, u);
            }
            if (referer != null) {
                newReferers.add(referer);
            }
			refreshCluster();
```