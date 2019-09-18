# Mark of the Web Explainer

## Author:

- huangdarwin@chromium.org

## Introduction

The web clipboard is does not currently indicate that its data originates from the web. This means that, unlike with downloaded files, native applications are unable to use such information to optionally provide defense in depth or other protections based on this information.

This explainer proposes a Mark of the Web (MoTW) for the Web Clipboard. This MoTW would be a format written in parallel with other already-written formats, which can inform native applications that the format originated from the web, as well as which source url the data came from.

## Goals

- Provide an additional method for defense in depth, as the Downloads Mark of the Web does.
- Preserve user privacy/security.

## Non-goals

- Require all native applications to check this MoTW (Like with Downloads, not all native applications must check this MoTW, especially at first).
- Discuss raw clipboard access design. (While the author of this explainer may also be considering raw clipboard access, this MoTW would be applied on all web clipboard data. It’s not advantageous to have a hypothetical, separate “sanitized clipboard MoTW” and “unsanitized clipboard MoTW”.)

## Mark of the Web

Mark of the Web would add an additional format, written on every write from a user agent. This should not be visible on read for existing web clipboard APIs, but would be visible to native application clipboards, and could be accessible via the proposed raw clipboard access API.

```js
// NOTE: This is a hypothetical representation in JS, 
// but Mark of the Web will not be visible in JS, so a
// web application developer may not see any change.

// Without Mark of the Web
await navigator.clipboard.writeText(‘hello world’);
// clipboard contents after write:
// {
//   ‘text/plain’: ‘hello world’
// }

// With Mark of the Web
await navigator.clipboard.writeText(‘hello world’);
// clipboard contents after write:
// {
//   ‘text/plain’: ‘hello world’,
//   // Only present on a clipboard write from a user agent.
//   // Equal to ‘about:internet’ when in incognito or if the
//   // url is too long.
//   ‘text/web-source-url: ‘example.com’ 
// }
```

## Considered alternatives

### Exclude source url?

Providing a source url could allow antivirus scanners to detect websites that attempt to place malicious content on the clipboard, and has parity with the shape of Downloads, where both source url and referrer url are provided.

However, providing a source url is a privacy concern, as this is new information that was not previously exposed to the clipboard. In most platforms, any native application would be able to continuously read/poll the clipboard in the background, so this is introducing new information to native applications, which could be used by a malicious application to fingerprint a user (see which sites a user has visited and copied from). However, the improvement in security, by allowing native applications to better identify untrusted data, could be worth the privacy concern. 

Native applications in some platforms can already passively view screen content (ex. remote desktop applications), so this may not be a significant privacy concern.

### Provide referrer url?

As with providing a source url, providing a referrer url is also a privacy concern, as this could inform a malicious listening native application of multiple sites in a user’s browsing history with each user copy. As the value of providing a referrer url seems fairly small, this explainer has opted to exclude a referrer url.

### Omit all url data?

A Mark of the Web doesn’t strictly need to include origin sites. Instead, an alternative design could simply name the *type* of origin. A potential format name could be `'text/origin'`, and potential values could be `'incognito'`, `'web'`, and `’installed-web-application’`. 

While this would still potentially degrade privacy, by easily providing information to native applications regarding the type of source for clipboard data, the security benefits should outweigh the privacy concerns. That said, the risk of providing a source url seemed fairly small, so this was not the chosen path.

Note that given the current design of raw clipboard access, this could allow for leakage of an origin of clipboard data, which may result in leakage of this data to web applications, given this information.

## Stakeholder Feedback / Opposition

N/A

## References & acknowledgements

Many thanks for the valuable feedback and advice from:

- engedy@chromium.org
- pwnall@chromium.org
- slightlyoff@chromium.org
- Chromium’s Windows and MacOS implementations.

