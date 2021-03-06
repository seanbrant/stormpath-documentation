.. _account-mgmt:

**********************
4. Account Management
**********************

a. Modeling Your User Base
===========================

The first question that we need to address is how we are going to model our users inside Stormpath. User Accounts in Stormpath aren't directly associated with Applications, but only indirectly via **Directories** and also possibly **Groups**. 

All of your Accounts will have to be associated with at least one Directory resource, so we can start there.  

.. _directory-mgmt:

i. Directories
--------------
    
The **Directory** resource is a top-level container for Account and Group resources. A Directory also manages security policies (like password strength) for the Accounts it contains. Directories can be used to cleanly manage segmented user Account populations. For example, you might use one Directory for company employees and another Directory for customers, each with its own security policies.

For more detailed information about the Directory resource, please see the :ref:`ref-directory` section in the Reference chapter.

Types of Directories
^^^^^^^^^^^^^^^^^^^^
Stormpath supports three types of Directories:

1. Natively-hosted Cloud Directories that originate in Stormpath
2. Mirror Directories that act as secure replicas of existing LDAP user directories outside of Stormpath, for example those on Active Directory servers.
3. Social Directories that pull in account information from four sites that support social login: Google, Facebook, Github and LinkedIn.
   
You can add as many Directories of each type as you require.

.. note::

  Multiple Directories are a more advanced feature of Stormpath. If you have one or more applications that all access the same Accounts, you usually only need a single Directory, and you do not need to be concerned with creating or managing multiple Directories.

  If however, your application(s) needs to support login for external third-party accounts like those in Active Directory, or you have more complex account segmentation needs, Directories will be a powerful tool to manage your application's user base.

.. _about-cloud-dir:

Cloud Directories
^^^^^^^^^^^^^^^^^
The standard, default Directory resource. They can be created using a simple POST API.

How to Make a Cloud Directory
"""""""""""""""""""""""""""""

The following API request:

.. code-block:: http

  POST /v1/directories HTTP/1.1
  Host: api.stormpath.com
  Content-Type: application/json;charset=UTF-8

  {
    "name" : "Captains",
    "description" : "Captains from a variety of stories"
  }

Would yield the following response:

.. code-block:: HTTP 

  HTTP/1.1 201 Created
  Location: https://api.stormpath.com/v1/directories/bckhcGMXQDujIXpbCDRb2Q
  Content-Type: application/json;charset=UTF-8

  {
    "href": "https://api.stormpath.com/v1/directories/2SKhstu8PlaekcaEXampLE",
    "name": "Captains",
    "description": "Captains from a variety of stories",
    "status": "ENABLED",
    "createdAt": "2015-08-24T15:32:23.079Z",
    "modifiedAt": "2015-08-24T15:32:23.079Z",
    "tenant": {
      "href": "https://api.stormpath.com/v1/tenants/1gBTncWsp2ObQGeXampLE"
    },
    "provider": {
      "href": "https://api.stormpath.com/v1/directories/2SKhstu8PlaekcaEXampLE/provider"
    },
    "comment":" // This JSON has been truncated for readability",
    "groups": {
      "href": "https://api.stormpath.com/v1/directories/2SKhstu8PlaekcaEXampLE/groups"
    }
  }

.. _about-mirror-dir:

Mirror Directories
^^^^^^^^^^^^^^^^^^ 

Mirror Directories are a big benefit to Stormpath customers who need LDAP directory accounts to be able to securely log in to public web applications without breaking corporate firewall policies. Here is how they work:

- After creating an LDAP Directory in Stormpath, you download a Stormpath Agent. This is a simple standalone software application that you install behind the corporate firewall so it can communicate directly with the LDAP server.
- You configure the agent via LDAP filters to view only the accounts that you want to expose to your Stormpath-enabled applications.
- The Agent will start synchronizing immediately, pushing this select data outbound to Stormpath over a TLS (HTTPS) connection.
- The synchronized user Accounts and Groups appear in the Stormpath Directory. The Accounts will be able to log in to any Stormpath-enabled application that you assign.
- When the Agent detects local LDAP changes, additions or deletions to these specific Accounts or Groups, it will automatically propagate those changes to Stormpath to be reflected by your Stormpath-enabled applications.
  
User Accounts and Groups in mirrored directories are automatically deleted when any of the following things happen:

- The original object is deleted from the LDAP directory service.
- The original LDAP object information no longer matches the account filter criteria configured for the agent.
- The LDAP directory is deleted.

The big benefit is that your Stormpath-enabled applications still use the same convenient REST+JSON API – they do not need to know anything about things like LDAP or legacy connection protocols.

.. _modeling-mirror-dirs:

Modeling Mirror Directories
"""""""""""""""""""""""""""

If you Application is going to have an LDAP integration, it will need to support multiple Directories — one Mirror Directory for each LDAP integration.

In this scenario, we recommend linking each Account in a LDAP Mirror Directory with a master Account in a master Directory. This offers a few benefits:

1. You can maintain one Directory that has all your user Accounts, retaining globally unique canonical identities across your application

2. You are able to leverage your own Groups in the master Directory. Remember, most data in a Mirror Directory is read-only, meaning you cannot create your own Groups in it, only read the Groups synchronized from Active Directory and LDAP

3. Keep a user’s identity alive even after they've left your customer's organization and been deprovisioned in AD/LDAP. This is valuable in a SaaS model where the user is loosely coupled to an organization. Contractors and temporary workers are good examples

The Stormpath Agent (see :ref:`ref-ldap-agent`) is regularly updating its Mirror Directory and sometimes adding new user Accounts. Because this data can be quite fluid, we recommend initiating all provisioning, linking, and synchronization on a successful login attempt of the Account in the Mirror Directory. This means that the master Directory would start off empty, and would then gradually become populated every time a user logged in.

