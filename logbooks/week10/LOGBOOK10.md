# Cross-Site Scripting (XSS) Attack Lab

## Task 1: Posting a Malicious Message to Display an Alert Window

After logging in as Alice, we enter the XSS payload to the brief description
field of the profile:

![task1a](task1a.png)

After saving and going back to the profile page, we get an alert:

![task1b](task1b.png)

This is an instance of Stored XSS, since the payload is stored in the user
profile and will be loaded everytime the profile page is visited.

## Task 2: Posting a Malicious Message to Display Cookies

For this task, we just replace the previous payload with the new one:

```html
<script>alert(document.cookie);</script>
```

And in the content of the alert we see that our cookies are being displayed:

![task2a](task2a.png)
