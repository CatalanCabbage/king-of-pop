# The Royals Quiz (king-of-pop) CTF
A CTF to demonstrate Jackson's JSON deserialization vulnerability (CVE-2017-7525).  

# Setup
- Run the server. [_server readme_](https://github.com/CatalanCabbage/king-of-pop-server#to-run-in--2-minute-)
```bash
git clone https://github.com/CatalanCabbage/king-of-pop-server.git
cd king-of-pop-server/package
#Extract the package.zip file present in this folder
java -jar target/king-of-pop-1.0-SNAPSHOT.jar
```
- Run the client.
```bash
git clone https://github.com/CatalanCabbage/king-of-pop.git
cd king-of-pop
npm install
npm run dev
```
- Access the UI on port 5000. Eg: `http://localhost:5000`

# Sample screenshots  
_Question screen, poulated by the GET API_  
![image](https://user-images.githubusercontent.com/45961072/140027066-7eeb1c63-1d69-4441-b8db-cb681046016f.png)  


_Answer screen, poulated by the POST API_  
![image](https://user-images.githubusercontent.com/45961072/140027079-a2dc7e5e-22a8-4d27-8267-25434db9dc3c.png)  



# Walkthrough

## Hints given
1. Want hint? "Always read documentation." I have no further comments.

2. Music is a way of life, and I'm a huge fan of Michael Jackson. I'd probably even change my name to Jackson(Mr.J?), and my sons would be called J's sons. Would be so cool. So when in doubt, Mr.J is what you need.  
_Beat it!_

3. I love feedback. You can give any feedback. Letter of thanks, cash money, new TV, fan mail, pringles - just send it over, I won't object.

4. Don't copy StackOverflow code. But I say dat: if u do, @ least u have some1 2 blame. Agree?

5. Before music, my life was a mess, I'd just watch tv-serials all day long.  
Now I work while jamming to Jackson5, de-toxifying and de-tv-serializing my life.

6. This quiz is unbeetable. Unsolved from 2017. Maybe many information lostin abstractTransletion. English my third languege.


## Solution
### Just looking.
A quick look at the quiz indicates that there are just two screens. One that shows options, one that shows the right answer.  
On the network tab, no interesting calls are made; one `GET`, and one `POST`.  
A [GitHub Gist](https://gist.github.com/CatalanCabbage/381acb9f6819b9cb62f5a1807beea003#what-is) is linked at the bottom of the screen.  

### What's the goal?
The goal is to get the "photo of boss". But who is the "boss"?  
View Gist revisions, or view the Gist raw, and information is hidden as comments in the .md file - the "boss" is the "king of Pop" - Michael Jackson.  

**So the goal is to get the image of the King of Pop, Michael Jackson.**

<details>
    <summary>Pointers 🛈</summary>

1. For the goal, *from the Gist:*
> Many easy question are asked first, then in last *main boss level* question is asked.  
> When boss level question is answered corectly, **user see photo of boss and show me screenshot to get prize.**  

2. To look at the comments, Hint #1:  
> Want hint? "Always read **documentation**." I have no further **comments**.

3. From the comment: 
> 最终关卡的问题是“谁是流行音乐之王”, which translates to 'The question at the final level is "Who is the king of pop music?"'

</details> 

### The easy solution
We need to get the image of the "King of Pop".  
We know the `POST` API accepts a `questionID` and returns the `image`.   
So sending a request with `"questionID" : "king_of_pop"` should return the image of the King of Pop, and we're done!  

Except, it doesn't. .-.  
A `king_of_default` error is returned, stating that "king_of_pop is not implemented yet".  

This is in line with this excerpt from the Gist:  
> Boss level question not implemented still.  
> Everything finished and super boss photo selected but code pending.

We can conclude that **the image for king_of_pop is present, but somehow the mapping hasn't been done**, which is why the request sends the default image although the image is in fact present on the server.

<details>
    <summary>Pointers 🛈</summary>

`king_of_pop` : The format of `questionID` is given in the readme, printed in the console, and can also be checked from the `POST` calls from the UI.
</details> 

### Understanding the application
Now that the server doesn't give us the image we ask for, we need to take it. And to do that, let's map out how the application runs.  

The Gist file gives us most of the information:  
Client is Svelte, which is inconsequential at this point since that's basically vanilla.  
Server is Java, there are 2 APIs in total which use JSON to communicate.  
The DB's implementation is a props file which contains mappings to the file-system.  


|Component|Comments|
|:---:|:---:|
|Client code|Svelte|
|Server code|Java|
|Client-server communication|2 APIs|
|Communication format|JSON|
|Image hosting|A file-system|
|Database|A props file|

<details>
    <summary>Pointers 🛈</summary>

1. What's the DB implementation? *From the Gist:*
> store key-values in a properties file, where value is the image path.  
</details>

### What's the goal? v2
Froom the previous sections:  
- *"So the goal is to get the image of the King of Pop, Michael Jackson."*  
- *"We can conclude that the image for king_of_pop is present, but somehow the mapping hasn't been done"*  
- *"The DB's implementation is a props file which contains mappings to the file-system, where value is the image path"*

From our newfound knowledge of the "DB":  
Since we have no info about how the file-system is set up, our best bet is to manipulate the props file to get the image.  

**Adding a new DB mapping, `king_of_pop = king_of_pop.jpg` and then querying `king_of_pop` should give us the image!**

### Quick walk through OWASP top Vulns
_This section is not actually needed, but a top 10 refresher doesn't hurt. :)_  
Our goal is to get to the `king_of_pop.jpg` image on the server.  
What kind of exploit can we leverage to get there, which of the top 10 are applicable here?

Reference: [OWASP Top 10 2021](https://cheatsheetseries.owasp.org/IndexTopTen.html)

|#|OWASP Vulnerability|What is it?|Remarks|Is it applicable?|
|:---:|:---:|:---:|:---:|:---:|
|1|Broken Access Control|Authorization bugs, IDOR, etc.|No roles/user accounts involved here|❌|
|2|Cryptographic Failures|Passwords, encryption|No passwords involved here|❌|
|3|Injection|User-supplied paths, commands, SQLi|No classic DB or commands supplied here|❌|
|4|Insecure Design|Logical sec issues|Maybe? But only 2 APIs which seem to be fine, so unlikely|❔|
|5|Security Misconfiguration|Default accounts, unnecessary functionality, etc|Only 2 APIs exposed, no account management|❌|
|6|Vulnerable and Outdated Components|Components running vulnerable versions|Maybe? We can revisit the components|❔|
|7|Identification and Authentication Failures|Session management, password handling flows, etc|No account management|❌|
|8|Software and Data Integrity Failures|Unsigned components, **insecure deserialization**|Communication format is JSON, might be applicable|❔|
|9|Security Logging and Monitoring Failures|No/inadequate logging|Not applicable here|❌|
|10|Server-Side Request Forgery|Improper calls made from internal components|No such internal components in this setup|❌|

### What options do we have?  
From the previous section, possible applicable vulns are:
- Insecure design
- Software and Data Integrity Failures
- Vulnerable and Outdated Components

**Insecure design:** Ignore this for now since the app isn't very complex, fuzzing APIs doesn't give us anything either.  
**Software and Data Integrity Failures:** Insecure deserialization! The application uses JSON, so this might be applicable.  
Also, this usually leads to an RCE, which we could use to modify the DB.    
**Vulnerable and Outdated Components:** Maybe, but what components does it use?  

Insecure deserialization - a quick search shows us the most popular Java libraries that handle JSON: Jackson, GSON, JSON.simple, JSON-P.  
Everything in this app screams Jackson.  
**So the vulnerable/outdated component in this app could be Jackson and the vulnerability, Jackson JSON deserialization.**  
*(spoiler: it is.)*


<details>
    <summary>Pointers 🛈</summary>

1. Why look for *Java* JSON libraries? *From the Gist:*  
> **The Java server** take request and give response.

2. Pointer indicating that one parameter takes Objects, *from the Gist:*  
> feedback: 'this is amazing question thank u xoxo' //Can send thanks **message or happy selfie or cash money no objection**

3. Pointer indicating that one parameter takes Objects, Hint #3:  
> I love **feedback**. You can give any feedback. Letter of thanks, cash money, new TV, fan mail, pringles - just send it over, I won't **object**.

4. Pointer indicating Jackson JSON vulnerability, Hint #2:  
> Music is a way of life, and I'm a huge fan of Michael Jackson. I'd probably even change my name to **Jackson**(Mr.J?), and my sons would be called **J's sons**. Would be so cool. So when in doubt, **Mr.J is what you need**.  
> _Beat it!_

5. Pointer indicating Jackson JSON vulnerability, Hint #5:
> Before music, my life was a mess, I'd just watch tv-serials all day long.  
> Now I work while jamming to **Jackson5**, de-toxifying and **de-tv-serializing** my life.

</details>

### Finding the Gadget
It's Jackson's JSON deserialization. But there are many CVEs, which gadget do we use?  
Generally, CVE-2017-7525 shows up on the top-5 results; and doing a Ctrl+F on the gist for "gadget" gives us "templates implementation".  
The gadget from CVE-2017-7525 was: `com.sun.org.apache.xalan.internal.xsltc.trax`**.TemplatesImpl** via `com.sun.org.apache.xalan.internal.xsltc.runtime`**.AbstractTranslet**  
Hint #6 gives us a nudge in this direction too.  

The JSON body format for that exploit looks like this ([_reference_](https://adamcaudill.com/2017/10/04/exploiting-jackson-rce-cve-2017-7525/#building-an-exploit)):  
```javascript
{
    'question': 'doesnt_matter',
    'option': 'doesnt_matter_either'
    'feedback':[ 'com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl',
        {
            'transletBytecodes' : [ '<base64 encoded classFile>' ],
            'transletName' : 'a.b',
            'outputProperties' : {}
        }
    ]
}
```

<details>
    <summary>Pointers 🛈</summary>

1. Gadget hint, *from the Gist:*  
> **templates implementation** and design will be downfall of billion doller quiz, just like Samseng who build extra **gadget**

2. Indicating the CVE, Hint #6:  
> This quiz is unbeetable. Unsolved from **2017**. Maybe many information lostin **abstractTransletion**. English my third languege.

</details>


### "Putting it all together"
To summarize:  
- We need to modify the DB (props file) to add a new mapping to `king_of_pop.jpg`  
- We have a Jackson Deserialization RCE  

We can use the RCE to add the new mapping.  
But what's the name of the props file, where is it located on the disk?  
From the StackOverflow link in the Gist comments, the name of the file is `property.dat`; we can just copy-paste the StackOverflow code and modify the values.  
*Ref: [StackOverflow link](https://stackoverflow.com/questions/22370051/how-to-write-values-in-a-properties-file-through-java-code/22370284#22370284)*

So the exploit would look something like:  
```java
FileOutputStream fr = new FileOutputStream(new File("property.dat")); //Write to this file
Properties p = new Properties();
p.setProperty("king_of_pop", "king_of_pop.jpg"); //Set our property
//p.setProperty("king_of_default", "king_of_pop.jpg"); //If we map this instead, the image will be shown in the UI itself after exploitation
p.store(fr, "Properties");
fr.close();
```

Steps to exploit this:  
- Craft a Java file(say `Exploit.java`) with the props-modifying payload. [_View sample Exploit.java_](https://github.com/CatalanCabbage/king-of-pop-server/blob/master/exploit/Exploit.java#L1)
- Compile `Exploit.java` and convert that classFile to base64
- Add the base64 class as a param to the JSON body. [_ref: Body template in finding the Gadget_](#finding-the-gadget)
- Send it! [_View sample exploit JSON body_](https://github.com/CatalanCabbage/king-of-pop-server/blob/master/attackscripts/attack.json#L1)
- Query the inserted key to get the image 

_UI after the exploit:_  
![image](https://user-images.githubusercontent.com/45961072/140646629-421a8ea1-3752-4ff2-a80d-e38f75efcc2c.png)


<details>
    <summary>Pointers 🛈</summary>

1. StackOverflow link, *from the Gist:*  
> Anyway I not the kind to cry upon spilt milk or fix unbroken milk, so [I do it his way anyway] (https://stackoverflow.com/a/22370284)

2. Indicating StackOverflow copy-pasta and the `.dat` file, Hint #4:  
> Don't copy StackOverflow code. But I say **dat**: if u do, @ least u have some1 2 blame. Agree?

</details>

---

# Appendix, recommended reading
- What is deserialization? [OWASP Deserialization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html#what-is-deserialization) 
- Jackson serialization-deserialization [Jackson Polymorphic types](https://programmerbruce.blogspot.com/2011/05/deserialize-json-with-jackson-into.html)
- Exploiting the Jackson RCE, quick overview: [Adam Caudill's blog post](https://adamcaudill.com/2017/10/04/exploiting-jackson-rce-cve-2017-7525/)
- Sample exploit for this CTF: [Exploit.java for this CTF](https://github.com/CatalanCabbage/king-of-pop-server/blob/master/exploit/Exploit.java#L1)
- Sample payload for this CTF: [JSON body to be sent](https://github.com/CatalanCabbage/king-of-pop-server/blob/master/attackscripts/attack.json#L1)
- Sample exploit: [Jackspoilt Github repo](https://github.com/conikeec/jackspoilt)

# Improvements
> Could have done these to improve the experience. But hindsight is 20/20!  

Since attempting different CVEs is time-consuming:  
- Return a different error message when RCE happens successfully to indicate that it works  
- Add hint #6 to the gist 