For more information on how to this works, please see :ref:`mirror-dir-authn`.

.. _make-mirror-dir:

How to Make a Mirror Directory
""""""""""""""""""""""""""""""

Presently, Mirror Directories can be made via the Stormpath Admin Console, or using the REST API. If you'd like to do it with the Admin Console, please see `the Directory Creation section of the Admin Console Guide <http://docs.stormpath.com/console/product-guide/#create-a-directory>`_. For more information about creating them using REST API, please see :ref:`mirror-dir-authn`. 

.. _about-social-dir:
    
Social Directories
^^^^^^^^^^^^^^^^^^

Stormpath works with user Accounts pulled from social login providers (currently Google, Facebook, Github, and LinkedIn) in a way very similar to the way it works with user Accounts from LDAP servers. These external Identity Providers (IdPs) are modeled as Stormpath Directories, much like Mirror Directories. The difference is that, while Mirror Directories always come with an Agent that takes care of synchronization, Social Directories have an associated **Provider** resource. This resource contains the information required by the social login site to work with their site (e.g. the App ID for your Google application or the App Secret).

Stormpath also simplifies the authorization process by doing things like automating Google's access token exchange flow. All you do is POST the authorization code from the end-user and Stormpath returns a new or updated user Account, along with the Google access token which you can use for any further API calls. 

