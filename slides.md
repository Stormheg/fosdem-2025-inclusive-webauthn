---
# You can also start simply with 'default'
theme: default
# some information about your slides (markdown enabled)
title: "Secure and Inclusive: WebAuthn for (Multi-Factor) Authentication"
info: |
  ## Secure and Inclusive: WebAuthn for (Multi-Factor) Authentication
background: /background-pattern.svg
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true

layout: cover
---

# **Secure and Inclusive:** WebAuthn for (Multi-Factor) Authentication

<div class="absolute bottom-10">
  <span class="font-700">
    Storm Heg | FOSDEM | February 2nd, 2025
  </span>
</div>

---
layout: intro
transition: fade-out
---

# `whoami`

- **Storm Heg**, he/him, The Netherlands 🇳🇱
- **Independent contractor**, building quality websites with Wagtail CMS and Django (Python) for a living since 2019
- **Open Source ❤️**
  - Member of the Wagtail CMS core team
  - Admin for the Django Commons initiative, helping facilitate community maintenance of open source in the Django ecosystem
  - Creator of open-source of my own, such as [`django-otp-webauthn`](https://github.com/Stormbase/django-otp-webauthn)

---
layout: default
transition: fade-out
---

# Contents

- **Why is Multi Factor Authentication important?**
- **Security vs usability trade-offs**
- **Introducing WebAuthn**
- **Why WebAuthn is a better alternative**
- **What you need to implement WebAuthn**
- **Open Source you can use**

---
layout: two-cols-header
transition: fade-out
---

## Why is Multi Factor Authentication important?

<div v-click>

**Passwords are not enough**

- Hard to remember – that's why `Welcome123!` is a thing...
- Preyed on by phishing attacks
- Reuse of passwords leads to credential stuffing
</div>

<span v-click>(Yes, Password managers exist – but not everyone uses them)</span>



::right::

<div v-click>

**MFA adds an extra layer of security**

- Something you know (password)
- Something you have (phone, security key)
- Something you are (biometrics)
</div>


<style>
  ul {
    list-style-type: none;
  }
</style>

<!--
Let's take a look at the shortcomings of passwords.

- Hard to remember
- Preyed on by phishing attacks
- Reuse of passwords leads to credential stuffing (where an attacker uses a known username/password combination from a data breach to try and log in to other services)

Password managers exist, but not everyone uses them or knows how to use them.

I'd say that password managers were invented as a workaround for the shortcomings of passwords

With multi-factor, we are trying to add another layer of security by requiring something else in addition to the password. Like something you have (your phone, that receives SMS codes or generates TOTP codes), or something you are (biometrics).
-->

---
layout: default
transition: slide-left
---

## Security vs. Usability

<br/>

<Transform :scale="0.8">
  <h2 v-click>Has this ever happened to you?</h2>
</Transform>

<br/>

<ul>
  <li v-click>You are waiting for an email to arrive with a code to log in</li>
  <li v-click>You need an SMS code, but your phone has no reception</li>
  <li v-click>You switched phone numbers, and now you can't log in because you didn't update your number</li>
</ul>

<strong v-click>But all in the name of security, right?</strong>

<!-- 

Now, has any of the following situations ever happened to you?

- You are waiting for an email to arrive with a code to log in
- You need an SMS code, but your phone has no reception
- You switched phone numbers, and now you can't log in because you didn't update your number


Annoying situations to be in right? It's an inconvenience to wait for something to arrive or to take out your phone to approve a login.
-->

---
layout: default
transition: slide-left
---

## Security vs. Usability - continued

<ul>
  <li v-click>Emails and SMS are still vulnerable to phishing attacks</li>
  <li v-click>Not very intuitive to use for everyone</li>
  <li v-click>Creates friction logging in to your service</li>
</ul>

<strong v-click>Isn't there a better way...</strong>

<!-- 
Okay, we introduced inconvenience but we still solved our security problem right?

Sort of, we just made it a harder for an attacker. But phishing is still a great risk to the security of our service.

And in exchange for this added security we also made it more difficult for our users to log in to our service. That's not very inclusive is it? Can't we come up with something better?
 -->


---
layout: two-cols
image:
---

## Entering the WebAuthn era

(You might have heard of it already as 'Passkeys')

- **WebAuthn** is a web standard for secure, passwordless authentication
- **Based on public-key cryptography**
- **Supported by all major browsers**
- **Around since 2020** or so, but relatively unknown
- **Gaining traction** due to Apple prominently supporting it in iOS 16
- **Native, predictable prompts** for a consistent user experience
- **The future of authentication**, according to yours truly 😉

::right::

<img src="/login_with_passkey.png" alt="macOS system prompt asking for a fingerprint scan to confirm login" class="mx-auto" />

<!--

I wouldn't be standing here if I didn't believe there was a better way. And there is! It's called WebAuthn, short for Web Authentication.

You may have heard of it already as Passkeys.

It's a web standard for secure, passwordless authentication. It's based around public-key cryptography and does not rely on shared secrets like passwords do.

It's supported by all major browsers and has been around in the form I'm describing today since 2020 or so. It's gaining traction due to Apple prominently supporting it in iOS 16.

What I like about it is that it provides a native, predictable prompt for the user to interact with. This makes it consistent user experience across different sites.

I think it's the future of authentication, but first, let's speed-run through how it works from a high level.

 -->

---
layout: two-cols-header
---

# Quick overview of WebAuthn

&nbsp;

<strong>When registering:</strong>

 <ol>
  <li v-click>Ask the browser to create a public-private key-pair <code>navigator.credentials.create()</code></li>
  <li v-click>Send the public key to the server, store with reference to user</li>
  <li v-click>Securely store the private key on the device</li>
</ol>

<p v-click>This public-private key-pair is informally called a <i>Passkey</i></p>

<strong v-click>And when logging in:</strong>

<ol>
  <li v-click>Ask the browser to sign a challenge with the private key <code>navigator.credentials.get()</code></li>
  <li v-click>Send the signed challenge to the server</li>
  <li v-click>Verify the signature is correct using the public key</li>
  <li v-click>Log in the user associated with the public key if the signature is correct</li>
</ol>

<!-- 
  When a user wants to register a Passkey, we call a browser api to create a public-private key pair. The public key is sent to the server, where it is stored with a reference to the user. The private key is securely stored on the device.

  This key pair is what is informally called a Passkey.

  When a user wants to log in, the browser is asked to sign a challenge with the private key. The signed challenge is sent to the server, where it is verified that the signature is correct using the public key. If the signature is correct, the user is logged in.
 -->

---
layout: default
---

## Benefits of WebAuthn

&nbsp;

- **No passwords** to remember
- **No shared secrets**
- **Unique for every site**
- You can register separate Passkeys for **multiple devices**
- **Phishing-resistant**, only works on the site it was registered on
- The Passkey <small>(_something you have_)</small> is typically protected by **biometrics** <small>(_something you are_)</small> or your device's **PIN/Password** <small>(_something you know_)</small>
  - There is your additional factor! 🎉

<!-- 

Benefits of WebAuthn:

- No passwords to remember
- No shared secrets
- Unique for every site - if a Passkey is compromised on one site, it doesn't affect others
- You can register separate Passkeys for multiple devices - not something that is possible with passwords
- Phishing-resistant, they only work on the site they were registered on. You can't use a Passkey to login to a phishing site on another domain.

The device that saved the Passkey is going to protect it with biometrics or a PIN/password. So there is your additional factor!

This is an optional feature you can request when necessary, WebAuth calls this "user verification required".

 -->

---
layout: two-cols
---

## Drawbacks of WebAuthn

(Because no technology is perfect)

- **Losing your Passkey is still a risk** 
- **Cloud sync** - a risk?
  - Your `<insert BigTech>`-account is now a single point of failure
  - (but so was your password manager already...)

<!-- 
Let's touch on some drawbacks of WebAuthn:

- Losing your Passkey/device is still a risk - if your device is lost, stolen or replaced, you can't use the Passkey stored on it anymore.
- Cloud sync - convenient to overcome the previous point, but also poses a risk. Your BigTech-account is now a single point of failure. But then again, so was your password manager already.

Side note: most password managers can also do WebAuthn, so you can use your password manager to store your Passkeys if you want to.

 -->

---
layout: default
---

# Implement WebAuthn

Implementing WebAuthn is quite involved

**Server-side**:

- [ ] API endpoints for **registration** and **authentication** ceremonies
- [ ] **Store** the public key of the Passkey
- [ ] **Verify** the signature of the Passkey

**Client-side**:

- [ ] Communicate with your server
- [ ] Make the appropriate `navigation.credentials.*` browser API calls
  - [ ] <small>and handle any exceptions that may arise 🤐</small>
- [ ] Implement any additional features, like Passkey autofill <span>by calling the browser API slightly differently</span>
- [ ] Area on your site for users to manage their Passkeys

<span v-click>In general, spend effort to make it a smooth experience for your users</span>

<style>
  ul {
    list-style-type: none;
  }
</style>

<!-- 

Do I have you convinced to implement WebAuthn in your application yet? Great! Let's talk about what you need to do to implement it. Because doing that from scratch is quite involved.

On the server-side, you need to implement API endpoints for the registration and authentication ceremonies. You need to store the public key of the Passkey and verify the signature of the Passkey.

On the client-side, you need to communicate with your server, make the appropriate browser API calls and handle any exceptions that may arise. You also need to implement any additional features, like Passkey autofill by calling the browser API slightly differently. And you need to provide an area on your site for users to manage their Passkeys.

In general, it is quite some effort to implement WebAuthn well. Can't it be easier? Yes, it can!
 -->

---
layout: full
---

# Your Open Source options

Plenty of libraries and frameworks exist to help you implement WebAuthn in your application!

| Framework     | Libraries                                                  |
| ------------- | ---------------------------------------------------------- |
| Django        | `django-otp-webauthn`, `django-passkeys`, `django-allauth` |
| Node.JS       | `@simplewebauthn/server`, `webauthn`                       |
| Symphony      | `webauthn`                                                 |
| Laravel       | `laragear/webauthn`                                        |
| Ruby on Rails | `webauthn-ruby`                                            |

<span v-click>Disclaimer: I'm the author of `django-otp-webauthn`</span>

<!-- 
If you are building a website you are most likely using a framework or library to help you. Like Django, Node, Laravel, or whatever name your favorite framework has.

Good people have already gone through the trouble of implementing WebAuthn for you by implementing libraries for these frameworks. So you don't have to do it all from scratch.
 -->

---
layout: statement
---

## Thank you!
