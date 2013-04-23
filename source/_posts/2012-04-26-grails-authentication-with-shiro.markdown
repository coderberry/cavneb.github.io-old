---
layout: post
title: "Grails Authentication with Shiro"
date: 2012-04-26 18:03
comments: true
categories: 
- Grails
- Authentication
- Shiro
---

> Source code for the example project can be found at [https://github.com/cavneb/grails-shiro-example](https://github.com/cavneb/grails-shiro-example).

[Shiro](http://shiro.apache.org) is a security framework that is meant to make life easier. The goal of it is to allow the developer to perform *authentication, authorization, cryptography* and *session management*. After reading over the [10 minute tutorial](http://shiro.apache.org/10-minute-tutorial.html), I trust that it is an excellent option to use in my Grails application. It should be also noted that the [Grails.org](http://www.grails.org) website also uses Shiro for all authentication.

My goal for this article is to help people understand how to implement the Shiro plugin into their Grails application as simply as possible. I will also cover some common usages of the plugin for reference.

## A bit more about Shiro

I strongly suggest going through the [documentation](http://shiro.apache.org/documentation.html) for Shiro before continuing. If not, there are a couple of tidbits that will help you understand how it works.

First, a term that Shiro uses for a *User* is *Subject*. Anytime the *Subject* is referenced, it is referring to Shiro's version of a User class.

Second, there is always a *Subject* that represents the current user. The *Subject* can be handled prior to authentication. This means that if someone was to visit your site without any authentication, there is already an instanciated class that represents them. In Java, you can obtain the *Subject* with the following:

```java
Subject currentUser = SecurityUtils.getSubject();
```

To authenticate the user, you must create a `UserPasswordToken` and call `login` on the instanciated *Subject*. The success or failure of the login are returned via [exceptions](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/AuthenticationException.html). Here is an example of code that may be used when logging in a user.

```java
if ( !currentUser.isAuthenticated() ) {
    UsernamePasswordToken token = new UsernamePasswordToken("foo", "bar");
    token.setRememberMe(true); // Use remember me if desired - all built in!

    // Attempt to authenticate. If no exception thrown, the login was successful
    try {
      currentUser.login( token );
    
    // Username wasn't in the system
    } catch ( UnknownAccountException uae ) {
      
    // Password didn't match
    } catch ( IncorrectCredentialsException ice ) {
      
    // Account with username is locked
    } catch ( LockedAccountException lae ) {

    // Unexpected error      
    } catch ( AuthenticationException ae ) {

    }
}
```

Cool! Now we have a bit of knowledge to help us know what we are doing in our Grails implementation.

## Installation

Let's start by creating a new Grails application. I am using version 2.0.3. In the code samples to follow, anything that starts with a $ means that it is a shell command.

<pre><code>$ grails create-app shiro-example
$ cd shiro-example</code></pre>

The [Shiro Plugin](http://grails.org/plugin/shiro) can be installed by using the dependency in your `BuildConfig.groovy`. Even though the plugin page shows version 1.1.4, we will use the latest snapshot release:

{% codeblock grails-app/conf/BuildConfig.groovy lang:java %}
    plugins {
        ...
        runtime ":shiro:1.2.0-SNAPSHOT"
        ...
{% endcodeblock %}

Once you have the dependency setup in the `BuildConfig.groovy` file, you must compile the app to make sure the dependencies are installed.

<pre><code>$ grails compile</code></pre>

Let's kickstart our app now by using the `shiro-quick-start` command. I am using the optional `--prefix=[package]`. Make sure you have a dot at the end if you are using the prefix.

<pre><code>$ grails shiro-quick-start --prefix=com.example.
| Created file grails-app/domain/com/example/User.groovy
| Created file grails-app/domain/com/example/Role.groovy
| Created file grails-app/realms/com/example/DbRealm.groovy
| Created file grails-app/controllers/com/example/AuthController.groovy
| Created file grails-app/views/auth/login.gsp
| Created file grails-app/conf/com/example/SecurityFilters.groovy</code></pre>

Whoa! What just happened? As you can see, you now have created some files:

`domain/../User.groovy` is the *domain class* for your user.

`domain/../Role.groovy` is the *domain class* for your user roles.

`realms/../DbRealm.groovy` is the class Shiro uses to handle authentication and permissions.

`controllers/../AuthController.groovy` and `views/auth/login.gsp` are the user-facing controller and view meant to handle login.

`config/../SecurityFilters.groovy` adds a [beforeInterceptor](http://grails.org/doc/2.0.3/ref/Controllers/beforeInterceptor.html) to all url's. This blocks/redirects users from accessing pages they are not authorized to visit (based on their roles).

> **What is a realm?**<br/>
> Shiro can use different *realms* for authenticating a user. A *realm* is basically a resource Shiro should use to authenticate a user. Further explanation can be found [here](http://www.brucephillips.name/blog/index.cfm/2009/4/5/An-Introduction-to-Ki-formerly-JSecurity--A-Beginners--Tutorial-Part-2).

## Configuration

## Bootstrapping

Bootstrapping users and roles is similar to how it's done with the [Spring Security Core plugin](http://grails.org/plugin/spring-security-core). Let's create a couple of roles and users to go with them.

{% include_code BootStrap.groovy lang:java %}

On line 5, we include the Shiro Security Service. This is necessary for us to encode the password into a password hash for the user.

On lines 9-15, the admin and user roles are created.

On lines 17-21, the admin user is either loaded from the database or is created. Here is where we use the Shiro Security Service for encoding the password.

On lines 23-26, the roles *ADMIN_ROLE* and *USER_ROLE* are assigned to the admin user. Note that we add both roles to this user. This allows the admin user to act as a normal user as well. This may or may not be appropriate, depending on the application.

Lines 28-36 are similar to 23-26, but for the standard user role instead of the admin role.

## Kick the Tires

Now that we have the default users created, let's run the app and test to make sure the login works. Start up the application with the `run-app` command:

<pre><code>$ grails run-app</code></pre>

Once it's running, open up the browser and go to <a href="http://localhost:8080/shiro-example/" target="_blank">http://localhost:8080/shiro-example/</a>.

{% img /images/posts/shiro-example-1.png %}

Click on the link *com.example.AuthController* and login as the admin user (username: 'admin', password: 'password').

{% img /images/posts/shiro-example-2.png %}

If you successfully logged in, you will be redirected back to the first page. This doesn't help us very much.

## Adding Security

Now that we've figured out how to authenticate users with the login form, we need to create some pages that require being authenticated. Pay attention.. this is really the core of the plugin.. and the part that was most difficult for me to figure out.

Let's start off by creating a controller with three actions: `index`, `secured` and `admin`.

<pre><code>$ grails create-controller com.example.Home</code></pre>

Create three actions in the controller:

{% codeblock grails-app/controllers/com/example/HomeController.groovy lang:java %}
package com.example

class HomeController {

    def index() {
        render "This page is not secured"
    }

    def secured() {
       render "This page requires a user to be logged in"
    }

    def admin() {
       render "This page requires the logged in user to be an administrator"
    }

}
{% endcodeblock %}

As you can see, the three actions *should* have different security levels. `index` will allow anyone to be able to view the page. `secured` requires that a user be logged in. `admin` requries that the user logged in must have the *ROLE_ADMIN* role assigned to them. 

Try out the URL's to the new actions created. Notice that they all lead you back to the login page if you haven't logged in. Stranger yet is that once you log in, you should see this;

{% img /images/posts/shiro-example-3.png %}

Don't worry though. This is expected and can be resolved easily by updating `SecurityFilters.groovy`.

```java
package com.example

/**
 * Generated by the Shiro plugin. This filters class protects all URLs
 * via access control by convention.
 */
class SecurityFilters {

    /**
     * Array of controller/action combinations which will be skipped from authentication
     * if the controller and action names match. The action value can also be '*' if it
     * encompasses all actions within the controller.
     */
    static nonAuthenticatedActions = [
            [controller: 'home', action: 'index']
    ]

    /**
     * Array of controller/action combinations that will be authenticated against the user's
     * role. The map also includes the roles which the controller/action pair will match
     * against.
     */
    static authenticatedActions = [
            [controller: 'home', action: 'secured', roles: ['ROLE_ADMIN', 'ROLE_USER']],
            [controller: 'home', action: 'admin', roles: ['ROLE_ADMIN']]
    ]

    def filters = {

        all(controller: '*', action: '*') {
            before = {

                // Determine if the controller/action belongs is not to be authenticated
                def needsAuth = !nonAuthenticatedActions.find {
                    (it.controller == controllerName) &&
                            ((it.action == '*') || (it.action == actionName))
                }

                if (needsAuth) {

                    // Get the map within the authenticated actions which pertain to the current
                    // controller and view.
                    def authRoles = authenticatedActions.find {
                        (it.controller == controllerName) &&
                                ((it.action == '*') || (it.action == actionName))
                    }

                    if (authRoles) {

                        // Perform the access control for each of the roles provided in the authRoles
                        accessControl {
                            authRoles.roles.each { roleName ->
                                role(roleName)
                            }
                        }
                    }

                    // Skip authentication if the authRoles was not found
                    else {
                        return true
                    }
                }

                // Skip authentication if no auth is needed
                else {
                    return true
                }
            }
        }

    }
}
```

This may be a lot to take in but the only sections that are really important are lines 14-16 and 23-26. Here we are establishing which controller/action combinations are not to be authenticated, and which are to be authenticated against the provided roles. 

Now when you access the pages, the expected security should be in place. Give it a shot.

> **Confused?**<br/>
> This can be a confusing topic, and moreso a confusing portion of the topic. Please feel free to ask any questions in the comments section and I will do my best to help out.*

## Creating Users

The `shiro-quick-start` command did create the login page and authentication controller, but it didn't help us a bit on creating users. Good thing it's pretty easy to do. 

We could create a controller called *Users* to handle the signup process, however, I don't want to do that. The reason is because I would like to eventually have a *Users* controller that will perform all the CRUD operations for the users and would only be accessible by administrators. 

Instead, let's create a new controller for handling the signup process. Let's call it `Signup`.

<pre><code>$ grails create-controller com.example.Signup
| Created file grails-app/controllers/com/example/SignupController.groovy
| Created file grails-app/views/signup
| Created file test/unit/com/example/SignupControllerTests.groovy</code></pre>

Update your SignupController as follows:

```java
package com.example

import org.apache.shiro.authc.UsernamePasswordToken
import org.apache.shiro.SecurityUtils

class SignupController {
    def shiroSecurityService

    def index() {
        User user = new User()
        [user: user]
    }

    def register() {

        // Check to see if the username already exists
        def user = User.findByUsername(params.username)
        if (user) {
            flash.message = "User already exists with the username '${params.username}'"
            render('index')
        }

        // User doesn't exist with username. Let's create one
        else {

            // Make sure the passwords match
            if (params.password != params.password2) {
                flash.message = "Passwords do not match"
                render('index')
            }

            // Passwords match. Let's attempt to save the user
            else {
                // Create user
                user = new User(
                        username: params.username,
                        passwordHash: shiroSecurityService.encodePassword(params.password)
                )

                if (user.save()) {

                    // Add USER role to new user
                    user.addToRoles(Role.findByName('ROLE_USER'))

                    // Login user
                    def authToken = new UsernamePasswordToken(user.username, params.password)
                    SecurityUtils.subject.login(authToken)

                    redirect(controller: 'home', action: 'secured')
                }
                else {

            }
            }
        }
    }
}
```

In the code above, there are two actions: `index` and `register`. The `index` action is what contains the form (view). The `register` action performs the user registration and responds/redirects according to the results.

Paste this into your `/grails-app/views/home/index.gsp` view:

```html
<%@ page contentType="text/html;charset=UTF-8" %>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
    <title>Register</title>
    <meta name="layout" content="main"/>
</head>

<body>

<h1>Signup</h1>

<g:if test="${flash.message}">
    <div class="alert alert-info">${flash.message}</div>
</g:if>

<g:hasErrors bean="${user}">
    <div class="alert alert-error">
        <g:renderErrors bean="${user}" as="list"/>
    </div>
</g:hasErrors>

<g:form action="register">
    <p>
        <label for="username">Username</label>
        <g:textField name="username" value="${user.username}"/>
    </p>

    <p>
        <label for="password">Password</label>
        <g:passwordField name="password" value=""/>
    </p>

    <p>
        <label for="password">Confirm Password</label>
        <g:passwordField name="password2" value=""/>
    </p>

    <p>
        <g:submitButton name="submit" value="Submit"/>
    </p>
</g:form>

</body>
</html>
```

Go ahead and try the signup by going to [http://localhost:8080/shiro-example/signup](http://localhost:8080/shiro-example/signup).

{% img /images/posts/shiro-example-4.png %}

If the signup is successful, you will see a page like this:

{% img /images/posts/shiro-example-5.png %}

Congrats! You now have a full circle authenticated app using **Shiro**!

## Tag Lib Reference

```
<shiro:isLoggedIn>Body</shiro:isLoggedIn>
```
This tag only writes its body to the output if the current user is logged in.

```
<shiro:isNotLoggedIn>Body</shiro:isNotLoggedIn>
```
This tag only writes its body to the output if the current user is NOT logged in.

```
<shiro:authenticated />
``` 
A synonym for 'isLoggedIn'. This is the same name as used by the standard Shiro tag library.

```
<shiro:notAuthenticated />
```
A synonym for 'isNotLoggedIn'. This is the same name as used by the standard Shiro tag library.

```
<shiro:user>Body</shiro:user>
```
This tag only writes its body to the output if the current user is either logged in or remembered from a previous session (via the "remember me" cookie).

```
<shiro:notUser>Body</shiro:notUser>
``` 
This tag only writes its body to the output if the current user is neither logged in nor remembered from a previous session (via the "remember me" cookie).

```
<shiro:remembered>Body</shiro:remembered>
``` 
This tag only writes its body to the output if the current user is remembered from a previous session (via the "remember me" cookie) but not currently logged in.

```
<shiro:notRemembered>Body</shiro:notRemembered>
```
This tag only writes its body to the output if the current user is not remembered from a previous session (via the "remember me" cookie). This is the case if they are a guest user or logged in.

```
<shiro:principal type="type" property="property" />
```
Outputs the string form of the current user's identity. By default this assumes that the subject has only one principal; its string representation is written to the page. 

The ***type*** is the species the type or class of the principal to use. 

The ***property*** value specifies the name of the property on the principal to use as the string representation, e.g. `firstName`.

```
<shiro:hasRole name="role_name">Body</shiro:hasRole>
``` 
This tag only writes its body to the output if the current user has the given role. 

The ***name*** property is a the *String* name of the role, e.g. `ROLE_USER`.

```
<shiro:lacksRole name="role_name">Body</shiro:lacksRole>
```
This tag only writes its body to the output if the current user does not have the given role. 

The ***name*** property is a the *String* name of the role, e.g. `ROLE_USER`.

```
<shiro:hasAllRoles in="roles">Body</shiro:hasAllRoles>
```
This tag only writes its body to the output if the current user has all the given roles (inversion of lacksAnyRole). 

The ***in*** property is a *List<String>* of roles, e.g. `['ROLE_ADMIN', 'ROLE_USER']`.

```
<shiro:lacksAllRoles in="roles">Body</shiro:lacksAllRoles>
``` 
This tag only writes its body to the output if the current user doesn't have all of the given roles (inversion of hasAllRoles). 
The ***in*** property is a *List<String>* of roles, e.g. `['ROLE_ADMIN', 'ROLE_USER']`.

```
<shiro:hasAnyRole in="roles">Body</shiro:hasAnyRole>
```
This tag only writes its body to the output if the current user has any of the given roles (inversion of lacksAllRoles). 

The ***in*** property is a *List<String>* of roles, e.g. `['ROLE_ADMIN', 'ROLE_USER']`.

```
<shiro:lacksAnyRole in="roles">Body</shiro:lacksAnyRole>
```
This tag only writes its body to the output if the current user has none of the given roles (inversion of hasAllRoles). 

The ***in*** property is a *List<String>* of roles, e.g. `['ROLE_ADMIN', 'ROLE_USER']`.

```
<shiro:hasPermission type="type" 
                     permission="permission" 
                     actions="actions" 
                     target="target">Body</shiro:hasPermission>
```
This tag only writes its body to the output if the current user has the given permission. 

The ***type*** property is the permission type. 

The ***permission*** is the permission name. Either *type* or *permission* must be used. 

The ***actions*** must be present if the *type* is present. 

The *target* is optional and will be defaulted to '*' if not provided. It is also only applicable for when the *type* attribute is available.

```
<shiro:hasPermission type="type" 
                     permission="permission" 
                     actions="actions" 
                     target="target">Body</shiro:hasPermission>
```
This tag only writes its body to the output if the current user has the given permission. Attribute descriptions are the same as found on `hasPermissions` above.

```
<shiro:hasPermission type="type" 
                      permission="permission" 
                      actions="actions" 
                      target="target">Body</shiro:hasPermission>
```
This tag only writes its body to the output if the current user does not have the given permission. Attribute descriptions are the same as found on `hasPermissions` above.

## Notes

I did not cover topics such as permissions. For more information on this, please refer to the [Shiro documentation](http://shiro.apache.org/documentation.html) and the [Shiro Plugin](http://grails.org/plugin/shiro) source code.

[Gavin Hogan](https://github.com/gavinhogan) has forked the grails-shiro plugin and has done some excellent work on updating the documentation. His fork can be found at [https://github.com/gavinhogan/grails-shiro/](https://github.com/gavinhogan/grails-shiro/).