Modeling Social Directories
"""""""""""""""""""""""""""

Modeling your users who authorize via Social Login could be accomplished by creating a Directory resource for each social provider that you want to support, along with one master Directory for your application. For more about how these Directories are provisioned, please see :ref:`non-cloud-login`.

.. note::

  For both Mirror and Social Directories, since the relationship with the outside directory is read-only, the remote directory is still the "system of record".

How to Make a Social Directory
""""""""""""""""""""""""""""""

Presently, Social Directories can be made via the Stormpath Admin Console or using REST API. For more information about creating them with the Admin Console please see the `Directories section of the Stormpath Admin Console Guide <http://docs.stormpath.com/console/product-guide/#create-a-directory>`_. For more information about creating them using REST API, please see :ref:`social-authn`. 

.. _group-mgmt:

ii. Groups
----------

The Group resource can either be imagined as a container for Accounts, or as a label applied to them. Groups can be used in a variety of ways, including organizing people by geographic location, or by the their role within a company. 

For more detailed information about the Group resource, please see the :ref:`ref-group` section of the Reference chapter. 

.. _hierarchy-groups:

Modeling User Hierarchies Using Groups
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Groups, like labels, are inherently "flat". This means that they do not by default include any kind of hierarchy. If a hierarchical or nested structure is desired, it can be simulated in one of two ways: Either, using the Group resource's ``description`` field, or with the Group's associated customData resource. 

A geographical region can, for example, be represented as ``"North America/US/US East"`` in the Group's ``description`` field, allowing for queries to be made using simple pattern-matching queries. So to find all Groups in the US, you'd make the following HTTP GET::

  https://api.stormpath.com/v1/directories/$DIR_ID/groups?description=US*

Or, to find all Groups in the US East region only, you would GET::

  https://api.stormpath.com/v1/directories/$DIR_ID/groups?description=US%20East*

.. note::

  URL encoding will change a space into "%20".

It can also be included in the customData resource, as a series of key-value relations. The downside to this second approach is that customData resources are not currently searchable in the same manner as the Group's ``description`` field is.

How to Create a Group
^^^^^^^^^^^^^^^^^^^^^

So let's say we want to add a new Group resource with the name "Starfleet Officers" to the "Captains" Directory. 

The following API request:

.. code-block:: http    

  POST /v1/directories/2SKhstu8PlaekcaEXampLE/groups HTTP/1.1
  Host: api.stormpath.com
  Content-Type: application/json;charset=UTF-8

  {
    "name" : "Starfleet Officers",
    "description" : "Commissioned officers in Starfleet",
    "status" : "enabled"
  }

Would yield this response:

.. code-block:: http 

  HTTP/1.1 201 Created
  Location: https://api.stormpath.com/v1/groups/1ORBsz2iCNpV8yJExAMpLe
  Content-Type: application/json;charset=UTF-8
  
  {
    "href":"https://api.stormpath.com/v1/groups/1ORBsz2iCNpV8yJExAMpLe",
    "name":"Starfleet Officers",
    "description":"Commissioned officers in Starfleet",
    "status":"ENABLED",
    "createdAt":"2015-08-25T20:09:23.698Z",
    "modifiedAt":"2015-08-25T20:09:23.698Z",
    "customData":{
      "href":"https://api.stormpath.com/v1/groups/1ORBsz2iCNpV8yJExAMpLe/customData"
    },
    "directory":{
      "href":"https://api.stormpath.com/v1/directories/2SKhstu8PlaekcaEXampLE"
    },
    "tenant":{
      "href":"https://api.stormpath.com/v1/tenants/1gBTncWsp2ObQGeXampLE"
    },
    "accounts":{
      "href":"https://api.stormpath.com/v1/groups/1ORBsz2iCNpV8yJExAMpLe/accounts"
    },
    "accountMemberships":{
      "href":"https://api.stormpath.com/v1/groups/1ORBsz2iCNpV8yJExAMpLe/accountMemberships"
    },
    "applications":{
      "href":"https://api.stormpath.com/v1/groups/1ORBsz2iCNpV8yJExAMpLe/applications"
    }
  }

.. _account-creation:

b. How to Store Accounts in Stormpath
=====================================

The Account resource is a unique identity within your application. It is usually used to model an end-user, although it can also be used by a service, process, or any other entity that needs to log-in to Stormpath.

For more detailed information about the Account resource, see the :ref:`ref-account` section of the Reference chapter.  

New Account Creation
--------------------

The basic steps for creating a new Account are covered in the :ref:`Quickstart <quickstart>` chapter. In that example, we show how to add an Account to an Application. Below, we will also show how to add an Account to a specific Directory or Group. 

.. _add-new-account:

Add a New Account to a Directory
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Because Accounts are "owned" by Directories, you create new Accounts by adding them to a Directory. You can add an Account to a Directory directly, or you can add it indirectly by registering an Account with an Application, like in the :doc:`Quickstart </003_quickstart>`. 

.. note::

  This section will show examples using a Directory's ``/accounts`` href, but they will also function the same if you use an Application’s ``/accounts`` href instead.

Let's say we want to add a new account for user "Jean-Luc Picard" to the "Captains" Directory, which has the ``directoryId`` value ``2SKhstu8PlaekcaEXampLE``. The following API request:

.. code-block:: http 

  POST /v1/directories/2SKhstu8PlaekcaEXampLE/accounts HTTP/1.1
  Host: api.stormpath.com
  Content-Type: application/json;charset=UTF-8

  {
    "username" : "jlpicard",
    "email" : "capt@enterprise.com",
    "givenName" : "Jean-Luc",
    "surname" : "Picard",
    "password" : "uGhd%a8Kl!"
  }

.. note::

  The password in the request is being sent to Stormpath as plain text. This is one of the reasons why Stormpath only allows requests via HTTPS. Stormpath implements the latest password hashing and cryptographic best-practices that are automatically upgraded over time so the developer does not have to worry about this. Stormpath can only do this for the developer if we receive the password as plaintext, and only hash it using these techniques.

  Plaintext passwords also allow Stormpath to enforce password restrictions in a configurable manner. 

  Most importantly, Stormpath never persists or relays plaintext passwords under any circumstances.

  On the client side, then, you do not need to worry about salting or storing passwords at any point; you need only pass them to Stormpath for hashing, salting, and persisting with the appropriate HTTPS API call.

Would yield this response:

.. code-block:: http 

  HTTP/1.1 201 Created
  Location: https://api.stormpath.com/v1/accounts/3apenYvL0Z9v9spExAMpLe
  Content-Type: application/json;charset=UTF-8

  {
    "href": "https://api.stormpath.com/v1/accounts/3apenYvL0Z9v9spExAMpLe",
    "username": "jlpicard",
    "email": "capt@enterprise.com",
    "givenName": "Jean-Luc",
    "middleName": null,
    "surname": "Picard",
    "fullName": "Jean-Luc Picard",
    "status": "ENABLED",
    "createdAt": "2015-08-25T19:57:05.976Z",
    "modifiedAt": "2015-08-25T19:57:05.976Z",
    "emailVerificationToken": null,
    "customData": {
      "href": "https://api.stormpath.com/v1/accounts/3apenYvL0Z9v9spExAMpLe/customData"
    },
    "providerData": {
      "href": "https://api.stormpath.com/v1/accounts/3apenYvL0Z9v9spExAMpLe/providerData"
    },
    "comment":" // This JSON has been truncated for readability"
  }

Add an Existing Account to a Group
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
      
So let's say we want to add "Jean-Luc Picard" to the "Starfleet Officers" Group inside the "Captains" Directory.

We make the following request:

.. code-block:: http 

  POST /v1/groupMemberships HTTP/1.1
  Host: api.stormpath.com
  Content-Type: application/json;charset=UTF-8
  
  {
    "account" : {
        "href" : "https://api.stormpath.com/v1/accounts/3apenYvL0Z9v9spExAMpLe"
     },
     "group" : {
         "href" : "https://api.stormpath.com/v1/groups/1ORBsz2iCNpV8yJExAMpLe"
     }
  }

And get the following response:

.. code-block:: http

  HTTP/1.1 201 Created
  Location: https://api.stormpath.com/v1/groupMemberships/1ufdzvjTWThoqnHf0a9vQ0
  Content-Type: application/json;charset=UTF-8

  {
    "href": "https://api.stormpath.com/v1/groupMemberships/1ufdzvjTWThoqnHf0a9vQ0",
    "account": {
      "href": "https://api.stormpath.com/v1/accounts/3apenYvL0Z9v9spExAMpLe"
    },
    "group": {
      "href": "https://api.stormpath.com/v1/groups/1ORBsz2iCNpV8yJExAMpLe"
    }
  }

.. _importing-accounts:

Importing Accounts
------------------

Stormpath also makes it very easy to transfer your existing user directory into a Stormpath Directory using our API. Depending on how you store your passwords, you will use one of three approaches:

1. **Passwords in Plaintext:** If you stored passwords in plaintext, you can use the Stormpath API to import them directly. Stormpath will create the Accounts and secure their passwords automatically (within our system). Make sure that your Stormpath Directory is configured to *not* send Account Verification emails before beginning import.
2. **Passwords With MCF Hash:** If your password hashing algorithm follows a format Stormpath supports, you can use the API to import Accounts directly. Available formats and instructions are detailed :ref:`below <importing-mcf>`.
3. **Passwords With Non-MCF Hash:** If you hashed passwords in a format Stormpath does not support, you can still use the API to create the Accounts, but you will need to issue a password reset afterwards. Otherwise, your users won't be able to use their passwords to login.

.. note::

  To import user accounts from an LDAP or Social Directory, please see :ref:`non-cloud-login`.

Due to the sheer number of database types and the variation between individual data models, the actual importing of users is not something that Stormpath handles at this time. What we recommend is that you write a script that is able to iterate through your database and grab the necessary information. Then the script uses our APIs to re-create the user base in the Stormpath database. 
   
Importing Accounts with Plaintext Passwords
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In this case, it is recommended that you suppress Account Verification emails. This can be done by simply adding a ``registrationWorkflowEnabled=false`` query parameter to the end of your API like so::

  https://api.stormpath.com/v1/directories/WpM9nyZ2TbaEzfbeXaMPLE/accounts?registrationWorkflowEnabled=false

.. _importing-mcf:

Importing Accounts with MCF Hash Passwords
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you are moving from an existing user repository to Stormpath, you may have existing password hashes that you want to reuse in order to provide a seamless upgrade path for your end users. Stormpath does not allow for Account creation with *any* password hash, the password hash must follow modular crypt format (MCF), which is a ``$`` delimited string. 
This works as follows:

1. Create the Account specifying the password hash instead of a plain text password. Stormpath will use the password hash to authenticate the Account’s login attempt.

2. If the login attempt is successful, Stormpath will recreate the password hash using a secure HMAC algorithm.
   
Supported Hashing Algorithms
""""""""""""""""""""""""""""

