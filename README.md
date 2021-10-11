Code for the royals-of-x ctf

# What is
this? This is ultimate ~king~ royals quiz.  
Many easy question are asked first, then in last main boss level question is asked.  
When boss level question is answered corectly, user see photo of boss and show me screenshot to get prize.  
<!--最终关卡的问题是“谁是流行音乐之王”。I think "papa john" can be option for extra funny.-->
>Boss level question not implemented still.   
>Everything finished and super boss photo selected but code pending. Hiring for new developer to complete game.
# Client
## UI
UI designed very well in svelte for maximum user happyness.  
Also best in world UX DX FX. 
## APIs
### Get Question API
```javascript
GET /question
```

Sample response:
```javascript
{
    question: 'king_of_rap', //Question ID
    //Option IDs are `name`s
    options: [
        {name: 'Daddy Yankie'},
        {name: 'Eminem'},
        {name: 'Slim Shady'},
        {name: 'BTS'}
    ]
}
```

### Submit Answer API
```javascript
POST /answer
```

Sample request:
```javascript
{
    question: 'king_of_rap', //Question ID
    option: 'BTS', //Option ID
    feedback: 'this is amazing game thank u xoxo' //Can send thanks message or happy selfie or money
}
```

Sample response:
```javascript
{
    image: '', //Image ID
    imageData: '', //Image data in encrypted base64 secret format
    message: '' //Tells if you got wrong answer or not the right answer
}
```

### Monitoring
Analysis statistics need to be done using Google Analytics.  
>Monitoring not implemented still. Hiring for new developer to complete analysis. Minimum 15 year experience.

# Server

## General overview
The Java server take request and give response.  
What response depend on what request.  
What else there to say?

## Interoperability
This game is very highly interoperatable. It can be play on Windows 7, Windows 8, Windows 10, Windows 11, mobile, Samsung movile, Apple tablet and many more.  
Very fun game, anybody play with anything.

## Scalability
For now there is only 10 question, and there is 10 images.  
But future there will **1000 questions and 1000 images and I will show ads and become $$$$.**  
So storing image on my PC willn't scale. I need to move image to Microsoft AWS or such.

>"We need to reduce tight dependencies. Instead of using image file names directly in code, store key-values in a properties file, where value is the image path.  
>Then refer to that key in java code, so that paths can be changed anytime by directly changing props file without touching java code."  

That is what last developer say, but I don't understand why.  
Because anyway he write `image_name=image_name.jpg` in props file, why not just use `"image_name" + ".jpg"`? He trying to charge extra for typing extra without use, he fired.  
<!--Anyway I not the kind to cry upon spilt milk or fix unbroken milk, so [I do it his way anyway](https://stackoverflow.com/a/22370284)-->
