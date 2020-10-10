# spring-social-apple [![Build Status](https://travis-ci.org/mihajul/spring-social-apple.svg)](https://travis-ci.org/mihajul/spring-social-apple) [![Maven Central](https://img.shields.io/maven-central/v/com.github.mihajul/spring-social-apple.svg)](https://maven-badges.herokuapp.com/maven-central/com.github.mihajul/spring-social-apple) 

## Table of Contents  
- [Introduction](#introduction)  
- [How to use](#how-to-use)  
- [Configuring](#configuring)  
- [Problems and Limitations](#problems)  
- [License](#license)  
- [Credits](#credits)  


## <a name="introduction">Introduction</a> [&#8593;](#toc)

A provider extension for Spring Social to enable connectivity with Apple Sign In (Apple ID) https://developer.apple.com/sign-in-with-apple/

## <a name="how-to-use">How to Use</a> [&#8593;](#toc)

* Maven Integration

If you're using Maven in your project, then you can integrate spring-social-apple by adding the following dependency in your pom.xml
	
```xml
<dependency>
   <groupId>com.github.mihajul</groupId>
   <artifactId>spring-social-apple</artifactId>
   <version>1.0.0</version>
</dependency>
```

If you prefer using the latest snapshot build, include the following lines to your pom.xml.
	
```xml
    <repositories>
        <repository>
            <id>oss.snapshots</id>
            <name>OSS Sonatype Snapshot Repository</name>
            <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
            <releases>
                <enabled>false</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>com.github.mihajul</groupId>
            <artifactId>spring-social-apple</artifactId>
             <version>1.0.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
```


## <a name="configuring">Configuring Spring-Social-Apple</a> [&#8593;](#toc)

1. Obtain the necessary ids and keys from apple:
	- Apple Certificate (.p8)
	- Apple Client ID
	- Apple Key ID
	- Apple Team ID
		
These all can be generated by going to keys and identifiers menu in Application Developer account of apple


2. Register the connection factory with spring social
```java
@Value("classpath:apple-AuthKey_XXXXXXX.p8")
private org.springframework.core.io.Resource appleCertificate;
	
...
	
//register your connection factory:
	
String appleSecret = "";
try {
	appleSecret = org.apache.commons.io.IOUtils.toString(
				appleCertificate.getInputStream(), StandardCharsets.UTF_8.name());
}catch(Exception e) {
	e.printStackTrace();
}
AppleConnectionFactory apple = new AppleConnectionFactory(
	env.getProperty("apple.clientId"),
	env.getProperty("apple.teamId"),
	env.getProperty("apple.keyId"),
	appleSecret
);
apple.setScope("name email");

cfConfig.addConnectionFactory(apple);
	
```


## <a name="problems">Problems and Limitations</a> [&#8593;](#toc)
There are some issues because the apple Oauth2 implementation differs from the spring social implementation (it works in a differnent way than google, facebook, twitter, etc. )
Because apple doesn't provide an API to get user data based on the authentication token, the only information available to us after the login is what we can extract from the signed JWT token (id_token) obtained after the authorization grant code validation:
- provider user id (unique identifier of the user assigned by apple)
- email

### Obtaining the user first and last name

On the first login into the application the when the apple server makes a POST to the redirect_url you specified ( for example http://your-server/auth/apple ) they include a "user" object 
like ```{"name":{"firstName":"XXX","lastName":"XXX"},"email":"xxx@xxx.com"}```

You can parse it using the following code:

```java
String userJson = request.getParameter("user");
AppleUserInfo userInfo = om.readValue(userJson, AppleUserInfo.class);
if(userInfo != null && userInfo.getName() != null) {
	profile.setFirstName(userInfo.getName().getFirstName());
	profile.setLastName(userInfo.getName().getLastName());
}
```

Apple only returns the user object the first time the user authorizes the app. Persist this information from your app; subsequent authorization requests won’t contain the user object.
More details here: https://stackoverflow.com/questions/63500926/apple-sign-in-authorize-method-returns-name-only-first-time

### Issue with the jwt key expiration

Apple doesn't use a secret key like other identity providers but requires a JWT token based on the certificate they supply.
The JWT token we generate has a maximum expiration time of 6 months.
Because Spring Social requires the secret key only when the connection factory is created, it will probably stop working if the server is not restarted in more than 6 months.

## <a name="license">License</a> [&#8593;](#toc)

Please check the [license](https://github.com/mihajul/spring-social-apple/blob/master/LICENSE) file.

## <a name="credits">Credits</a>

This plugin was inspired by spring-social-live: https://github.com/sachin-handiekar/spring-social-live