Stormpath only supports password hashes that use the following algorithms:

- **bcrypt**: These password hashes have the identifier ``$2a$``, ``$2b$``, ``$2x$``, ``$2a$``
- **stormpath2**: A Stormpath-specific password hash format that can be generated with common password hash information, such as algorithm, iterations, salt, and the derived cryptographic hash. For more information see :ref:`below <stormpath2-hash>`.
  
Once you have a bcrypt or stormpath2 MCF password hash, you can create the Account in Stormpath with the password hash by POSTing the Account information to the Directory or Application ``/accounts`` endpoint and specifying ``passwordFormat=mcf`` as a query parameter::

  https://api.stormpath.com/v1/directories/WpM9nyZ2TbaEzfbeXaMPLE/accounts?passwordFormat=mcf

.. _stormpath2-hash:

The stormpath2 Hashing Algorithm
++++++++++++++++++++++++++++++++

stormpath2 has a format which allows you to derive an MCF hash that Stormpath can read to understand how to recreate the password hash to use during a login attempt. stormpath2 hash format is formatted as::

  $stormpath2$ALGORITHM_NAME$ITERATION_COUNT$BASE64_SALT$BASE64_PASSWORD_HASH

.. list-table:: 
  :widths: 20 20 20 
  :header-rows: 1

  * - Property
    - Description
    - Valid Values
  
  * - ``ALGORITHM_NAME``
    - The name of the hashing algorithm used to generate the ``BASE64_PASSWORD_HASH``.
    - ``MD5``, ``SHA-1``, ``SHA-256``, ``SHA-384``, ``SHA-512``
  
  * - ``ITERATION_COUNT``
    - The number of iterations executed when generating the ``BASE64_PASSWORD_HASH``
    - Number > 0
  
  * - ``BASE64_SALT``
    - The salt byte array used to salt the first hash iteration.
    - String (Base64). If your password hashes do you have salt, you can leave it out entirely. 

  * - ``BASE64_PASSWORD_HASH``
    - The computed hash byte array.
    - String (Base64)


Importing Accounts with Non-MCF Hash Passwords
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In this case you will be using the API in the same way as usual, except with the Password Reset Workflow enabled. That is, you should set the Account's password to a large randomly generated string, and then force the user through the password reset flow. For more information, please see the :ref:`Password Reset section below <password-reset-flow>`.

.. _add-user-customdata:

How to Store Additional User Information as Custom Data
-------------------------------------------------------

While Stormpath’s default Account attributes are useful to many applications, you might want to add your own custom data to a Stormpath Account. If you want, you can store all of your custom account information in Stormpath so you don’t have to maintain another separate database to store your specific account data.

One example of this could be if we wanted to add information to our "Jean-Luc Picard" Account that didn't fit into any of the existing Account attributes.

For example, we could want to add information about this user's current location, like the ship this Captain is currently assigned to. To do this, we specify the ``accountId`` and the ``/customdata`` endpoint. 

So if we were to send following REST call:

.. code-block:: http

  POST /v1/accounts/3apenYvL0Z9v9spExAMpLe/customData HTTP/1.1
  Host: api.stormpath.com
  Content-Type: application/json;charset=UTF-8

  {
    "currentAssignment": "USS Enterprise (NCC-1701-E)"
  }

We would get this response:

.. code-block:: http  

  HTTP/1.1 201 Created
  Location: https://api.stormpath.com/v1/accounts/3apenYvL0Z9v9spExAMpLe/customData
  Content-Type: application/json;charset=UTF-8

  {
    "href": "https://api.stormpath.com/v1/accounts/3apenYvL0Z9v9spExAMpLe/customData",
    "createdAt": "2015-08-25T19:57:05.976Z",
    "modifiedAt": "2015-08-26T19:25:27.936Z",
    "currentAssignment": "USS Enterprise (NCC-1701-E)"
  }

This information can also be appended as part of the initial Account creation payload. 

For more information about the customData resource, please see the `customData section <http://docs.stormpath.com/rest/product-guide/#custom-data>`_ of the REST API Product Guide .

c. How to Search Accounts
=========================

You can search Stormpath Accounts, just like all Resource collections, using Filter, Attribute, and Datetime search. For more information about how search works in Stormpath, please see the :ref:`Search section <about-search>` of the Reference chapter.

Search can be performed against one of the collections of Accounts associated with other entities:

``/v1/applications/$APPLICATION_ID/accounts``

``/v1/directories/$DIRECTORY_ID/accounts``

``/v1/groups/$GROUP_ID/accounts``

