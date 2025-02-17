---
title: Store and Organize User Data
order: 4
---

## User profiles

A user profile is a collection of event data and descriptive attributes associated with a user. 

Specifically, profiles include data about all historical events triggered by a user in your app or website, a list of identifiers associated with a user (such as their email address or customer ID), and device information (e.g. iOS or Android, OS version number, device model).

Profiles are used to build and manage audiences (groups of users with shared attributes), generating calculated attributes, and enriching incoming data before forwarding it to any connected outputs. You can view specific user profiles in the User Activity View by searching for a user identifier.

All user profiles are uniquely identified with an mParticle ID: a 64-bit signed integer generated by mParticle for every profile created. Whenever IDSync receives a request to identify a user, it will return a single MPID that is used to ensure that incoming data is attributed to the correct user profile.

Learn more about profiles in User Profiles.

## User identifiers

User identifiers, referred to as `user identities` in the IDSync API, are attributes like email addresses or customer IDs defined as key/value pairs.

Identifiers are used to identify users whenever an IDSync request is received.

The complete list of supported identifiers are:

* `customer_id`
* `email`
* `facebook`
* `twitter`
* `google`
* `microsoft`
* `other`
* `other_id_2`
* `other_id_3`
* `other_id_4`
* `other_id_5`
* `other_id_6`
* `other_id_7`
* `other_id_8`
* `other_id_9`
* `other_id_10`
* `mobile_number`
* `phone_number_2`
* `phone_number_3`

There are several subcategories of identifiers with some unique characteristics, described below.

### Login IDs

Login IDs are used to identify one, and only one, MPID for a known user when resolving an identification request.

If your account uses the Profile Link strategy, then the first time IDSync receives an identification request containing a login ID with no associated user profiles a new user profile will be created.

To designate one or more identifiers as login IDs, contact your account representative directly or submit a request to mParticle support.

#### Using login IDs to protect known identity records

A login ID identifies a single known user. In order to maintain the integrity of known identity records, a record with at least one login ID can only be returned if the identify request includes a matching login ID.

#### Example

**Identity records**

| User Profile 1 | User Profile 2 | User Profile 3 |
| --- | --- | --- |
| MPID: `1234`<br>Customer ID: `h.jekyll.85`<br>Email: `ed.hyde@example.com`<br>IDFV: `1234` | MPID: `5678`<br>Email: `h.jekyll.md@example.com` | IDFV: `1234` |

**Scenarios**

| **Login Identity Setting** | **IDSync API Request** | **Results** |
| --- | --- | --- |
| Email & <br>Customer ID | Type: `Identify` <br>Email: `ed.hyde@example.com` | User Profile 1 has 2 login IDs, but we only need to match at least one to return the profile. **User Profile 1 is returned.** |
| Email | Type: `Identify` <br>Email: `h.jekyll.md@example.com`<br>IDFV: `5678` | The request matches the Login ID of User Profile 2. **User Profile 2 is returned.** |
| Email | Type: `Identify` <br>IDFV: `1234` | The IDFV matches User Profile 1, but since User Profile 1 includes a Login ID 'email', it cannot be returned to a request that doesn't also include the Login ID. mParticle does not have enough information to resolve to and return User Profile 1. **A new User Profile 3 is created.** |

#### Using login IDs to handle new known users

One way identity strategies handle new known users is by applying rules about what to do when a new login ID is received for the first time. 

For example, the [Profile link](/guides/idsync/profile-link-strategy/) strategy always creates a new identity record when a login ID is received for the first time. The [Profile conversion](/guides/idsync/profile-conversion-strategy/) strategy does not create a new identity record when a login ID is first received. The new ID is added to the existing identity record.

##### Example

**Identity records**

| User Profile 1 | User Profile 2 |
| --- | --- |
| MPID: `1234`<br>Customer ID: `h.jekyll.85`<br>Email: `ed.hyde@example.com`<br>IDFV: `1234` | MPID: `5678`<br>Email: `h.jekyll.md@example.com` |

**Scenarios**

| **Immutable Identity Setting** | **IDSync API Request** | **Results** |
| --- | --- | --- |
| Customer ID | Type: `Search` <br>Customer ID: `h.jekyll.85` | The request matches the Customer ID of User Profile 1. **User Profile 1 is returned.** |
| Customer ID | Type: `Search` <br>Email: `h.jekyll.md@example.com` | User Profile 2 does contain login ID with this email address, but the request does not match any known immutable IDs. **User Profile Not Found is returned.** |
| Customer ID | Type: `Search` <br>Customer ID: `9101` | Although this is the first time mParticle sees this potential login ID, **mParticle does not create a new User Profile** based on this email address, the request does not match any known Immutable IDs. **User Profile Not Found is returned.** |

### Immutable IDs

Immutable IDs are identifiers that cannot be changed once they have been set.

