---
id: security
title: Threat Models and Security Profiles
---

Running any software that stores personal information exposes the
developer/company to risks. Analyzing which threat agents pose a risk,
understanding the possible motivations for an attack, or why an agent is a
threat, knowing the attack surface, the likelihood, and the impact are important
all aspects of a threat model.

This documentation can not substitute a thorough and serious threat model, yet
it will provide some guidelines to help configure ORY Kratos in a way that makes
it best suited for any risk assessment.

> Please be aware that this chapter is still work in progress. Not all
> mitigation strategies have been implemented yet in ORY Kratos!

## Understanding Threats

This section examines several threat vectors in systems that manage identities.

### Account Enumeration Attacks

> "Often, web applications reveal when a username exists on system, either as a
> consequence of a misconfiguration or as a design decision. For example,
> sometimes, when we submit wrong credentials, we receive a message that states
> that either the username is present on the system or the provided password is
> wrong. The information obtained can be used by an attacker to gain a list of
> users on system. This information can be used to attack the web application,
> for example, through a brute force or default username/password attack.
> Description of the Issue"
>
> [Source](<https://wiki.owasp.org/index.php/Testing_for_User_Enumeration_and_Guessable_User_Account_(OWASP-AT-002)>)

#### Scenarios

Considering the above, an example would be for example an adult website. A
threat agent wants to blackmail a well known politician by checking if someone
can sign up at that website using the `well-known-politician@email.com` email.

If the service responds with
`Sorry, that email is already signed up here. Did you try to log in instead?`,
the agent is able to proceed with some type of blackmail scheme.

[OWASP defines several Black-Box tests](<https://wiki.owasp.org/index.php/Testing_for_User_Enumeration_and_Guessable_User_Account_(OWASP-AT-002)#Black_Box_testing_and_example>)
that cover Account Enumeration Scenarios.

#### Mitigation

ORY Kratos can be configured to send an out-of-band message to the email used
for login, registration, account recovery, etc.:

- If an application or user tries to sign in using an unknown email address, an
  email will be sent to that address reading "You tried to sign in at X but you
  do not have an account yet, did you mean to sign up instead?"
- ...

### Bruteforce Attacks

Will be addressed in a future release.

### Phishing Attacks

Will be addressed in a future release.

### Social Engineering Attacks

Will be addressed in a future release.

### SMS Spoofing Attacks

Will be addressed in a future release.

## Choosing the right Security Profile and Configuration

Will be addressed in a future release.

### Argon2

ORY Kratos uses Argon2 for password hashing. Argon2 is the official winner of
the PHC 2017. You can tweak the Argon2 configuration in your ORY Kratos
configuration file:

```yaml
hashers:
  argon2:
    memory: 1048576
    iterations: 2
    parallelism: 4
    salt_length: 16
    key_length: 32
```

## Digital Identity Guidelines

There is no one standard to digital identity. ORY Kratos closely follows
emerging frameworks and guidelines such as:
[Digital Identity Guidelines established by the National Institute of Standards and Technology (NIST)](https://pages.nist.gov/800-63-3/)
(and a follow-up [FAQ](https://pages.nist.gov/800-63-3/)) .

As ORY Kratos grows, this document will continue to expand and add sections
covering individual security recommendations established by NIST.

### Password Policy

Almost every service with a login offers some type of registration using a
password. Therefore, there are many strategies floating around, with many of
them implementing terrible and insecure patterns such as:

- Not allowing special characters in passwords.
- Not allowing the use of password managers by disabling the "paste"
  functionality in password fields.
- Requiring you to rotate your password every month.
- ...

Troy Hunt has written an
[excellent piece on password policies](https://www.troyhunt.com/passwords-evolved-authentication-guidance-for-the-modern-era/)
and why they recently changed and how.

ORY Kratos implements a password policy that:

- Checks if a password has previously been leaked using the
  [HIBP API](https://haveibeenpwned.com/API/v2); and
- Checks if a password is too similar to one of the identifiers (in a future
  release [kratos#184](https://github.com/ory/kratos/issues/184)).

This is a rundown of all the practices ORY Kratos implements and why. **Some
things need to be implemented by yourself** as they must be implemented in the
User Interface that interfaces with ORY Kratos. You can find these in section
[User Interface Guidelines](#user-interface-guidelines).

#### Password Complexity

This outline and quotes are defined in the
[NIST Digital Identity Guidelines - 5.1.1.2 Memorized Secret Verifiers](https://pages.nist.gov/800-63-3/sp800-63b.html).
ORY Kratos, unless explicitly advertised, implements these guidelines and best
practices.

Passwords must have a minimum length of 8 characters and all characters
(unicode, ASCII) must be allowed:

> Verifiers SHALL require subscriber-chosen memorized secrets to be at least 8
> characters in length. Verifiers SHOULD permit subscriber-chosen memorized
> secrets at least 64 characters in length. All printing ASCII [RFC 20]
> characters as well as the space character SHOULD be acceptable in memorized
> secrets. Unicode [ISO/ISC 10646] characters SHOULD be accepted as well. To
> make allowances for likely mistyping, verifiers MAY replace multiple
> consecutive space characters with a single space character prior to
> verification, provided that the result is at least 8 characters in length.
> Truncation of the secret SHALL NOT be performed. For purposes of the above
> length requirements, each Unicode code point SHALL be counted as a single
> character.

Passwords must be checked against a database of compromised secrets such as
[Have I Been Pwnd](https://haveibeenpwned.com):

> When processing requests to establish and change memorized secrets, verifiers
> SHALL compare the prospective secrets against a list that contains values
> known to be commonly-used, expected, or compromised. For example, the list MAY
> include, but is not limited to:
>
> - Passwords obtained from previous breach corpuses.
> - Dictionary words.
> - Repetitive or sequential characters (e.g. ???aaaaaa???, ???1234abcd???).
> - Context-specific words, such as the name of the service, the username, and
>   derivatives thereof.
>
> If the chosen secret is found in the list, the CSP or verifier SHALL advise
> the subscriber that they need to select a different secret, SHALL provide the
> reason for rejection, and SHALL require the subscriber to choose a different
> value.

Show the user a password-strength meter (to be implemented, see
[#136](https://github.com/ory/kratos/issues/136)):

> Verifiers SHOULD offer guidance to the subscriber, such as a password-strength
> meter [Meters], to assist the user in choosing a strong memorized secret. This
> is particularly important following the rejection of a memorized secret on the
> above list as it discourages trivial modification of listed (and likely very
> weak) memorized secrets

Do not require mixtures of characters types or prohibiting repeated characters:

> Verifiers SHOULD NOT impose other composition rules (e.g., requiring mixtures
> of different character types or prohibiting consecutively repeated characters)
> for memorized secrets. Verifiers SHOULD NOT require memorized secrets to be
> changed arbitrarily (e.g., periodically). However, verifiers SHALL force a
> change if there is evidence of compromise of the authenticator.

#### User Interface Guidelines

These best practices need to be implemented in your User Interface and can not
be handled by ORY Kratos. All ORY-built reference and demo applications
implement these best practices:

Allow pasting of passwords:

> Verifiers SHOULD permit claimants to use ???paste??? functionality when entering a
> memorized secret. This facilitates the use of password managers, which are
> widely used and in many cases increase the likelihood that users will choose
> stronger memorized secrets.

Allow the user to show the secret in the UI:

> In order to assist the claimant in successfully entering a memorized secret,
> the verifier SHOULD offer an option to display the secret ??? rather than a
> series of dots or asterisks ??? until it is entered. This allows the claimant to
> verify their entry if they are in a location where their screen is unlikely to
> be observed. The verifier MAY also permit the user???s device to display
> individual entered characters for a short time after each character is typed
> to verify correct entry. This is particularly applicable on mobile devices.

#### Password Hints

> Memorized secret verifiers SHALL NOT permit the subscriber to store a ???hint???
> that is accessible to an unauthenticated claimant.
>
> [NIST Digital Identity Guidelines - 5.1.1.2 Memorized Secret Verifiers](https://pages.nist.gov/800-63-3/sp800-63b.html)
