# 0003: Allowing phone numbers to create an account

- **Author**: Oliver Roick, Adrienne Platner
- **Status**: Accepted, 2017-02-01

## Context

Users currently need to register with an email address and confirm this email address via the link provided in a confirmation email. 

We operate in countries where the number of people who have access to email is little. Project managers create both email accounts and platform accounts for users that don’t have an email address. Which is not convenient, time-consuming and introduces security risks, because many might have access to the same account. 

Many more people do have a mobile phone; the phone number can be used to register accounts (instead of using email) and to verify the account. 

## Decision

In addition to emails, we will allow creating accounts by providing a mobile phone number. Users provide either a valid email address or a valid phone number to create an account

**Details**:

- An SMS is sent to the phone including a one-time code that needs to be entered on the site to verify the account. 
- An SMS is sent to the phone including a one-time code to reset the password. The account owner enters the verification code along with the phone number on the platform site to be able to reset the password. 
- When the phone number is changed, a verification code is sent to the new number. The code must be entered on the site the verify the new number. An additional SMS is sent to the existing phone number advising of the change and directing the user to contact their project manager/owner if this is unauthorized. This is a generic message that includes only a point of contact at Cadasta; none of the user's or project manager's identifying information is included. 

## Consequences

By allow users to sign up with a mobile phone number, users who do not have an email address will be able to sign up to the Cadasta platform. 
