



## AbstractApplicationContext出发事件

```java
protected void publishEvent(Object event, @Nullable ResolvableType eventType) {
    Assert.notNull(event, "Event must not be null");
    if (this.logger.isTraceEnabled()) {
        this.logger.trace("Publishing event in " + this.getDisplayName() + ": " + event);
    }

    Object applicationEvent;
    if (event instanceof ApplicationEvent) {
        applicationEvent = (ApplicationEvent)event;
    } else {
        applicationEvent = new PayloadApplicationEvent(this, event);
        if (eventType == null) {
            eventType = ((PayloadApplicationEvent)applicationEvent).getResolvableType();
        }
    }

    if (this.earlyApplicationEvents != null) {
        this.earlyApplicationEvents.add(applicationEvent);
    } else {
        this.getApplicationEventMulticaster().multicastEvent((ApplicationEvent)applicationEvent, eventType);
    }

    if (this.parent != null) {
        if (this.parent instanceof AbstractApplicationContext) {
            ((AbstractApplicationContext)this.parent).publishEvent(event, eventType);
        } else {
            this.parent.publishEvent(event);
        }
    }

}
```





## SimpleApplicationEventMulticaster中执行其中的listeners

getApplicationListeners(event, type))或者这个类型下的event的listeners

```java
public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
   ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
   for (final ApplicationListener<?> listener : getApplicationListeners(event, type)) {
      Executor executor = getTaskExecutor();
      if (executor != null) {
         executor.execute(() -> invokeListener(listener, event));
      }
      else {
         invokeListener(listener, event);
      }
   }
}
```





## AbstractApplicationContext中refresh

这里只是对ApplicationEventMulticaster的实现进行注册

```java
protected void initApplicationEventMulticaster() {
    ConfigurableListableBeanFactory beanFactory = this.getBeanFactory();
    if (beanFactory.containsLocalBean("applicationEventMulticaster")) {
        this.applicationEventMulticaster = (ApplicationEventMulticaster)beanFactory.getBean("applicationEventMulticaster", ApplicationEventMulticaster.class);
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Using ApplicationEventMulticaster [" + this.applicationEventMulticaster + "]");
        }
    } else {
        this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
        beanFactory.registerSingleton("applicationEventMulticaster", this.applicationEventMulticaster);
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Unable to locate ApplicationEventMulticaster with name 'applicationEventMulticaster': using default [" + this.applicationEventMulticaster + "]");
        }
    }

}
```