#End-to-End testing with Puppeteer and Jest 

In this article I will cover basic introduction to testing and then dive deeper into End-to-End testing using Jest and Puppeteer, which are very popular javascript  frameworks.  

Requirements for this tutorial are: 

1. Basic knowledge of javascript 

2. You know how to use CLI 

3. You are familiar with Node.js and NPM 

4. Also some knowledge of git may be handy 

So let me first say few words about testing in general. Testing is crucial part of every software development process. It prevents regression (bugs occurring frequently in our code) and overall speed up development process. There are basically three main types of tests:  

* Unit testing – With unit tests we are testing small isolated pieces of our code. They are easiest one to create and maintain. 

* Integration testing – In this type of testing we combine and test individual units and test them as a group. 

* End-to-End(E2E) testing –  Is defined as defined as the testing of complete functionality of some application. 

Throughout this tutorial we will be focusing on E2E testing as the title suggests. We will be writing our tests with two powerful tools, which are Jest and Pupeeteer: 

* Jest: Is fully featured testing framework which is developed by Facebook. It needs very little configuration and works basically out of the box. 

* Puppeteer: A Node.js library created by Google, which provides a convenient API to control Headless Chrome. 

And the last thing we will need is jest-puppeteer preset, which will allow us to combine two frameworks mentioned above. 

So what you will learn in the end ?  

1. How to Integrate Puppeteer with Jest 

2. Testing forms 

3. Testing frontend in general 

4. Taking screenshot 

5. Emulating mobile device 

6. Intercepting requests 

7. How to target new tab opened in the headless browser 

8. And some tricks and tips on how to improve your tests 

##Project Setup

First you will need download or clone project which I prepared [GitHub Starter Project](https://github.com/Zovi343/E2E_Testing_with_Puppeteer_Final). If you don't prefere to code along, you can donload already finished project [GitHub Final Project](https://github.com/Zovi343/E2E_Testing_with_Puppeteer_Final).

####After donloading Starter Project:

* cd into the repository
```
cd <your_path>/E2E_Testing_with_Puppeteer_Starter
```

* install dependencies
```
npm install
```

* run the project
```
npm run dev-server
```

Great now our application is running on http://localhost:8080. Next thing to do do is to install all the neccessary tools.

```
npm install puppeteer jest jest-puppeteer
```

Also we will need to additionaly install jest-cli globaly in order to be able to run only one test separetly from others.

```
npm install -g jest-cli
```

##First Look at Puppeteer

Let's run puppeter for the first time on it's own, so you will actually see how it works alone without Jest in the first place. In the root of the project you will find `puppeteer_firts_try.js` file which contains some basic instruction for Puppeteer.
Run it with:
```
node puppeteer_firts_try.js
```

* in `puppeteer_firts_try.js`
```javascript
const puppeteer = require('puppeteer');

(async function main(){
    try {
        const browser = await puppeteer.launch({ headless: false });
        const page = await browser.newPage();
        await page.goto('http://localhost:8080/');
        
        await new Promise(resolve =>  setTimeout(resolve, 5000));
        console.log('done');
        await browser.close();
    } catch (e) {
        console.log('Err', e);
    }
})();
```





 

 

 

 