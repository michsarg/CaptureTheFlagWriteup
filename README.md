## 3004 Cybersecurity Project
## Capture The Flag: Select Write Ups

The final project for UWA unit CITS3004 Cybersecurity involved 22 capture the flag challenges. I completed all challenges and received full marks but have only included the most unique and interesting challenges here. The full set of CTF challenges is available at https://github.com/Ccamm/CITS3004-Project

---

### 05 - Web Ninja

#### Overview
A website build with the Flask framework features of an email sign up page with a vulnerability.

#### Process
In researching Jinja vulnerabilities, a suggestion was found to use a *Server Side Template Injection*. I found helpful guide at https://medium.com/@nyomanpradipta120/ssti-in-flask-jinja2-20b068fdaeee. Burpsuite Repeater was used for this task due to the ease of repeating requests with simple alterations. 

I began by using the input *{{config}}@gmail.com* to the *email* field which dumped details of the config object, which interestingly contains executable functions. Then I was able edit the payload to traverse up the hierachy to find *subprocess.Popen* was accessible. This enabled me to submit terminal commands via the payload and find the flag. 

The final input was: 
*email={{"".__class__.__mro__[1].__subclasses__()[279]('cat flag',shell=True,stdout=-1).communicate()}}@gmail.com*

#### Flag
*CTF{tH3s3_n1Nj4s_aR3_h3cK1nG_mY_jInjA_t3MpLatEs!!11one!}*

---

### 07 - NoSQL Injection

#### Overview
The password for the admin account of a website using NoSql needs to be found. A hint is provided that the password has only lower and upper case letters and starts with CTF.

#### Process
I used Burpsuite for this attack. First I used the proxy module to capture a log in attempt, then sent it to the intruder module This allowed for automation of the a bruteforce attack by using regex, and detecting the admin password char by char. A successful discovery was indicated by the presence of */admin/* in the response. The line used in the request was: *{"username":"admin","password": {"$regex": "^§val1§.*"}}* where *§val1§* is the bruteforced char.

#### Flag
*CTFnosqlregexking*

---

### 08 - SecureApp PWN

#### Overview
This task required exploiting an app with a stack overflow vulnerability. An offline version of the app was provided for developing the exploit, but it had to be implmented on a remotely hosted version to obtain the real flag.

#### Process
Firstly, the offline secureapp needed chmod changes to make it executable. Then it was run with *edb-debugger*, and a very long non-repeating pattern string was input causing a stack overflow error. The edb-debugger then allowed me to determine which point of the input string was the end of the stack: the 219th position.

Now if I could input a payload of length 219, then follow this with the memory address of a vulnerable function, the function would be executed and return the flag. To find the vulnerable function I used *readelf*, and it seemed like the helpfully named "exploitme" was probably the right function to exploit. Running *objdump -d secureapp* indicated it was located at the address *000000000040125f*.

Finally, a payload could be created of 219 bytes followed by the *exploitme* address. This was saved to a bin file, then *pwntools* was used to insert the payload, execute the function, and access the flag.

#### Flag
*CTF{pwN3D_uR_w3aK_C_aPpLiC4sHon!11!}*

---

### 10 - Brain XOR Brawn

#### Overview
A ciphertext encoded as hex values requires decrypting with an XOR stream cipher using an 8 byte long key. The beginning of the plaintext is revealed to be CITS3004. A hint is given: If P XOR K = C then P XOR C = K.

#### Procedure
The hint and beginning characters are very useful, as we know the key is 8 chars long and the start of the plaintext also happens to be 8 chars long. So, using CyberChef, we can easily P XOR C = K and obtain the 8 char key, which is then repeatedly applied to the remaining ciphertext.

#### Flag:
*CITS3004{c1pH3rT3xT_pL41nT3xT_4tt4CK_1nC0mInG!!1!!}*

---

### 12 - Rocking With the Cats

#### Overview:
The task requires cracking a password hash, with the hints that the password is 32 characters long and in the infamous wordlist *rockyou.txt*.
The provided hash was: $2a$13$1MwIkr1P9kFp1beWyfNdVuVRGIDX1pnGtGuBGJ8wkEzVeBlF.CdNW

#### Procedure:
Firstly,the hash was determined to be using bcrypt due to the $2a$ prefix. Grep was then used to create a reduced dictionary of exclusively 32 character passwords, using the command: *grep -E '^[0-9a-zA-Z]{32,32}$' rockyou.txt > len_32.txt*. Finally, *hashcat* was used to brute force the hash using the reduced dictionary with the command: *hashcat -a 0 -m 3200 cat.hash len_32.txt -o outfile.txt –force*.

#### Flag:
*qwertyuiopasdfghjklqwertyuiopasd*

---

### 15 - Found Your Website

#### Overview
An attacker used a port scanner and web fuzzer to scan the server, from which they found an URL path to a damn vulnerable web application. The task is to analyse a pcap file to determine the directory of the DVWA.

#### Procedure
Wireshark was used to load the pcap file, and the IP source of the fuzzing was found to be *5.8.16.237*. Then the following filter was applied: *ip.src == 5.16.237 and http*.  By scrolling through the history, the GET requests finished at a POST request for */secure/login* meaning that is where the attacker was able to log in.

#### Flag
*/secure*

---

### 20 - Unfair Game

#### Overview
This challenge involved remotely connecting to a terminal based game which required correctly guessing 100 randomly generated numbers in order to win, against a timer. An offline version of the game written in C was provided for analysis.

#### Procedure
By experimenting with the provided unfair_challenge.c it was discovered the random seed is determined by the time of execution. Future "random" numbers use that seed, so if this seed value is learned, the numbers can be found.

By default C returns an int while Python returns a float between 0 and 1, and I could not find a way to get Python to replicate the C output with the same seed. So, after some research I found Cython is able to import C functions for use in Python, and the following sources were used: https://riptutorial.com/cython & https://gist.github.com/joshlk/5b1a2c3a8d4bcf94476cafa33e611795.

I wrote a python program to use C functions to create a random seed based on program start time, then populated a list with the first 100 random numbers. Then I set up a program that would loop over entering an answer and reading the response, which worked and I was able to gain the flag.

#### Flag
*CTF{0i_y0v_w3rE_cHe4t!nG!_y0u_w3rEnT_sUpPo5eD_2_gVeS5_mY_nUmBeRs!1!!}*
