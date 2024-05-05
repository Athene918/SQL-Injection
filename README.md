# PortSwigger SQL Injection Labs

## Description Of Lab

````bash
This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs an SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and no error messages are displayed. But the application includes a “Welcome back” message in the page if the query returns any rows.

The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.

To solve the lab, log in as the administrator user.
````

- So, Get your favorite proxy tool and let’s start intercepting some traffic. My personal fav. is OWASP’s Zed Attack Proxy but you can use Burp Suite, That’s a pretty good tool too. And I usually like to use the inbuilt browsers from ZAP or Burp. Saves time and also you don’t get tonnes of unnecessary requests in the history of your proxy.

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*q1E4DSjPMNj00dFoAjUezQ.png)

- I’ll just launch Firefox from here and as soon as I start surfing the lab’s webpage, We’ll start seeing the requests in the history tab.

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*E-vuYmvDooUvY0sPeYqkOA.png)

- 
![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*E-vuYmvDooUvY0sPeYqkOA.png)

- Here we see a tracking ID. Now let’s try modifying the tracking ID and see if we see any difference in the web page. For that right click anywhere on the request box and select the “Open/Resend with Request Editor” Option.

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*FPrhBJrqpVxEkX1YmuZw0g.png)

- Here I’ve edited the TrackingId and appended the string “hehe” to it.

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*S7Ssa0dFiyzqXMs2SuSzgw.png)

- We can see the difference in the size of response of original and edited requests which is a difference of 41 bytes.

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*oY3pzYuSt5r_2iMZ6dCHsg.png)

- Further investigating, we find that if the TrackingId has the right value, we see a “Welcome back!” string.

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*NsJLOoN2lk2P82wjQFnD7g.png)

- And if the request has a bad value for TrackingId, the “Welcome back!” string vanishes. Like here belo

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*cxcDYWMQv_RNtUFIephhAQ.png)

- We will now try exploiting this parameter for possible SQL injection vulnerability.

- And let’s try to think about what SQL query might be going on inside the backend of the application. It’s possible that it might look something like this :

```bash
FROM tracking SELECT * WHERE TrackingId='W0eYyaVPHM3ny8gw'
````
- Now if this query returns a value(row/record), this would indicate the value True in boolean and if it returns an empty set, this would indicate False. If the application gets a True value, it prints “Welcome Back!” alongside Home and My Account. So, we can try making the value true for checking if blind SQL injection vulnerability exists.

- So, Here’s our request editor and were ready to inject SQL.

- Request 1 :
![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*ODOnp8fUj_JjoTs0_f7UMg.png)

- Response 1:

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*SAC22Ne2Hhd-NHaJ-1Yruw.png)

- Wait, i know you’re excited seeing this. We’re Welcomed By The Application, But this doesn’t make sure that it’s because of the SQL we injected. I’ll tell you why in a bit.

- Request 2:

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*4cCq7aRebeAa3pV8wR-WzA.png)

- Response 2:

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*EeCIdkXKvNZqcGYywTTbow.png)

- Now It’s the time to celebrate. We’re now sure of the app being vulnerable to SQL injection.

- If you’re guessing why now? That’s a great question. And To Be Honest I’m not an expert about the why but, Do you think that computers are highly programmable? Of course they are, and you’re right. So what if the app was programmed to take only the value of - -TrackingId as long as they’re from the set [a-z][0–9] and as soon it get’s a special character, it’ll terminate the value of TrackingId. If that would’ve been the case, We would’ve gotten the same response in both the cases as we’ve used a the ‘ character.

- We clear now? Great, Let’s move on now.

- And, let’s try some more injections(I Used to be scared of them as a kid, But these are interesting.)

- Response 3:

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*wS6bT6g3xEQVd8otx_nGyA.png)

- Another Response where we succeeded getting “Welcome back!” in the response. And using this, we’ll be able to extract the password of the administrator. Let’s see how.

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*1JD47sIgYbWo9KzWvrW8cA.png)

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*IMgUJmnX4Fggc-qZsFicUw.png)

- Not Lucky Enough. Let’s try other possible combinations of characters.

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*1-YORXC_h6qh7dpK5WJUJw.png)

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*4SCPJk6_bzFN5WhSdF7OhA.png)

- Now that we have our fuzzer opened. Let’s feed it payloads(the thing’s we wanna try, i.e. all the lowercase characters and numbers). Select Payloads Option.

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*GCmrFHOAfE_UooW31Z7DBA.png)

- Add the characters and start the fuzzer.

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*k_r1Zi806jnWx9x49gAmWA.png)

- In our responses of fuzzer, we have a request that stands out. The one with a Response size of 2963 bytes. Got it? Extra size for the “Welcome back!” message.

- So, now we know that the first character of password is h. Let’s find the others using the same method. By fuzzing with possible characters for all locations.

- Starting the fuzzer for possible characters at possible positions, We already started getting the password, one character at a time. And I’ve sorted the requests by the size of their responses, this way it’s easy to find the responses which have the “Welcome back!” string.

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*fLnOIZqfWWp8TSkWQt9Thw.png)

- And we have our password built up. Type it somewhere and login as administrator with the password and we’re done.

![image](https://miro.medium.com/v2/resize:fit:640/format:webp/1*4h7JX5hkFsPCtpBYwlMkIg.png)

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*qdLfj1Pa9uMe0OyBzB2Pmw.png)
