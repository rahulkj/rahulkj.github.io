Spring profiles and tomcat
---

Lately, I ran into an issue where my spring application failed to start, when I deployed it to tomcat 7.

I received the following error:
```
org.springframework.beans.factory.NoSuchBeanDefinitionException: No matching bean of type [XXXXXXXXXXXXXXXXX] found for dependency: expected at least 1 bean which qualifies as autowire candidate for this dependency. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
```

After debugging for some time, I realized this had to do with spring profiles, which I had configured in my application.

### Configuring Tomcat to run applications with Spring Profiles

Now its time to tell Tomcat which profile is `active`. There are two ways to do this:
- defining the configuration in `web.xml`
- defining system property `-Dspring.profiles.active=your-active-profile`

For `windows`, create the file `setenv.bat` in Tomcat’s bin directory with the following contents:

```
JAVA_OPTS=%JAVA_OPTS% -Dspring.profiles.active=dev
```

For `linux`, create the file `setenv.sh` in Tomcat’s bin directory with the following contents:

```
JAVA_OPTS="$JAVA_OPTS -Dspring.profiles.active=dev"
```

Start tomcat using `startup.sh`, and there you go.. all the errors are gone and your application is running with the spring profile configured in `setenv.sh`/`setenv.bat`

If you want to achieve the same using the `web.xml` approach, update the `web.xml` as follows:
```
<context-param>
  <param-name>spring.profiles.active</param-name>
  <param-value>dev</param-value>
</context-param>
```
