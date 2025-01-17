# 2FA_broken_logic-Exploit

An automated exploit for Port Swigger Web Security Academy's "2FA broken logic" Lab

NOTE: This repo contains only the .py script file for the exploit. No dependencies or supplementary files are included. 

BACKGROUND

This is a quick, automated exploit written in python for the Port Swigger Web Security Academy Lab titled "2FA broken logic". The exploit was developed out of frusturation with the Burp Suite Community Edition tool. This lab involves sending a large number of requests to the server in order to brute-force a victim users account. In this particular lab, there is a pool of 10,000 potential values (for a 4-digit verification code) that have to be iteratd through in the request. The Community Edition of Burp Suite puts a choke on requests sent after a certain cap is reached (~ 50 or less requests in my experience). After this point, the requests are sent at an extruciatingly slow rate. Solving the lab in this way is impractical - scripting the exploit is much quicker and funner! 

Luckily, no such restrictions on requests to the server are enforced in this handy python scripts, which sends the reqeusts at a reasonable and CONSISTENT pace. Note that Burp will send the requests and process the responses at a _much_ quicker rate than this script before the cap is employed, meaining that if you are lucky enough to hit the target value in the first ~50 requests, solving the lab with Burp is a much faster solution. Also note that this choke limiting does not exist in the Burp Pro Edition; again, using Burp Pro, one can solve this lab much more quickly than with this script. 

GENERAL LAB INFORMATION

In this lab, we are given a simple, bare-bones web application with a login function allowing users to sign into their account. The attacker (ie. us) has an account and can sign in using credentials provided. Once the credentials are submitted, a verification code is created on the back-end and sent to the user's email; this code can be accessed by opening the 'Email Client' web-page in the browser. The code is submitted and verified, after which access to the user acount is granted. 

Along with credentials to our own account, we are also provided with the valid username for the victim account, 'carlos'. The goal of the lab is to exploit a flaw in the 2FA logic and gain access to the victim account. 

VULNERABLE WEBSITE

The vulnerable website has several pages of interest:

      1. https[:]//some_subdomain[.]web-security-academy[.]net
      
         The websites homepage
         
      2. https[:]//some_subdomain[.]web-security-academy[.]net/login
      
         The first login page which prompts the user to enter their account credentials (username/password)
         
      3. https[:]//some_subdomain[.]web-security-academy[.]net/login2
      
         The second login page which prompts the user to enter their verification code 
         
      4. https[:]//some_subdomain[.]web-security-academy[.]net/email
      
         The email client page where the user can claim their verification code 
         
After posting our user credential on the /login page, a verificaton code is sent to the email client and we are directed to the /login2 page. From here, two interesting observations may be made. 1) The verificaiton code that is generated is a 4-digit numerical value, which means it is brute-forcible. 2) A new cookie value is included in the GET request to /login2, {'verify': '_username_'}. This new cookie field is used on the backend to determine which account is attempting to verify with the verification in the POST to /login2. Using these facts, we can develop the following exploit.


THE EXPLOIT

The plan of attack is simple:

1. Send a GET request to /login2 with 'verify' set to 'carlos'. This will trigger the creation of a verificaion code for the account.
2. Send a flood of POST requests to /login2 with 'verify' set to carlos. Iterate the 'mfa-code' value in each request so that all possible value from 0000-9999          are tried until a successful login is complete. 

We can script this exploit easily with python. Since each instance of the lab will include a new subdomain for the URL, we begin by starting the script and entering the subdomain that has been assigned. The URL and 'Host', 'Referer' and 'Origin' header values for the GET/POST requests will be built using this subdomain value. The User-Agent header is is left blank, but can be populated as needed - all other headers are hardcoded but can be modified as needed directly in the code. 

There are 2 keys in the Cookies field: 'session', which initially has an arbitrary value hardcoded in and is then updated from the server responses, and 'verify', which has the victim account 'carlos' hardcoded in. 

The exploit begins by submitting the aforementioned GET reuquest, built with the custom parameters subitted by the user. The flood of post requests are then sent.
After each POST request is sent, the status code of the response is checked - if it is a 302, then we have successfully logged into the account and completed the lab. The valid verification code is returned, the for loop breaks and the program terminates. If it is not a 302, then the program continues to iterate through the possible code values and send the corresponding POST requests. 
      
Once this script finishes, we will have successfully accessed the 'carlos' account, having completely bypassed the password step in the authentication process. 

EXCEPTION HANDLING

Again, this ia a quick and dirty scrpt - built in are catches for 2 exceptions

1. **requests.exceptions.ReadTimeout** in the initial GET request - the timout value is in the request is set to 5 seconds, after which the user will recieve an error message and be advised to restart the program and the program terminates. 
2. **requests.exceptions.ConnectionError** during the request flooding - the handle_retry function is called, which will attempt to send the request an additional 5 times with a set wait period between requests. If a 200 or 302 is recieved, the function will terminate accordingly. Otherwise, the program will terminate with a message to restart

FUTURE IMPROVEMENTS

There are no immediate plans for changes to the script in its current form, but handling errors more gracefully and validating the subdomain entered by the user are potential improvements to explore. The script can also be modified to solve other Port Swigger lab challenges. 