In order to maintain the integrity of known user profiles, the value of an immutable ID may not be modified to protect against identity theft. A profile with at least one immutable ID can only be returned if the identification request includes at least one matching immutable ID. 

<aside>
    To be set as an immutable ID, an identifier must also be set as a unique identifier and a login ID.
</aside>

Immutable IDs may be used as query parameters for the profile API.

### Unique IDs

A unique identity (unique ID) is a setting that specifies that that user profile identifier must be unique. This means that only one mParticle user profile can have that value of the identifier. 

If a modify request to the [IDSync API](https://docs.mparticle.com/developers/idsync/http-api/#identify) would result in two identity records sharing the same value of a unique identity, mParticle will add or update the identifier on the requested user profile and remove it from any other user profile to enforce uniqueness. *Note that this doesn't mean all other identifiers are removed from the user profile. The history of that profile remains intact. But removing the conflicting identifier from the profile means it can no longer be used to lookup that profile. User profiles with no remaining identifiers are effectively 'orphaned'. They will not be deleted, but can never be returned by an IDSync API request.*

#### Example

**User profiles**

A user signs up for your iOS mobile app with the email [ed.hyde@example.com](mailto:ed.hyde@example.com). The same person also independently interacts with your helpdesk, using a different email address [h.jekyll.md@example.com](mailto:h.jekyll.md@example.com). This results in two user profiles being created, one for each email. Each has a unique mParticle ID:

| User Profile 1 | User Profile 2 |
| --- | --- |
| MPID: `1234`<br>Customer ID: `h.jekyll.85`<br>Email: `ed.hyde@example.com`<br>IDFV: `1234` | MPID: `5678`<br>Email: `h.jekyll.md@example.com` |

**Scenarios**

| **Unique Identity Setting** | **IDSync API Request** | **Results** |
| --- | --- | --- |
| Email | Type: `Modify`<br>MPID: `1234`<br>Customer ID: `h.jekyll.85`<br>Email `h.jekyll.md@example.com`<br>IDFV: `1234` | The modify request **updates the email address of User Profile 1** to `h.jekyll.md@example.com`. Since emails must be unique, mParticle searches for other User Profiles with the same email address. **The duplicate email address is deleted from User Profile 2**, and since it was the only identifier, it results in leaving User Profile 2 effectively 'orphaned'. |
| No Setting | Type: `Modify`<br>MPID: `1234`<br>Customer ID: `h.jekyll.85`<br>Email `h.jekyll.md@example.com`<br>IDFV: `1234` | The modify request **updates the email of User Profile 1 only** to `h.jekyll.md@example.com`. Since email uniqueness is not enforced, both User Profile 1 and User Profile 2 now have the same email address identifier value. |

## Identity records

Behind the scenes, mParticle maintains a user profile for each user. You can think of a user profile as a folder of data that describes all the events, user attributes, identities, attribution info, and device info for a user. User profiles help determine which users are included in different audiences, and they enrich incoming data with any relevant user information before forwarding it to a connected output.
 
The main purpose of IDSync is to assign incoming data to the correct user profile. However, to identify users in real time, IDSync doesn't look at the entire profile, but at that profile's identity record.
 
Think of an identity record as a label on the front of your folder of user data (the user profile). The identity record contains a list of all identifiers that can be used to look up a profile. Identity records always have a 1:1 relationship with their corresponding profile.

There are two key points to remember about identity records:

- Some uses of IDSync force identifiers to be unique to a single identity record. Email addresses are a good example. See [Unique Identities](#unique-ids) for more information.
- The identity record might not contain every possible type of identifier available in a profile, but it will contain the identifiers that are specified in your identity hierarchy.

## Identity scope

mParticle data is organized in three tiers: organization → account → workspace. Your identity scope determines how user data is shared between multiple workspaces and accounts under your organization.

In other words, an identity scope is a set of user data in which each user profile and 'known user' identity is required to be unique. Multiple accounts or workspaces under a single mParticle organization can share the same scope, but a single workspace cannot be connected to more than one scope.

For some use cases, it might be beneficial for an organization to maintain more than one scope.

For example:

- Food delivery apps have both customers and couriers as users of their app ecosystem, but analytics requirements for each group are very different. Additionally, a courier may also use the app as a customer. Storing the data from both roles against the same profile could create confusion. By creating a separate Identity Scope for each set of users, data is kept clean and relevant.
- Large enterprise organization may not yet have a consistent way of identifying users across branches and subsidiaries. Creating separate Identity Scopes allow pools of differently identified users to be kept separate.
- Businesses that operate internationally may need to separate their customers geographically to comply with local laws.
- Multi-sided organizations, such as social media organizations, may conduct separate B2C and B2B business. For example, a user of a social media app may use the same login to post personal status updates and also to purchase advertising. Multiple Identity Scopes allow these activities to be considered separately.
