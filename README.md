#End-to-End testing with Puppeteer and Jest 

In this article I will cover very briefly introduction to testing and then dive deeper into End-to-End testing using Jest and Puppeteer, which are very popular javascript frameworks used for testing.  

Requirements for this tutorial are: 

1. Basic knowledge of Javascript 

2. You know how to use CLI 

3. You are familiar with Node.js and NPM 

4. Also some knowledge of HTML and CSS may be handy 

So let me first say few words about testing in general. Testing is crucial part of every software development process. It prevents regression (bugs occurring frequently in our code) and overall speed up development process. There are basically three main types of tests:  

* Unit testing – With unit tests we are testing small isolated pieces of our code. They are easiest one to create and maintain. 

* Integration testing – In this type of testing we combine and test individual units and test them as a group. 

* End-to-End(E2E) testing –  Is defined as defined as the testing of complete functionality of some application. 

Throughout this tutorial we will be focusing on E2E testing as the title suggests. We will be writing our tests with two powerful tools, which are Jest and Pupeeteer: 

* [Jest](https://jestjs.io/): Is fully featured testing framework, which is developed by Facebook. It needs very little configuration and works basically out of the box. 

* [Puppeteer](https://pptr.dev/): A Node.js library created by Google, which provides a convenient API to control Headless Chrome. 

And the last thing we will need is [jest-puppeteer preset](https://github.com/smooth-code/jest-puppeteer), which will allow us to combine these two frameworks. 

So what you will learn in the end ?  

1. How to Integrate Puppeteer with Jest 

2. Testing forms 

3. Testing frontend in general 

4. Taking screenshot 

5. Emulating mobile device 

6. Intercepting requests 

7. How to target newly opened page in the headless browser 

8. And how to debug your tests and some tips and trick

##Project Setup

First you will need download or clone project which I prepared [GitHub Starter Project](https://github.com/Zovi343/E2E_Testing_with_Puppeteer_Final). If you don't prefere to code along, you can download already finished project [GitHub Final Project](https://github.com/Zovi343/E2E_Testing_with_Puppeteer_Final).

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

Great now our application is running on http://localhost:8080. After visiting it you should see something like this: 

![test-result](https://raw.githubusercontent.com/Zovi343/E2E_Testing_with_Puppeteer_Article/master/img/app.png) 
Next thing to do is to install all the necessary tools.

```
npm install puppeteer jest jest-puppeteer
```

Also we will need to additionally install [jest-cli](https://jestjs.io/docs/en/cli) globally in order to be able to run only one test separately from others.

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

With [puppeteer.launch()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-puppeteerlaunchoptions) on line 5 we are launching new Chromium instance, in options we specify `headless: false`, which means that browser won't run in the headless mode (basically without graphical user interface). On the next line we open new page and then on the line 7 we navigate to http://localhost:8080. `waitUntil: 'domcontentloaded'` option on line 7 specify that our code will wait till DOM content is loaded. Line 9 just makes app to stop for 5 seconds, so you can observe it. And on line 11 we close the browser.

##Integrating Puppeteer with Jest
Now we will integrate Puppeteer with Jest. But why is this necessary in the first place? We do this, because Puppeteer on itself isn't testing framework, it is tool which allow us to control Headless Chrome. So in order to make our work easier, we combine it with Jest, which provides great testing utilities.

####Jest Configuration
* create `jest.config.js` file in the root of the project and paste this code in:
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

####Configuration for jest-puppeteer preset
* create `jest-puppeteer.config.js` file in the root of the project and use this code:
```javascript {.line-numbers}
module.exports = {
    launch: {
        headless: process.env.HEADLESS !== 'false',
        slowMo: process.env.SLOWMO ? process.env.SLOWMO : 0,
    }
}
```
Here in `lauch` object we can specify options for Chromium instance, which will be launched before our test suite and which will be accessible to all our test files. So you can specify all the options, which you would normally pass here [puppeteer.launch()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#puppeteerlaunchoptions). So on line 3 we are specifying if Puppeteer should lunch Browser in `headless` mode or not. And on line 4 we are doing the same, but now with `slowMo`, which slows Puppeteer down by milliseconds that we specify. So we will be able to observe what it actually does, this option is great for debugging.

## Writing Our Tests

###Testing Frontend
With everything set up we can finally start writing our tests. Let's start with something simple. 

* in `src/test/` you will find file named `frontend.test.js` into which you need to write this code to:
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
And you should see something like this:

![test-result](https://raw.githubusercontent.com/Zovi343/E2E_Testing_with_Puppeteer_Article/master/img/test_result.png)

Let's analyze this code line by line. On first line we are setting `timeout` variable, which we are later using to specify timeout for our tests(note that we specify this timeout in milliseconds). So as you can see if we are running Puppeteer in slowMo we increase our timeout from 20000 ms to 70000 ms. This ensure that our tests won't timeout. On line 3 we use [beforeAll](https://jestjs.io/docs/en/api#beforeallfn-timeout), this function will run some code before all tests in our file are executed. As you can see we pass to this function an async callback in which we navigate to `URL` which we specified earlier as global variable. But from where we took `page` variable? `page` is actually exposed to each test file in our test suite thanks to jest-puppeteer preset. On line 7 we are using [describe](https://jestjs.io/docs/en/setup-teardown#order-of-execution-of-describe-and-test-blocks) which allow us to group tests together. And then we write our actual test. This test is rather simple on line 9 we get page title and then we use Jest built in assertion library [expect](https://jestjs.io/docs/en/expect) to test if we got correct title.


Now let's and another test to this file. Paste this code right under the first test in our describe block.
```javascript {.line-numbers}
test('Header of the page', async () => {
        const h1Handle = await page.$('.learn_header');
        const html = await page.evaluate(h1Handle => h1Handle.innerHTML, h1Handle);

        expect(html).toBe("What will you learn");
    }, timeout);
```

On line 2 we are using [page.$()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pageselector) which allow us to select HTML element by normal CSS selector. And it returns [ElementHandle](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-class-elementhandle) which we can later use to get innerHTML of this element. On line 3 we then use [page.evaluate()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pageevaluatepagefunction-args), which evaluates a function in the page context, to basically evaluate our handle in page context and that way get access to its innerHTML.

###Form Tests
With some basic tests already written we will now try to test one form which I have prepared for us. 

* rename `form.test.js.example` in `src/test` to `form.test.js` and paste this code in to describe block which is alredy coded there:
```javascript {.line-numbers}
    test('Submit form with valid data', async () => {
        await page.click('[href="/login"]');
        
        await page.waitForSelector('form');
        await page.type('#name', 'Rick');

        await page.type('#password','szechuanSauce');
        await page.type('#repeat_password','szechuanSauce');

        await page.click('[type="submit"]');
        await page.waitForSelector('.success');
        const html = await page.evaluate(() => {
            return document.querySelector('.success').innerHTML;
        });

        expect(html).toBe('Successfully signed up!');
    }, timeout);
```

So first thing we do in this snippet is that we click on Login link in navigation. We use [page.click()]() for this, which takes as an argument a CSS selector. Since we navigated to different URL we use [page.waitForSelector()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pagewaitforselectorselector-options) to wait till our form will be rendered by DOM, so we can start to do something with it. Then we use [page.type()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pagetypeselector-text-options) method to fill out our form, this method takes two argument a CSS selector and a text, which we want to type. We then proceed by submitting our form, waiting for success message to appear and getting its innerHTML.

If you now run `npm run test` you should have three passing tets.

###Taking Screenshots on Desktop and Mobile
With our form and frontend tested, we can turn our attention to screenshot taking and emulation of mobile devices in Puppeteer.

* rename `screenshots.test.js.example` in `src/test` to `screenshots.test.js` and paste this code in to describe block which is alredy coded there
```javascript {.line-numbers}
    test('Take screenshot of home page', async () => {
        await page.setViewport({ width: 1920, height: 1080 });
        await page.screenshot({
            path: './src/test/screenshots/home.jpg',
            fullpage: true,
            type: 'jpeg'
        });
    }, timeout);
```
In this block of code we first set viewport of the page with [page.setViewport()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pagesetviewportviewport) and then we take a screenshot with [page.screenshot()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pagescreenshotoptions) to which we provide some options in order to specify where to store our image, and in what format should we store it. 

After running this test you should be able to find an image called home.jpg in `test/screenshots` folder.

Now let's take screenshot while emulating mobile device.
First add this line of code at the top of the file: 
```javascript {.line-numbers}
const devices = require('puppeteer/DeviceDescriptors');
```
And then add this test into our describe block: 
```javascript {.line-numbers}
test('Emulate Mobile Device And take screenshot', async () => {
    await page.goto(`${URL}/not-found`, {waitUntil: 'domcontentloaded'})
    const iPhonex = devices['iPhone X'];
    await page.emulate(iPhonex);
    await page.setViewport({ width: 375, height: 812, isMobile: true});
    await page.screenshot({
        path: './src/test/screenshots/home-mobile.jpg',
        fullpage: true,
        type: 'jpeg'
    });
}, timeout);
```

This code is similar to the previous one. The difference that now we are requiring devices from `puppeteer/DeviceDescriptors` then we access iPhone X on line 3 from devices object. On the next line we emulate this device with [page.emulate()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pageemulateoptions). And then we simply take screenshot as in the previous code snippet.

###Request Interception and Targeting Newly Opened Page
So now we will look into some more advanced features as is request interception. And I will also cover how to target newly opened opened page in headless browser.

* rename `general.test.js.example` in `src/test` to `general.test.js` and paste there this code:
```javascript {.line-numbers}
   test('Intercept Request', async () => {
        await page.setRequestInterception(true);
        page.on('request', interceptedRequest => {
            if (interceptedRequest.url().endsWith('.png')) {
                interceptedRequest.abort();
            } else {
                interceptedRequest.continue();
            }
        });
        await page.goto(URL, {waitUntil: 'networkidle0'});
        // await jestPuppeteer.debug();
        await page.setRequestInterception(false);
    }, timeout);
```

Here on the first line we are [page.setRequestInterception()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pagesetrequestinterceptionvalue) to `true`, which enables us to intercept outgoing requests. On lines 3-9 we are telling to Puppeteer to abort every request, which ends with `'.png'`. So thanks to this our page won't be able to load the image, which is currently on the homepage, well at least after we reload page, because the image was loaded before we set request interception. Then on line 10 will reload our page with [page.reload()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pagereloadoptions), so we will be able to see that image is not displayed. But how actually? That's what commented code on line 11 is for, but I will come back to this in debugging section. And on line 12 we set [page.setRequestInterception()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pagesetrequestinterceptionvalue) to false, which is very IMPORTANT! Because if you don't set it to `false` request interception will be set for all tests, which comes after this one and that can cause you lot of problems!

Now let's add our last test, with which I will show you how can you target newly opened page in headless browser.

* add this test to our describe block:
```javascript {.line-numbers}
   test('Target newly opened page', async () => {
        const newPagePromise = new Promise(resolve => browser.once('targetcreated', target => resolve(target.page())));
        await page.click('.repo_link');

        const newPage = await newPagePromise;
        const title = await newPage.title();
        await newPage.close();

        expect(title).toBe('GitHub - Zovi343/E2E_Testing_with_Puppeteer_Final');
    }, timeout);
```

On line 2 we are creating new Promise in which we are listening with [browser.on('targetcreated')](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-event-targetcreated) if new target (page) was created. Again we are able to access `browser`, because it is expose to us thanks to jest-puppeteer preset. Then we click on the link on the homepage, which opens new tab and point to [GitHub Starter Project](https://github.com/Zovi343/E2E_Testing_with_Puppeteer_Final). On line 7 we await Promise, which we created on line 2 and this Promise returns newly opened page. So in the end we are able to get title of this newly opened page and make our assertions.


##Debugging Your Test
Many times you find yourself with lot of failing tests and it can be really hard to find out what is going on only from terminal. That's why I will here show some tools to debug your tests.

####Headless and SlowMo options
So for debugging you want to launch Puppeteer in headless mode and also in slow motion, so you will be able to actually see what is going on. Since we set these two options in `jest-puppeteer.config.js` all we need to do now is set two environment variables when running our tests from terminal.

Run:
```
HEADLESS="false" SLOWMO=100 npm run test
```

####Running Tests Seperately
Sometimes you need to run one tests separately from others in order to find bugs. To do this we will use [jest-cli](https://jestjs.io/docs/en/cli), which we installed at the beginning. Now let's come back to the request interception test, because we weren't really able to observe it.

Run: 
```
HEADLESS="false" SLOWMO=100 jest -t 'Intercept Request'
```

But there was still pretty fast, wasn't it? Let's fix this with `jestPuppeteer.debug()`.

####jestPuppeteer.debug()
[jestPuppeteer.debug()](https://github.com/smooth-code/jest-puppeteer/blob/master/README.md#put-in-debug-mode) suspends test execution and gives you time to see what is going on in the browser. In order to continue execution you just need to press Enter.
So now you can just uncomment line 11 from code for request interception and you will be clearly able to see that image is not displayed, because request for it was intercepted.

##Bonus Puppeteer Recorder
At the end I would like to suggest chrome extension, which may come really handy when you are writing tests with puppeteer. It is [Puppeteer Recorder](https://chrome.google.com/webstore/detail/puppeteer-recorder/djeegiggegleadkkbgopoonhjimgehda?hl=en), which allows you to record your browser interactions and generate a Puppeteer script.

##Conclusion
In this article we explored two very popular frameworks Jest and Puppeteer. And we learned that when we combine these two tools together we get very powerfull setup for our testing environment. We covered a lot, you learned how to integrate Puppeteer with Jest, how to write different test cases, how to debug your tests and more. But I can assure that there is still lot of things which we haven't covered, so If you feel like it dive into official documentation and laern even more!
