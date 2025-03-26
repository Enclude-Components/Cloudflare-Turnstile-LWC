# Screen Flows: A Security Deep Dive
This document aims to answer the question of whether a screen flow can be protected against bot attacks using a tool such as Google's Recaptcha. The short answer is "Yes", but to have a chance of implementing it in a secure way, one must carefully consider how the screen flow functions. 

## Anatomy of a Screen Flow

Screen flows can be thought of as a series of forms being presented to a user. Each screen action taken makes a request to the server with details of the the user's intent. An example of a screen flow request body looks like this (some irrelevant keys removed):

```JSON
{
    "action": "NEXT",
    "fields": [
        {
            "field": "Input",
            "value": "Test",
            "isVisible": true
        }
    ],
    "serializedState": "AAAAW3sidC...",
}
```

### "action"
This corresponds to the button pressed by the user (ex. Previous, Next, etc.)

### "fields"
This array contains the data of the input elements available to the user. In this example, there is a text input called `Input` with user input value of `Test`. 

### "serializedState"
This is a very long string representing the entire state of the flow. That state of the flow may include server-side variables, the current screen the user is on, which paths they've taken, etc. Another way to imagine this is that the state of the flow exists in this string, not on the server. A flow's state can always be reconstructed from this string, and when a user requests the next screen of the flow, they include this string in the request, which the server uses to reconstruct the flow in order to serve the next screen to the user.

Here is the full string in my example, which deserves some closer examination

```
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
```

#### Component Parts of the Serialized Encoded State:

##### Binary Length Prefix
`AAAAW+` is a base 64 binary integer, `91` in  decimal. A `+` is added at the end for padding. ([Base 64 to Binary Converter](https://cryptii.com/pipes/base64-to-binary)). This gives the length of the unencrypted header discussed in the next section, presumably so the server can know how far to read in order to retrieve the head information.

##### 91-Character JSON Header
The next substring is
```
+3sidCI6IjAwREFVMDAwMDBBS2E1YyIsInYiOiIwMkdBVTAwMDAwMTFMRWsiLCJhIjoiZmxvd2VuY3J5cHRpb25rZXkiLCJ1IjoiMDA1QVUwMDAwMElrTG5NIn3sl9Q
```
A `+` is added at the beginning for padding. ([Base 64 Decoder](https://www.base64decode.org/)). The result is the 91-character length JSON string below
```JSON
{"t":"00DAU00000AKa5c","v":"02GAU0000011LEk","a":"flowencryptionkey","u":"005AU00000IkLnM"}
```
This payload contains an Organization ID and, presumably, directions on how to decrypt the remaining string on the server.

##### The Remaining Serialized Flow State

The remaining string seems to be an encrypted string representing the flow's state. Disclaimer: I can't find any Salesforce documentation to officially support this, but the following three points lead me to believe that it is:
1. The remaining string does not decode from base 64
2. Its entropy is higher than the unencrypted header, meaning it's apparently more random
3. There is a `flowencryptionkey` in the header.

## Security Implications

### The Good
> [!TIP]
> Flow State is Safe from Arbitrary Manipulation

Given that the state of the flow is encrypted with each request, a malicious user cannot modify the string to produce any arbitrary valid state of the flow. For instance, if there is a variable in the flow whose value is hard-coded, a malicious user could not guess how to modify the serializedState string to modify the value of the variable.

### The Bad
> [!CAUTION]
> Flow State is Vulnerable to Replay Attacks

However, once a malicious user has a serializedState string that encodes a valid flow state which they intend to abuse, they can reuse that string for further subsequent requests, a kind of replay attack.

For example, imagine a flow with two screens:
1. Recaptcha Test: This screen presents a recaptcha challenge to confirm that the running user is not a bot. When the user presses `Next`, the result is sent to the server to safely validate it. A variable `user_is_human` is set to `true`. Then they are returned the next screen.
2. Sensitive Form: This screen presents a form which allows users to create a record, perhaps sending an automated email to the provided address.

If a malicious user wants to make a request to the protected form on Screen 2 via an automated bot script, they could simply complete the form once as a human. After completing the Recaptcha challenge, they will receive a serializedState string whose `user_is_human` is set to `true`. They need not even know that this variable exists; the malicious user only knows that this state is valid in order to advance to the subsequent screen. They then include the forged serializedState in their bot requests without ever needing the bot to complete the challenge.

## The Resolution
In order to protect a screen flow on a public site, the Recaptcha test would need to be validated server-side immediately before executing the protected action (ex. inserting records or sending emails) without giving the user an opportunity to forge the serializedState string. This may mean, for example, ensuring the the Recaptcha test is on the same screen as the input form and validating it first before carrying out any other actions. If a flow has multiple protected actions on different screens, this would mean having to perform multiple independent Recaptcha tests, since any subsequent ones could be done with a forged state.