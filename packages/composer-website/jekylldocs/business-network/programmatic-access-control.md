---
layout: default
title: Programmatic access control
category: tasks
section: business-network
sidebar: sidebars/accordion-toc0.md
excerpt: Transaction processor functions can be used to [**implement participant-based access control**](../managing/current-participant.html) by checking the participant type and identifier.
index-order: 508
---

# Programmatic access control

It is recommended that you use declarative access control to implement access control rules in your business network definition.
However, you can implement programmatic access control in your transaction processors by retrieving and testing either the current participant or the current identity.
You can run tests against the properties of the current participant or the current identity to permit or reject the execution of a transaction processor function.

A transaction processor function can call the `getCurrentParticipant` function to get the current participant:

```javascript
let currentParticipant = getCurrentParticipant();
```

The current participant is an instance of a modelled participant from the business network definition, or an instance of the system type `org.hyperledger.composer.system.NetworkAdmin`.

A transaction processor function can call the `getCurrentIdentity` function to get the current identity:

```javascript
let currentIdentity = getCurrentIdentity();
```

The current identity is an instance of the system type `org.hyperledger.composer.system.Identity`, which represents an identity within a deployed business network.

## Before you start

Before you follow these steps, you must have modeled a participant in a business
network definition and deployed it as a business network. You must have created
some instances of those participants, and issued those participants with identities.

The procedure below shows an example using the following participant models:

```
namespace net.biz.digitalPropertyNetwork

participant Person identified by personId {
  o String personId
  o String firstName
  o String lastName
}

participant PrivilegedPerson extends Person {

}
```

## Procedure

1. In your transaction processor function, verify the type of the current participant
   meets the requirements by using the `getCurrentParticipant` function:

   ```javascript
   async function onPrivilegedTransaction(privilegedTransaction) {
       let currentParticipant = getCurrentParticipant();
       if (currentParticipant.getFullyQualifiedType() !== 'net.biz.digitalPropertyNetwork.PrivilegedPerson') {
           throw new Error('Transaction can only be submitted by a privileged person');
       }
       // Current participant must be a privileged person to get here.
   }
   ```

2. In your transaction processor function, verify the participant ID of the current
   participant by using the `getCurrentParticipant` function:

   ```javascript
   async function onPrivilegedTransaction(privilegedTransaction) {
       let currentParticipant = getCurrentParticipant();
       if (currentParticipant.getFullyQualifiedIdentifier() !== 'net.biz.digitalPropertyNetwork.Person#PERSON_1') {
           throw new Error('Transaction can only be submitted by person 1');
       }
       // Current participant must be person 1 to get here.
   }
   ```

   The participant ID of the current participant can be compared to a participant
   that is linked to an asset (by a relationship) to verify that the current
   participant has the authority to access or modify an asset:

   ```javascript
   async function onPrivilegedTransaction(privilegedTransaction) {
       // Get the owner of the asset in the transaction.
       let assetOwner = privilegedTransaction.asset.owner;
       let currentParticipant = getCurrentParticipant();
       if (currentParticipant.getFullyQualifiedIdentifier() !== asset.owner.getFullyQualifiedIdentifier()) {
           throw new Error('Transaction can only be submitted by the owner of the asset');
       }
       // Current participant must be the owner of the asset to get here.
   }
   ```

3. In your transaction processor function, verify the certificate of the current identity
   meets the requirements by using the `getCurrentIdentity` function:

   ```javascript
   async function onPrivilegedTransaction(privilegedTransaction) {
       let currentIdentity = getCurrentIdentity();
       // Get the PEM encoded certificate from the current identity.
       let certificate = currentIdentity.certificate;
       // Perform testing on the PEM encoded certificate.
       if (!certificate.match(/^----BEGIN CERTIFICATE----/)) {
            throw new Error('Transaction can only be submitted by a person with a valid certificate');
       }
       // Current identity must have a valid certificate to get here.
   }
   ```