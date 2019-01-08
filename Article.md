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

* [Jest](https://jestjs.io/): Is fully featured testing framework which is developed by Facebook. It needs very little configuration and works basically out of the box. 

* [Puppeteer](https://pptr.dev/): A Node.js library created by Google, which provides a convenient API to control Headless Chrome. 

And the last thing we will need is [jest-puppeteer preset](https://github.com/smooth-code/jest-puppeteer), which will allow us to combine two frameworks mentioned above. 

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

Let's run puppeteer for the first time on it's own, so you will actually see how it works alone without Jest in the first place. In the root of the project you will find `puppeteer_firts_try.js` file which contains some basic instruction for Puppeteer.
Run it with:
```
node puppeteer_firts_try.js
```

* in `puppeteer_firts_try.js`
```javascript {.line-numbers}
const puppeteer = require('puppeteer');

(async function main(){
    try {
        const browser = await puppeteer.launch({ headless: false });
        const page = await browser.newPage();
        await page.goto('http://localhost:8080', {waitUntil: 'domcontentloaded'});
        
        await new Promise(resolve =>  setTimeout(resolve, 5000));
        console.log('done');
        await browser.close();
    } catch (e) {
        console.log('Err', e);
    }
})();
```

With `await puppeteer.launch({ headless: false });` on line 5 we are launching new Chromium instance, in options we specify `headless: false`, which means that browser won't run in the headless mode(basically without graphical user interface). On the next line we open new page and then on the line 7 we navigate to http://localhost:8080. `{waitUntil: 'domcontentloaded'}` option on line 7 specify that our code will wait till DOM content is loaded. Line 9 just makes app to stop for 5 seconds, so you can observe it. And on line 11 we close the browser.

##Integrating Puppeteer with Jest
Now we will integrate Puppeteer with Jest. But why is this neccessary in the first place? We do this, because Puppeteer on itself is not testing framework, it is tool which allow us to control Headless Chrome. So in order to make our work easier, we combine it with Jest, which provides great testing utilities.

#####Jest Configuration
* create jest.config.js file in the root of the project and paste this code in
```javascript {.line-numbers}
module.exports = {
    preset: "jest-puppeteer",
    globals: {
      URL: "http://localhost:8080"
    },
    testMatch: [
      "**/test/**/*.test.js"
    ],
    verbose: true
}
```

On line 2 we specify `jest-puppeteer` preset, which will allow us to use Puppeteer with Jest. In `globals` we declare variables, which will be available in our whole test suite. And in `testMatch` we are only saying in which folder and for which files Jest should look for. 

#####Configuration for jest-puppeteer preset
* create jest-puppeteer.config.js file in the root of the project and use this code
```javascript {.line-numbers}
module.exports = {
    launch: {
        headless: process.env.HEADLESS !== 'false',
        slowMo: process.env.SLOWMO ? process.env.SLOWMO : 0,
    }
}
```
Here in `lauch` object we can specify options for Chromium instance, which will be launched before our test suite and accessible to all test files. So you can specify all the options, which you would normally pass here `puppeteer.launch({ /*your options...*/ });`. Here you can find [all available options](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#puppeteerlaunchoptions). So on line 3 we are specifying if Puppeteer should lunch Browser in `headless` mode or not. And on line 4 we are doing the same, but now with `slowMo` (slow motion).

## Writing Our Tests

#####Testing Frontend

With everything set up we can finally start writing our tests. Let's start with something simple:
* in `src/test/` create file named frontend.test.js and write following code in it
```javascript {.line-numbers}
const timeout = process.env.SLOWMO ? 70000 : 20000;

beforeAll(async () => {
    await page.goto(URL, {waitUntil: 'domcontentloaded'});
});

describe('Test header and title of the page', () => {
    test('Title of the page', async () => {
        const title = await page.title();
        expect(title).toBe('E2E Puppeteer Testing');
        
    }, timeout);
});
```

Now you can run:
```
npm run test
```
![test-result]()

Let's analyze this code line by line. On first line we are setting `timeout` variable, which we are later using to specify timeout for our tests(note that we specify this timeout in milliseconds). So as you can see if we are running Puppeteer in slowMo we increase our timeout from 20000 ms to 70000 ms. This ensure that our tests won't timeout. 
On line 3 we use `beforeAll`, this function will run some code before all tests in our file are executed. As you can see we pass to this function an async callback in which we navigate to `URL` which we specified earlier as global variable. But from where we took `page`? `page` is actually exposed to each test file in our test suite thanks to jest-puppeteer preset.
On line 7 we are using `describe` which allow us to group tests together.
And then we write our actual test. This test is rather simple on line 9 we get page title and then we use Jest built in assertion library [expect](https://jestjs.io/docs/en/expect) to test if we got correct title.

Now let's and another test to this file. Paste this code right under the first test in our describe block.
```javascript {.line-numbers}
test('Header of the page', async () => {
        const h1Handle = await page.$('h1');
        const html = await page.evaluate(h1Handle => h1Handle.innerHTML, h1Handle);

        expect(html).toBe("E2E Testing with  <span class=\"pupp\">Puppeteer</span> and <span class=\"jest\">Jest</span>");
    }, timeout);
```