``/v1/organizations/$ORGANIZATION_ID/accounts``

As mentioned in the :ref:`Search section <about-search>` of the Reference chapter, the Account resource's **searchable attributes** are: 

- ``givenName``
- ``middleName``
- ``surname``
- ``username``
- ``email``
- ``status``

Example Account Searches
------------------------

Below are some examples of different kinds of searches that can be performed to find Accounts.

Search an Application's Accounts for a Particular Word 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A simple :ref:`search-filter` using the ``?q=`` parameter to the Application's ``/accounts`` collection will find us any Account associated with this Application that has the filter query string as part of any of its searchable attributes. 

**Query**

.. code-block:: http 

  GET /v1/application/1gk4Dxzi6o4Pbdlexample/accounts?q=luc HTTP/1.1
  Host: api.stormpath.com
  Content-Type: application/json;charset=UTF-8

.. note::

  Matching is case-insensitive. So ``?q=luc`` and ``?q=Luc`` will return the same results.

**Response**

.. code-block:: http  

  HTTP/1.1 200 OK
  Location: https://api.stormpath.com/v1/applications/1gk4Dxzi6o4Pbdlexample/accounts
  Content-Type: application/json;charset=UTF-8

  {
    "href": "https://api.stormpath.com/v1/applications/1gk4Dxzi6o4Pbdlexample/accounts",
    "offset": 0,
    "limit": 25,
    "size": 1,
    "items": [
        {
            "href": "https://api.stormpath.com/v1/accounts/3apenYvL0Z9v9spexAmple",
            "username": "jlpicard",
            "email": "capt@enterprise.com",
            "givenName": "Jean-Luc",
            "middleName": null,
            "surname": "Picard",
            "fullName": "Jean-Luc Picard",
            "status": "ENABLED",
            "...": "..."
        }
    ]
  }

Find All the Disabled Accounts in a Directory
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An :ref:`search-attribute` can be used on a Directory's Accounts collection in order to find all of the Accounts that contain a certain value in the specified attribute. This could be used to find all the Accounts that are disabled (i.e. that have their ``status`` set to ``disabled``). 

**Query**

.. code-block:: http 

  GET /v1/directories/accounts?status=DISABLED HTTP/1.1
  Host: api.stormpath.com
  Content-Type: application/json;charset=UTF-8

**Response**

.. code-block:: http  

  HTTP/1.1 200 OK
  Location: https://api.stormpath.com/v1/
  Content-Type: application/json;charset=UTF-8

  {
      "href": "https://api.stormpath.com/v1/directories/2SKhstu8PlaekcaEXampLE/accounts",
      "offset": 0,
      "limit": 25,
      "size": 1,
      "items": [
          {
              "href": "https://api.stormpath.com/v1/accounts/72EaYgOaq8lwTFHexAmple",
              "username": "first2shoot",
              "email": "han@newrepublic.gov",
              "givenName": "Han",
              "middleName": null,
              "surname": "Solo",
              "fullName": "Han Solo",
              "status": "DISABLED",
              "...": "..."
          }
      ]
  }

Find All Accounts in a Directory That Were Created on a Particular Day 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:ref:`search-datetime` is used when you want to search for Accounts that have a certain point or period in time that interests you. So we could search for all of the Accounts in a Directory that were modified on Dec 1, 2015.

**Query**

.. code-block:: http 

  GET /v1/directories/2SKhstu8PlaekcaEXampLE/accounts?modifiedAt=2015-12-01 HTTP/1.1
  Host: api.stormpath.com
  Content-Type: application/json;charset=UTF-8

.. note::

  The parameter can be written in many different ways. The following are all equivalent:

  - ?modifiedAt=2015-12-01
  - ?modifiedAt=[2015-12-01T00:00, 2015-12-02T00:00]
  - ?modifiedAt=[2015-12-01T00:00:00, 2015-12-02T00:00:00]

  For more information see :ref:`search-datetime`.

**Response**

.. code-block:: http  

  HTTP/1.1 200 OK
  Location: https://api.stormpath.com/v1/
  Content-Type: application/json;charset=UTF-8

  {
      "href": "https://api.stormpath.com/v1/directories/2SKhstu8Plaekcai8lghrp/accounts",
      "offset": 0,
      "limit": 25,
      "size": 1,
      "items": [
          {
              "href": "https://api.stormpath.com/v1/accounts/72EaYgOaq8lwTFHILydAid",
              "username": "first2shoot",
              "email": "han@newrepublic.gov",
              "givenName": "Han",
              "middleName": null,
              "surname": "Solo",
              "fullName": "Han Solo",
              "status": "DISABLED",
              "createdAt": "2015-08-28T16:07:38.347Z",
              "modifiedAt": "2015-12-01T21:22:56.608Z",
              "...": "..."
          }
      ]
  }

.. _managing-account-pwd:

d. How to Manage an Account's Password
======================================

Manage Password Policies
------------------------

In Stormpath, password policies are defined on a Directory level. Specifically, they are controlled in a **Password Policy** resource associated with the Directory. Modifying this resource also modifies the behavior of all Accounts that are included in this Directory. For more information about this resource, see the :ref:`Password Policy section in the Reference chapter <ref-password-policy>`.

.. note::

  This section assumes a basic familiarity with Stormpath Workflows. For more information about Workflows, please see `the Directory Workflows section of the Admin Console Guide <http://docs.stormpath.com/console/product-guide/#directory-workflows>`_. 

Changing the Password Strength resource for a Directory modifies the requirement for new Accounts and password changes on existing Accounts in that Directory. To update Password Strength, simply HTTP POST to the appropriate ``$directoryId`` and ``/strength`` resource with the changes.

This call:

