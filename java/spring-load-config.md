bean类

~~~
public class Blog {
    private String title;

    ... getter and settter
}
~~~

spring.xml配置  

```
    <bean id="blog" class="spring.bean.Blog">
        <property name="title" value="hello spring"></property>
    </bean>
```


test

```
	@Test
    public void test() {
        BeanFactory xmlBeanFactory = new XmlBeanFactory(new ClassPathResource("spring.xml"));
        Blog bean = (Blog)xmlBeanFactory.getBean("blog");

        Assert.assertEquals(bean.getTitle(), "hello spring");
    }
```


 `XmlBeanFactory`继承于`DefaultListableBeanFactory`，`DefaultListableBeanFactory`是整个bean加载的核心部分，是Spring注册及加载的bean的默认实现。 `XmlBeanFactory`只是使用了自定义的XML读取器`XmlBeanDefinitionReader`


## 加载配置

```
BeanFactory bf = new XmlBeanFactory(new ClassPathResource("spring.xml"));
```
加载过程大致示意图
![@startuml
XmlBeanFactory->XmlBeanDefinitionReader: loadBeanDefinitions(resource)
XmlBeanDefinitionReader->DefaultBeanDefinitionDocumentReader:doRegisterBeanDefinitions(document)
DefaultBeanDefinitionDocumentReader->BeanDefinitionParserDelegate:parseBeanDefinitionElement(ele)
BeanDefinitionParserDelegate->BeanDefinitionParserDelegate:parseBeanDefinitionElement(ele)
@enduml](http://www.plantuml.com/plantuml/png/fOsn3i8m44FtVWLZ6V837JAWm8mwTJqbfegKvj3bEl3tfA9Ba418ZBQVxMb99r2-a5UMXx7JIplSOeuQEO-W01aEYIcqIUa5XLVnE7RTXvwnrQ4rQHiwzkk2hFjuu15pB0fvVmWxM1z-63AsJQya1UAGC9DYk6-o9Su9MxslIBturlvl-ma0)


跟踪`XmlBeanFactory`的构造方法，

```
	public XmlBeanFactory(Resource resource, BeanFactory parentBeanFactory) ... {
		super(parentBeanFactory);
		this.reader.loadBeanDefinitions(resource);
	}
```

XmlBeanFactory创建了XmlBeanDefinitionReader，XmlBeanDefinitionReader.loadBeanDefinitions：

```
	public int loadBeanDefinitions(Resource resource) ... {
		return loadBeanDefinitions(new EncodedResource(resource));
	}

	public int loadBeanDefinitions(EncodedResource encodedResource) ... {
          ...
		InputStream inputStream = encodedResource.getResource().getInputStream();	//获取配置文件的InputStream
    	InputSource inputSource = new InputSource(inputStream);	// 构造InputSource
		return doLoadBeanDefinitions(inputSource, encodedResource.getResource());	// 解析InputSource
	}
```


```
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)     ... {
    Document doc = doLoadDocument(inputSource, resource);	// 获取Document对象
	return registerBeanDefinitions(doc, resource);	// 解析Document
}  
```

```
	public int registerBeanDefinitions(Document doc, Resource resource) ... {
		...
		BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();		// 创建documentReader对象
		documentReader.registerBeanDefinitions(doc, createReaderContext(resource));	// 解析documentReader
	}
```
createReaderContext() 方法创建XmlReaderContext对象，用于存储上下文信息

createBeanDefinitionDocumentReader()方法创建了DefaultBeanDefinitionDocumentReader对象，DefaultBeanDefinitionDocumentReader.registerBeanDefinitions:

```
	public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
		...
		Element root = doc.getDocumentElement();	// 获取root元素
		doRegisterBeanDefinitions(root);	// 解析root元素
	}

    protected void doRegisterBeanDefinitions(Element root) {
        ...
		this.delegate = createDelegate(getReaderContext(), root, parent);	// 创建delegate对象
		preProcessXml(root);
		parseBeanDefinitions(root, this.delegate);	// 
		postProcessXml(root);
    }
```
preProcessXml和postProcessXml都是模板方法，提供给子类实现。
createDelegate()方法创建了BeanDefinitionParserDelegate实例。

```
	protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
		if (delegate.isDefaultNamespace(root)) {
			NodeList nl = root.getChildNodes();
			for (int i = 0; i < nl.getLength(); i++) {
				Node node = nl.item(i);
				if (node instanceof Element) {
					Element ele = (Element) node;
					if (delegate.isDefaultNamespace(ele)) {
						parseDefaultElement(ele, delegate);	// 解析node结点
					}
					else {
						delegate.parseCustomElement(ele);
					}
				}
			}
		}
		else {
			delegate.parseCustomElement(root);
		}
	}
```
上述方法根据Namespace Uri判断node是否为Spring定义的元素，如果是，则调用parseDefaultElement方法解析元素。

```
	private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
		if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {	// 解析import标签
			importBeanDefinitionResource(ele);
		}
		else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {	// 解析alias标签
			processAliasRegistration(ele);
		}
		else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {	// 解析bean标签
			processBeanDefinition(ele, delegate);
		}
		else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {	// 解析beans标签
			doRegisterBeanDefinitions(ele);
		}
	}
```
`parseDefaultElement`方法对import，alias，bean，beans标签进行了解析，这里主要看bean的解析过程：

```
	protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
		BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);	// 解析元素,将创建BeanDefinitionHolder对象
		if (bdHolder != null) {
			bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);	// 装饰模式

				BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());	// 注册BeanDefinition，即将解析结果存储在registry对象中.
			
			// 发送事件
			getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
		}
	}
```

BeanDefinitionHolder中包含了BeanDefinition，   
BeanDefinition是配置文件<bean>元素标签在容器中的内部表示形式。
在配置文件中可以定义父<bean>和子<bean>，父<bean>或没有父<bean>的bean使用RootBeanDefinition表示，是最常用的表示。  
spring将配置文件中<bean>配置信息转化为beanDefinition对象，并注册到DeanDefinitionegistry中。

BeanDefinitionParserDelegate.parseBeanDefinitionElement


```
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, BeanDefinition containingBean) {
	AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);

	String beanClassName = beanDefinition.getBeanClassName();
}

public AbstractBeanDefinition parseBeanDefinitionElement(Element ele, String beanName, BeanDefinition containingBean) {
        
    ...
	className = ele.getAttribute(CLASS_ATTRIBUTE).trim(); // 解析className

	parent = ele.getAttribute(PARENT_ATTRIBUTE);


	AbstractBeanDefinition bd = createBeanDefinition(className, parent);

	parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);
	bd.setDescription(DomUtils.getChildElementValueByTagName(ele, DESCRIPTION_ELEMENT));

	parseMetaElements(ele, bd);
	parseLookupOverrideSubElements(ele, bd.getMethodOverrides());
	parseReplacedMethodSubElements(ele, bd.getMethodOverrides());

	parseConstructorArgElements(ele, bd);
	parsePropertyElements(ele, bd);
	parseQualifierElements(ele, bd);

	bd.setResource(this.readerContext.getResource());
	bd.setSource(extractSource(ele));
}
```

下面看一下属性的解析过程

```
	public void parsePropertyElements(Element beanEle, BeanDefinition bd) {
		NodeList nl = beanEle.getChildNodes();
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
			if (isCandidateElement(node) && nodeNameEquals(node, PROPERTY_ELEMENT)) {
				parsePropertyElement((Element) node, bd);
			}
		}
	}
```
遍历所有的子元素，如果是property结点，则进行处理

```
public void parsePropertyElement(Element ele, BeanDefinition bd) {
	...
	Object val = parsePropertyValue(ele, bd, propertyName);
	PropertyValue pv = new PropertyValue(propertyName, val);
	parseMetaElements(ele, pv);
	pv.setSource(extractSource(ele));
	bd.getPropertyValues().addPropertyValue(pv);		
}

public Object parsePropertyValue(Element ele, BeanDefinition bd, String propertyName) {
		boolean hasRefAttribute = ele.hasAttribute(REF_ATTRIBUTE);
		boolean hasValueAttribute = ele.hasAttribute(VALUE_ATTRIBUTE);


		if (hasRefAttribute) {
			String refName = ele.getAttribute(REF_ATTRIBUTE);
			if (!StringUtils.hasText(refName)) {
				error(elementName + " contains empty 'ref' attribute", ele);
			}
			RuntimeBeanReference ref = new RuntimeBeanReference(refName);
			ref.setSource(extractSource(ele));
			return ref;
		}
		else if (hasValueAttribute) {
			TypedStringValue valueHolder = new TypedStringValue(ele.getAttribute(VALUE_ATTRIBUTE));
			valueHolder.setSource(extractSource(ele));
			return valueHolder;
		}
		else if (subElement != null) {
			return parsePropertySubElement(subElement, bd);
		}
		else {
			// Neither child element nor "ref" or "value" attribute found.
			error(elementName + " must specify a ref or value", ele);
			return null;
		}
}
```


最后看一下registerBeanDefinition：

```
				BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());



	public static void registerBeanDefinition(
			BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
			throws BeanDefinitionStoreException {

		// Register bean definition under primary name.
		String beanName = definitionHolder.getBeanName();
		registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

		// Register aliases for bean name, if any.
		String[] aliases = definitionHolder.getAliases();
		if (aliases != null) {
			for (String alias : aliases) {
				registry.registerAlias(beanName, alias);
			}
		}
	}
		
```

关键为`getReaderContext().getRegistry()`
```
	public final BeanDefinitionRegistry getRegistry() {
		return this.reader.getRegistry();
	}

```

XmlBeanDefinitionReader创建ReaderContext

```
	public XmlReaderContext createReaderContext(Resource resource) {
		return new XmlReaderContext(resource, this.problemReporter, this.eventListener,
				this.sourceExtractor, this, getNamespaceHandlerResolver());
	}
```
XmlBeanDefinitionReader把自身this做为参数创建	XmlBeanDefinitionReader		

XmlBeanFactory在创建XmlBeanDefinitionReader时也将自身作BeanDefinitionRegistry参数传递给XmlBeanDefinitionReader构造方法，所以`getReaderContext().getRegistry()`将返回xmlBeanFactory

xmlBeanFactory并没有重写registerBeanDefinition方法，DefaultListableBeanFactory.registerBeanDefinition:  

~~~java
	@Override
	public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {
		...
		this.beanDefinitionMap.put(beanName, beanDefinition);

	}	
~~~

