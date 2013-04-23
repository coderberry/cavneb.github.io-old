---
layout: post
title: "Grails Domain Classes and Enums"
date: 2012-04-23
comments: true
categories: 
- Groovy
- Grails 
---

For a long time, I have been creating static int values to represent states in domain classes. For example, I would have something like the following:

```java
package org.example

class PluginPendingApproval {
  
  static final int STATUS_PENDING  = 1
  static final int STATUS_APPROVED = 2
  static final int STATUS_REJECTED = 3

  int status

  static constraints = {
    status blank: false, inList: [
      STATUS_PENDING_APPROVAL, STATUS_APPROVED, STATUS_REJECTED
    ]
  }

}
```

Peter Ledbrook ( [@pledbrook](https://twitter.com/#!/pledbrook) ) gave me some tips on how to handle this better using `Enums`. Enum values should always be in uppercase letters because they are constants in the fixed set. If we were to refactor the code above, it would something more like this:


```java
package org.example

class PluginPendingApproval {
  
  enum ApprovalStatus {
    PENDING, APPROVED, REJECTED
  }

  ApprovalStatus status

  static constraints = {
    status blank: false
  }

}
```

This works great for a single domain class, but what if I wanted to use the same Enum in two different domain classes. For example:

```java
// PluginPendingApproval.groovy
package org.example

class PluginPendingApproval {

  static hasMany = [pluginPendingApprovalRequests: PluginPendingApprovalRequest]
  
  enum ApprovalStatus {
    PENDING, APPROVED, REJECTED
  }

  ApprovalStatus status

  static constraints = {
    status blank: false
  }

}

// PluginPendingApprovalRequest.groovy
package org.example

class PluginPendingApprovalRequest {

  static belongsTo = [pluginPendingApproval: PluginPendingApproval]
  
  enum ApprovalStatus {
    PENDING, APPROVED, REJECTED
  }

  ApprovalStatus status

  static constraints = {
    status blank: false
  }

}
```

I could keep do it this way where I have the enum code duplicated in two places, but that doesn't follow the DRY principal.

Let's refactor this and create a new Enum class that both domain classes can use:

```java
package org.example

public enum ApprovalStatusEnum {
    PENDING, APPROVED, REJECTED
}
```

Now both of my domain classes can access the same Enum:

```java
// PluginPendingApproval.groovy
package org.example

class PluginPendingApproval {

  static hasMany = [pluginPendingApprovalRequests: PluginPendingApprovalRequest]

  ApprovalStatus status

  static constraints = {
    status blank: false
  }

}

// PluginPendingApprovalRequest.groovy
package org.example

class PluginPendingApprovalRequest {

  static belongsTo = [pluginPendingApproval: PluginPendingApproval]

  ApprovalStatus status

  static constraints = {
    status blank: false
  }

}
```

For more information on Enums in Groovy, see [http://groovy.codehaus.org/Using+Enums](http://groovy.codehaus.org/Using+Enums)