.. code-block:: http

  POST v1/passwordPolicies/$DIRECTORY_ID/strength HTTP/1.1
  Host: api.stormpath.com
  Content-Type: application/json;charset=UTF-8

  {
    "minLength": 1,
    "maxLength": 24,
    "minSymbol": 1
  }

would result in the following response:

.. code-block:: http

  HTTP/1.1 200 OK
  Location: https://api.stormpath.com/v1/passwordPolicies/$DIRECTORY_ID/strength
  Content-Type: application/json;charset=UTF-8

  {
    "href": "https://api.stormpath.com/v1/passwordPolicies/$DIRECTORY_ID/strength", 
    "maxLength": 24, 
    "minDiacritic": 0, 
    "minLength": 1, 
    "minLowerCase": 1, 
    "minNumeric": 1, 
    "minSymbol": 1, 
    "minUpperCase": 1
  }

.. _change-account-pwd:

Change An Account's Password
----------------------------

At no point is the user shown, or does Stormpath have access to, the original password once it has been hashed during account creation. The only ways to change an Account password once it has been created are: 

1. To allow the user to update it (without seeing the original value) after being authenticated, or
2. To use the :ref:`password reset workflow <password-reset-flow>`.

To update the password, you simply send a POST to the ``v1/accounts/$ACCOUNT_ID`` endpoint with the new password:

.. code-block:: http 

  POST /v1/accounts/3apenYvL0Z9v9spexAmple HTTP/1.1
  Host: api.stormpath.com
  Content-Type: application/json

  {
    "password":"some_New+Value1234"
  }

If the call succeeds you will get back an ``HTTP 200 OK`` with the Account resource in the body. 

For more information about resetting the password, read on.

.. _password-reset-flow:

Password Reset
--------------

Password Reset in Stormpath is a self-service flow, where the user is sent an email with a secure link. The user can then click that link and be shown a password reset form. The password reset workflow involves changes to an account at an application level, and as such, this workflow relies on the application resource as a starting point. While this workflow is disabled by default, you can enable it easily in the Stormpath Admin Console UI. Refer to the `Stormpath Admin Console product guide <http://docs.stormpath.com/console/product-guide/#password-reset>`__ for complete instructions.

How To Reset A Password 
-----------------------

There are three steps to the password reset flow:

1. Trigger the workflow 
2. Verify the token
3. Update the password
   
**Trigger the workflow** 

To trigger the password reset workflow, you send an HTTP POST to the Application's ``/passwordResetTokens`` endpoint: 

.. code-block:: http 

  POST /v1/applications/1gk4Dxzi6o4Pbdlexample/passwordResetTokens HTTP/1.1
  Host: api.stormpath.com
  Content-Type: application/json

  {
    "email":"phasma@empire.gov"
  }

.. note::

  It is also possible to specify the Account Store in your Password Reset POST:

  .. code-block:: http

    POST /v1/applications/1gk4Dxzi6o4Pbdlexample/passwordResetTokens HTTP/1.1
    Host: api.stormpath.com
    Content-Type: application/json

    {
      "email":"phasma@empire.gov"
      "accountStore": {
        "href": "https://api.stormpath.com/v1/groups/2SKhstu8Plaekcai8lghrp"
      }
    }


If this is a valid email in an Account associated with this Application, you will get a success response:

.. code-block:: http

  HTTP/1.1 200 OK
  Content-Type: application/json

  {
    "href": "https://api.stormpath.com/v1/applications/1gk4Dxzi6o4PbdlBVa6tfR/passwordResetTokens/eyJraWQiOiIxZ0JUbmNXc3AyT2JRR2dEbjlSOTFSIiwiYWxnIjoiSFExaMPLe.eyJleHAiOjE0NDgwNDg4NDcsImp0aSI6IjJwSW44eFBHeURMTVM5WFpqWEVExaMPLe.cn9VYU3OnyKXN0dA0qskMv4T4jhDgQaRdA-wExaMPLe",
    "email": "phasma@empire.gov",
    "account": {
        "href": "https://api.stormpath.com/v1/accounts/2FvPkChR78oFnyfexample"
    }
  }

.. note::

  For a full description of this endpoint please see :ref:`ref-password-reset-token` in the Reference chapter. 

At this point, an email will be built using the password reset base URL specified in the Stormpath Admin Console. Stormpath sends an email (that you :ref:`can customize <password-reset-email-templates>`) to the user with a link in the format that follows:

``http://yoursite.com/path/to/reset/page?sptoken=$TOKEN``

So the user would then receive something that looked like this::

  Forgot your password? 

  We've received a request to reset the password for this email address. 

  To reset your password please click on this link or cut and paste this
  URL into your browser (link expires in 24 hours): 
  https://api.stormpath.com/passwordReset?sptoken=eyJraWQiOiIxZ0JUbmNXc[...]

  This link takes you to a secure page where you can change your password.

**Verify the token**

Once the user clicks this link, your controller should retrieve the token from the query string and check it against the Stormpath API. This can be accomplish by sending a GET to the Application's ``/passwordResetTokens/$TOKEN_VALUE`` endpoint:

.. code-block:: http 

  GET /v1/applications/1gk4Dxzi6o4Pbdlexample/passwordResetTokens/eyJraWQiOiIxZ0JUbmNXc[...] HTTP/1.1
  Host: api.stormpath.com
  Content-Type: application/json;charset=UTF-8

This would result in the exact same ``HTTP 200`` success response as when the token was first generated above.

**Update the password**

After a successful GET with the query string token, you can direct the user to a page where they can update their password. Once you have the password, you can update the Account resource with POST to the  `passwordResetTokens` endpoint. This is the same endpoint that you used to validate the token above.

