---
layout: post
title: "Multi-Associative GORM Criteria with Projections"
date: 2011-03-29
comments: true
categories:  
- Grails
- GORM
---

I have three domain classes: Lead, Submission and BuyerLog:

``` java
class Lead {
  static hasMany = [ submissions: Submission ]
  Date dateCreated
  // ...
}

class Submission {
  static belongsTo = [ lead: Lead ]
  static hasMany = [ buyerLogs: BuyerLog ]
  Lead lead
  // ...
}

class BuyerLog {
  static belongsTo = [ submission: Submission ]
  Submission submission
  String leadBuyer
  // ...
}
```
    
I have a need to get the number of duplicate leads which share the same leadBuyer (in the BuyerLog domain class). Here is the SQL:

``` sql
SELECT count(l.id)
  FROM lead AS l, submission AS s, buyerLog as bl
    WHERE l.id = s.leadId
      AND s.id = bl.submissionId
      AND bl.leadBuyer = $buyerName
      AND l.id != $lead.id
      AND l.dateCreated::date > $daysAgo
```
          
I want to do this using GORM / Criteria Builder. Here's my final code:

``` java
/**
 * Count duplicate submissions within 45 days
 * SELECT count(l.id)
 *      FROM lead AS l, submission AS s, buyerLog as bl
 *      WHERE l.id = s.leadId
 *          AND s.id = bl.submissionId
 *          AND bl.leadBuyer = $buyerName
 *          AND l.id != $lead.id
 *          AND l.dateCreated::date > $daysAgo
 */
protected def Boolean isDuplicateSubmission(Lead lead, ArrayList<String> buyerNames) {
  def isDuplicate = false
  def daysAgo = new Date() - 45
  def cnt = Lead.withCriteria {
    not {
      idEq(lead.id)
    }
    and {
      le('dateCreated', daysAgo)
      submissions {
        buyerLogs {
          inList('leadBuyer', buyerNames)
        }
      }
    }
    projections {
      rowCount()
    }
  }
  return (cnt > 0)
}
```
    
Thanks to schmolly159 on the #grails freenode IRC channel for the examples and continued help.