.. code-block:: http 

  POST /v1/applications/1gk4Dxzi6o4Pbdlexample/passwordResetTokens/eyJraWQiOiIxZ0JUbmNXc[...] HTTP/1.1
  Host: api.stormpath.com
  Content-Type: application/json;charset=UTF-8

  {
    "password": "updated+Password1234"
  }

On success, the response will include a link to the Account that the password was reset for. It will also send the password change confirmation email that was configured in the Administrator Console to the email account associated with the account.

Manage Password Reset Emails 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Password Reset Email is configurable for a Directory. There is a set of properties on the :ref:`ref-password-policy` resource that define its behavior. These properties are:

- ``resetEmailStatus`` which enables or disables the reset email.
- ``resetEmailTemplates`` which defines the content of the password reset email that is sent to the Account’s email address with a link to reset the Account’s password. 
- ``resetSuccessEmailStatus`` which enables or disables the reset success email, and
- ``resetSuccessEmailTemplates`` which defines the content of the reset success email.

To control whether any email is sent or not is simply a matter of setting the appropriate value to either ``ENABLED`` or ``DISABLED``. For example, if you would like a Password Reset email to be sent, send the following:

.. code-block:: http 

  POST /v1/passwordPolicies/$DIRECTORY_ID HTTP/1.1
  Host: api.stormpath.com
  Content-Type: application/json;charset=UTF-8

  {
      "resetEmailStatus": "ENABLED"
  }

.. _password-reset-email-templates:

Password Reset Email Templates
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The contents of the password reset and the password reset success emails are both defined in an :ref:`ref-emailtemplates` collection. 

To modify the emails that get sent during the password reset workflow, all you have to do is send an HTTP POST with the desired property in the payload body.

.. _verify-account-email:

e. How to Verify an Account's Email 
===================================

If you want to verify that an Account’s email address is valid and that the Account belongs to a real person, Stormpath can help automate this for you using `Workflows <http://docs.stormpath.com/console/product-guide/#directory-workflows>`_.

Understanding the Email Verification Workflow
---------------------------------------------

This workflow involves 3 parties: your application's end-user, your application, and the Stormpath API server.

1. When the Account is created in a Directory that has “Verification” enabled, Stormpath will automatically send an email to the Account's email address.
2. The end-user opens their email and clicks the verification link. This link comes with a token.
3. With the token, your application calls back to the Stormpath API server to complete the process.

If you create a new Account in a Directory with both Account Registration and Verification enabled, Stormpath will automatically send a welcome email that contains a verification link to the Account’s email address on your behalf. If the person reading the email clicks the verification link in the email, the Account will then have an ``ENABLED`` status and be allowed to log in to applications.

.. note::

  Accounts created in a Directory that has the Verification workflow enabled will have an ``UNVERIFIED`` status by default. ``UNVERIFIED`` is the same as ``DISABLED``, but additionally indicates why the Account is disabled. When the email link is clicked, the Account's status will change ``ENABLED``.


The Account Verification Base URL 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is also expected that the workflow’s **Account Verification Base URL** has been set to a URL that will be processed by your own application web server. This URL should be free of any query parameters, as the Stormpath back-end will append on to the URL a parameter used to verify the email. If this URL is not set, a default Stormpath-branded page will appear which allows the user to complete the workflow.

.. note::

  The Account Verification Base URL defaults to a Stormpath API Sever URL which, while it is functional, is a Stormpath API server web page. Because it will likely confuse your application end-users if they see a Stormpath web page, we strongly recommended that you specify a URL that points to your web application.

Configuring the Verification Workflow
-------------------------------------

This workflow is disabled by default on Directories, but you can enable it, and set up the account verification base URL, easily in the Stormpath Admin Console UI. Refer to the `Stormpath Admin Console Guide <http://docs.stormpath.com/console/product-guide/#directory-workflows>`_ for complete instructions.

Triggering the Verification Email (Creating A Token)
----------------------------------------------------

In order to verify an Account’s email address, an ``emailVerificationToken`` must be created for that Account. To create this token, you simply create an Account in a Directory, either programmatically or via a public account creation form of your own design, that has the account registration and verification workflows enabled.

Verifying the Email Address (Consuming The Token)
-------------------------------------------------

The email that is sent upon Account creation contains a link to the base URL that you've configured, along with the ``sptoken`` query string parameter::

  http://www.yourapplicationurl.com/path/to/validator/?sptoken=$VERIFICATION_TOKEN

The token you capture from the query string is used to form the full ``href`` for a special email verification endpoint used to verify the Account::

  /v1/accounts/emailVerificationsToken/$VERIFICATION_TOKEN

To verify the Account, you use the token from the query string to form the above URL and POST a body-less request against the fully-qualified end point:

.. code-block:: http 

  POST /v1/accounts/emailVerificationTokens/6YJv9XBH1dZGP5A8rq7Zyl HTTP/1.1
  Host: api.stormpath.com
  Content-Type: application/json;charset=UTF-8

Which will return a result that looks like this:

.. code-block:: http 

  HTTP/1.1 200 OK
  Location: https://api.stormpath.com/v1/accounts/6XLbNaUsKm3E0kXMTTr10V
  Content-Type: application/json;charset=UTF-8;

  {
    "href": "https://api.stormpath.com/v1/accounts/6XLbNaUsKm3E0kXMTTr10V"
  }

If the validation succeeds, you will receive back the ``href`` for the Account resource which has now been verified. An email confirming the verification will be automatically sent to the Account’s email address by Stormpath afterwards, and the Account will then be able to authenticate successfully.

If the verification token is not found, a ``404 Not Found`` error is returned with a payload explaining why the attempt failed.

.. note::

  For more about Account Authentication you can read :doc:`the next chapter </005_auth_n>`.

.. _resending-verification-email:

Resending The Verification Email 
--------------------------------

If a user accidentally deletes their verification email, or it was undeliverable for some reason, it is possible to resend the email using the :ref:`Application resource's <ref-application>` ``/verificationEmails`` endpoint. 

.. code-block:: http 

  POST /v1/applications/$APPLICATION_ID/verificationEmails HTTP/1.1
  Host: api.stormpath.com
  Content-Type: application/json;charset=UTF-8

  {
    "login": "email@address.com"
  }

If this calls succeeds, an ``HTTP 202 ACCEPTED`` will return. 

.. _customizing-email-templates:

f. Customizing Stormpath Emails via REST
========================================

What emails does Stormpath send?
--------------------------------

Stormpath can be configured to send emails to users as part of a Directory's Account Creation and Password Reset policies.

Account Creation
^^^^^^^^^^^^^^^^

Found in: :ref:`ref-accnt-creation-policy`

- *Verification Email*: The initial email that is sent out after Account creation that verifies the email address that was used for registration with a link containing the verification token.
- *Verification Success Email*: An email that is sent after a successful email verification.
- *Welcome Email*: An email welcoming the user to your application. 

For more information about this, see :ref:`verify-account-email`. 

Password Reset
^^^^^^^^^^^^^^

Found in: :ref:`ref-password-policy`

- *Reset Email*: The email that is sent out after a user asks to reset their password. It contains a URL with a password reset token.
- *Reset Success Email*:  An email that is sent after a successful password reset.

For more information about this, see :ref:`password-reset-flow`. 

Customizing Stormpath Email Templates 
-------------------------------------

The emails that Stormpath sends to users be customized by modifying the :ref:`ref-emailtemplates` resource. This can be done either via the "Directory Workflows" section of the `Stormpath Admin Console <https://api.stormpath.com/login>`__, or via REST. To find out how to do it via REST, keep reading. 

First, let's look at the default template that comes with the Stormpath Administrator's Directory:

.. code-block:: json 

  {
    "href":"https://api.stormpath.com/v1/emailTemplates/2jwPxFsnjqxYrojvU1m2Nh",
    "name":"Default Verification Email Template",
    "description":"This is the verification email template that is associated with the directory.",
    "fromName":"Jakub Swiatczak",
    "fromEmailAddress":"change-me@stormpath.com",
    "subject":"Verify your account",
    "textBody":"Hi,\nYou have been registered for an application that uses Stormpath.\n\n${url}\n\nOnce you verify, you will be able to login.\n\n---------------------\nFor general inquiries or to request support with your account, please email change-me@stormpath.com",
    "htmlBody":"<p>Hi,</p>\n<p>You have been registered for an application that uses Stormpath.</p><a href=\"${url}\">Click here to verify your account</a><p>Once you verify, you will be able to login.</p><p>--------------------- <br />For general inquiries or to request support with your account, please email change-me@stormpath.com</p>",
    "mimeType":"text/plain",
    "defaultModel":{
      "linkBaseUrl":"https://api.stormpath.com/emailVerificationTokens"
    }
  }

Message Format
--------------

The ``mimeType`` designates whether the email is sent as plain text or HTML. This in turns tells Stormpath whether to use the ``textBody`` or ``htmlBody`` text in the email. 

textBody and htmlBody
---------------------

These define the actual content of the email. The only difference is that ``htmlBody`` is allowed to contain HTML markup while ``textBody`` only accepts plaintext. Both are also able to use `Java Escape Sequences <http://web.cerritos.edu/jwilson/SitePages/java_language_resources/Java_Escape_Sequences.htm>`__. Both ``htmlBody`` and ``textBody`` can have customized output generated using template macros.

.. _using-email-macros:

Using Email Macros 
^^^^^^^^^^^^^^^^^^^^^^^^^

Stormpath uses Apache Velocity for email templating, and consequently you can use macros in your email templates. Macros are placeholder text that are converted into actual values at the time the email is generated. You could use a macro to insert your user's first name into the email, as well as the name of your Application. This would look like this: 

.. code-block:: java 

  "Hi ${account.givenName}, welcome to $!{application.name}!"

The basic structure for a macro is ``${resource.attribute}``. There are three kinds of ``resource`` that you can work with: 

- Account (``${account}``)
- an Account's Directory (``${account.directory}``), and 
- an Application (``$!{application}``). 
  
You can also include any ``attribute`` that isn't a link, as well as customData.

For a full list of email macros, see the :ref:`ref-email-macros` section of the Reference chapter. 

Macros and customData
"""""""""""""""""""""

The formatting for customData macros is as follows:

.. code-block:: velocity 

  $!{resource.attribute.customData.key}

You may have noticed here and with the Application resource that there is an included ``!`` character, this is called a "quiet reference". 

.. _quiet-macro-reference:

Quiet References
""""""""""""""""

Quiet references (``!``) tell Velocity that, if it can't resolve the object, it should just show nothing. Normally, if a macro was  ``Is your favorite color ${account.customData.favoriteColor}?``, and Velocity was able to find the value as ``blue``, it would output:

``Is your favorite color blue?``

However, if the value could not be found, it would output:

``Is your favorite color ${account.customData.favoriteColor}?``

To avoid this, we include the ``!`` which puts the macro into "quiet reference" mode. This means that if the value is not found, the output will be:

``Is your favorite color ?``

Since customData can contain any arbitrary key-value pairs, Stormpath recommends that any email macro references to customData keys use the ``!`` quiet reference. Applications should also use the quiet reference because there are possible cases where the Velocity engine might not have access to an Application resource. 