# End-to-End testovanie s Puppeteer and Jest 

V tomto článku sa vám v rýchlosti predstavím základné typy testov a následne vás bližšie prevediem End-to-End testovaní, na ktoré budeme používať Jest a Puppeteer, čo sú veľmi populárne Javascriptové frameworky využívané na testovanie.

Prerekvizity pre tento tutorial sú:  

1. Základná znalosť Javascriptu

2. Viete používať CLI 

3. NPM a Node.js vám nie sú cudzie

4. A taktiež vedomosti o HTML a CSS sa vám určite zídu 

Predtým ako začneme by som rád povedal zopár slov o testovaní vo všeobecnosti. Je nevyhnutnou súčasťou procesu vyvýjania softwaru. Môže drasticky znížiť cenu vášho projektu a zvýšiť produktivitu vášho vývojárskeho tímu. Existujú rôzne druhy testov, tu ich však rozdelíme iba na tieto tri hlavné typy:  

* [Unit testy](https://smartbear.com/learn/automated-testing/what-is-unit-testing/) – S unit testami testujeme malé izolované časti nášho kódu. 

* [Integračné testy](https://smartbear.com/learn/automated-testing/what-is-integration-testing/) – Je spôsob testovania, v ktorom kombinujeme jednotlivé časti nášho kódu, ktoré potom testujeme spoločne ako jednu ucelenú skupinu.

* [End-to-End(E2E) testovanie](https://smartbear.com/learn/automated-testing/end-to-end-testing/) – Je definované ako testovanie kompletnej funkcionality našej aplikácie.

Počas tohto tutoriálu sa budem sústreďovať na E2E testovanie ako napovedá aj názov tohto článku. Budeme písať testy pomocou dvoch výborných nástrojov, ktorými sú Jest a Puppeteer: 

* [Jest](https://jestjs.io/): Je plne vybavený testovací framework, ktorý je vyvýjaný Facebookom. Nepotrebuje v podstate žiadnu konfiguráciu, čiže funguje takmer ihneď po inštalácii.

* [Puppeteer](https://pptr.dev/): Knižnica  pre Node.js vytvorená Googlom, ktorá poskytuje praktickú API, pomocou ktorej môžeme ovládať [Headless Chrome](https://developers.google.com/web/updates/2017/04/headless-chrome). 

A posledná vec, ktorú budeme potrebovať, je [jest-puppeteer preset](https://github.com/smooth-code/jest-puppeteer), ktorý nám umožní skobinovať tieto dva frameworky. 

Takže čo sa vlastne dnes v závere naučíte ?  

1. Ako integrovať Puppeteer a Jest 

2. Testovanie formulárov

3. Testovanie frontendu

4. Ako urobiť screenshot 

5. Emulovanie mobilných zariadení

6. Ako zachytiť requesty

7. Ako targetovať novo otvorenú stránku v headless prehliadači

8. A ako debugovať vaše testy

## Project Setup

Najprv si budete potrebovať stiahnuť projekt, ktorý som pripravil [GitHub Starter Project](https://github.com/Zovi343/E2E_Testing_with_Puppeteer_Starter.git). Ak nemáte zaújem programovať počas tutorialu môžete si stiahnuť finálny projekt [GitHub Final Project](https://github.com/Zovi343/E2E_Testing_with_Puppeteer_Final).

#### Po stiahnutí projektu:

* cd do repozitára
```
cd <your_path>/E2E_Testing_with_Puppeteer_Starter
```

* install dependencies
```
npm install
```

* rozbehnite projekt
```
npm run dev-server
```

Výborne teraz naša aplikácia beží na http://localhost:8080. Po otvorení by ste mali vidieť niečo takéto: 

![app-image](https://raw.githubusercontent.com/Zovi343/E2E_Testing_with_Puppeteer_Article/master/img/app.png) 

Ďalšia vec, ktorú budeme potrebovať spraviť, je nainštalovať všetky nevyhnutné nástroje.

```
npm install puppeteer jest jest-puppeteer
```

Taktiež budeme potrebovať nainštalovať  [jest-cli](https://jestjs.io/docs/en/cli) aby sme mohli spúšťať jeden test separátne od ostatných. 

```
npm install -g jest-cli
```

## Prvý pohľad na Puppeteer

Najprv spustíme Puppeteer samostatne, aby ste mohli vidieť ako funguje bez Jest. V root directory projektu nájdete `puppeteer_firts_try.js` súbor, ktorý obsahuje základné inštrukcie pre Puppeteer. 

Do terminálu zadajte:
```
node puppeteer_firts_try.js
```

* v `puppeteer_firts_try.js`
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
Ako možete už pravdepodobne vidieť Puppeteer spolieha výhradne na promises, takže ho budeme vždy používať s async/await. 
S [puppeteer.launch()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-puppeteerlaunchoptions) na riadku 5 spúšťame novú inštanciu [Chromia](https://www.chromium.org/), v argumentoch špecifikujeme `headless: false`, čo znamená, že prehliadač sa nespustí v headless móde ( v podstate bez grafického používateľského rozhrania). Na ďalšiom riadku otvárame novú stránku a potom na riadku 7 navigujeme na http://localhost:8080. `waitUntil: 'domcontentloaded'`  argument na riadku 7 špecifikuje, že náš kód bude čakať až pokiaľ DOM content nebude načítaný. Riadok 9 len spôsobí, že aplikácia sa zastaví na 5 sekúnd, takže bude môcť vidieť, čo sa deje. A na riadku 11 zatvárame prehliadač. 

## Integrácia Puppeteer s Jest
Teraz zintegrujeme Puppeteer s Jest. Ale prečo to vlastne potrebujeme ? Robíme to, preto že Pupeteer sám o sebe nie je testovací framework, je to nástroj, ktorý nám umožňuje kontrolovať [Headless Chrome](https://developers.google.com/web/updates/2017/04/headless-chrome). Takže aby sme si uľahčili našu prácu, skombinujeme Puppeteer s Jest, ktorý nám poskytuje veľmi užitočné testovacie utilities. 

#### Jest Konfigurácia
* vytvorte `jest.config.js` súbor v root directory projektu a vložte tam tento kód :
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

Na riadku 2 špecifikujeme `jest-puppeteer` preset, ktorý nám umožní použiť Jest s Pupeteer. V `globals`deklarujeme premenné, ktoré budú dostupné vo všetkých testoch. A v  `testMatch` jednoducho hovoríme, v ktorých zložkách má Jest hľadať súborý.  

#### Konfigurácia pre jest-puppeteer preset
* vytvorte `jest-puppeteer.config.js` súbor v root directory nášho projektu a použite tento kód :
```javascript {.line-numbers}
module.exports = {
    launch: {
        headless: process.env.HEADLESS !== 'false',
        slowMo: process.env.SLOWMO ? process.env.SLOWMO : 0,
    }
}
```
V `launch`  objekte špecifikujeme argumenty pre inštanciu [Chromia](https://www.chromium.org/), ktorá bude spustená predtým ako pobežia naše testy a bude dostupná vo všetkých našich testovacích súboroch, takže tu môžete zadať všetky argumenty, ktoré by ste normálne dali do [puppeteer.launch()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#puppeteerlaunchoptions). Na riadku 3 špecifikujeme, či by mal Puppeteer spustiť prehliadač v  `headless` móde alebo nie. A na riadku 4 mu hovoríme aby bežal v `slowMo`, čo spomalý Puppeteer o milisekundy, ktoré špecifikujeme, takže budeme môcť pozorovať čo vlastne robí. Obidve možnosti, ktoré tu definujeme sú skvelé pre debugovanie. 

## Písanie Našich testov

### Testing Frontend
Keďže máme naše testovacie prostredie pripravené, môžeme začať s písaním prvých testov. Bude lepšie ak začneme s niečím jednoduchým.

* v `src/test/` nájdete súbor s menom `frontend.test.js` , do ktorého budete potrebovať vložiť tento kód :
```javascript {.line-numbers}
const timeout = process.env.SLOWMO ? 30000 : 10000;

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

A teraz môžete zadať toto do vášho terminálu:
```
npm run test
```
A mali by ste vidieť niečo takéto::

![test-result-image](https://raw.githubusercontent.com/Zovi343/E2E_Testing_with_Puppeteer_Article/master/img/test_result.png)

Poďme zanalyzovať tento kód po jednotlivých riadkoch. Na prvom riadku nastavujeme `timeout` premennú, ktorú neskôr používame na to aby sme špecifikovali timeout pre naše testy (nezabudnite, že špecifikujeme tento timeout v milisekundách). Ako môžete vidieť, ak Puppeteer beží v slowMo, zvýšime náš timeout z 10000 ms na 30000ms. Toto zaistí, že naše testy nezlyhajú kvôli timeoutu. Na riadku 3 používame [beforeAll](https://jestjs.io/docs/en/api#beforeallfn-timeout), táto funkcia vykoná nejaký kód predtým ako pobežia všetky testy v tomto súbore. Tejto funkcii dávame ako parameter async callback, v ktoróm navigujeme na `URL` , ktorú sme  špecifikovali predtým ako globálnu premennú. Ale odkiaľ sme zobrali `page`  premennú? Tá je dostupná tiež vo všetkých testovacích súboroch vďaka `jest-puppeteer` preset. Na riadku 7 používame [describe](https://jestjs.io/docs/en/setup-teardown#order-of-execution-of-describe-and-test-blocks) , ktoré nám umožňuje zoskupovať testy, a v ňom už potom píšeme naše samotné testy. Tento test je celkom jednoduchý, na riadku 9 získavame title stránky a potom používame assertion knižnicu [expect](https://jestjs.io/docs/en/expect), ktorá je vstavaná v [Jest](https://jestjs.io/), na to aby sme overili, či sme dostali správny výsledok. 


Skúsme teraz pridať ďalší test do tochto súboru. Vložte tento kód hneď pod náš prvý test v describe bloku:
```javascript {.line-numbers}
test('Header of the page', async () => {
        const h1Handle = await page.$('.learn_header');
        const html = await page.evaluate(h1Handle => h1Handle.innerHTML, h1Handle);

        expect(html).toBe("What will you learn");
    }, timeout);
```

Na druhom riadku použivame [page.$()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pageselector) funkciu, kotrá nám umožňuje vybrať HTML element pomocou normálneho CSS selektoru, a nakoniec nám táto funkcia  vráti [ElementHandle](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-class-elementhandle) , ktorú neskôr použijeme nato, aby sme získali innerHTML tohto elementu. Na riadku 3 potom používame [page.evaluate()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pageevaluatepagefunction-args) ,ktorá vyhodnotí funkciu v kontexte stránky, a tým pádom získame prístup k innerHTML naše [ElementHandle](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-class-elementhandle).

### Form Tests
Teraz keď už máme nejaké základné testy za nami, skúsime napísať test pre jednoduchý formulár, ktorý som pre nás pripravil.

![form-image](https://raw.githubusercontent.com/Zovi343/E2E_Testing_with_Puppeteer_Article/master/img/form.png)

* premenujte `form.test.js.example` v `src/test` na `form.test.js`a vložte tento kód do describe bloku, ktorý tam už je:
```javascript {.line-numbers}
    test('Submit form with valid data', async () => {
        await page.click('[href="/login"]');
        
        await page.waitForSelector('form');
        await page.type('#name', 'Rick');

        await page.type('#password','szechuanSauce');
        await page.type('#repeat_password','szechuanSauce');

        await page.click('[type="submit"]');
        await page.waitForSelector('.success');
        const html = await page.$eval('.success', el => el.innerHTML);

        expect(html).toBe('Successfully signed up!');
    }, timeout);
```

Prvá vec ktorú tu robíme, je že klikneme na Login link v navigácii. Používame na to [page.click()]() funkciu, ktorá berie jeden argument CSS selektor. Keďže sme navigovali na inú URL používame [page.waitForSelector()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pagewaitforselectorselector-options), na to aby sme počkali, kým DOM zobrazí náš formulár, takže budeme s ním môcť interagovať. Potom používame [page.type()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pagetypeselector-text-options) metódu, aby sme vyplnil náš formulár táto metóda berie dva argumenty CSS selektor a text, ktorý chceme napísať. Potom klikáme na submit button a čakáme kým sa objaví správa o úspešnom podaní formulára, ktorú získame pomocou [page.$eval()](https://pptr.dev/#?product=Puppeteer&version=v1.12.2&show=api-pageevalselector-pagefunction-args-1). 

Ak teraz zadáte `npm run test`, mali by ste vidieť tri úspešne testy .

### Branie Screenshotov na Mobilných a Desktopových zariadeniach
Formulár a frontend je otestovaný, takže môžeme obrátiť našu pozornosť na branie screenshotov a emulovanie mobilných zariadení.

* premenujte `screenshots.test.js.example` v `src/test` na `screenshots.test.js` a vložte tento kód do describe bloku, ktorý tam už je:
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
V tomto kóde najprv nastavíme viewport našej stránky s [page.setViewport()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pagesetviewportviewport)  a potom spravíme screenshot s funkciou [page.screenshot()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pagescreenshotoptions), ktorej poskytujeme nejaké argumenty, aby sme špecifikovali kde a v akom formáte uložiť screenshot.  

Po tom čo zadáte `npm run test` mali by ste byť schopný nájsť obrázok s názvom home.jpg v  `test/screenshots` zložke.

Teraz skúsme spraviť screenshot, počas toho ako emulujeme  mobilné zariadenie.
Najprv pridajte tento kód na vrch nášho súboru: 
```javascript {.line-numbers}
const devices = require('puppeteer/DeviceDescriptors');
```
A potom pridajte tento test do describe bloku: 
```javascript {.line-numbers}
test('Emulate Mobile Device And take screenshot', async () => {
    await page.goto(`${URL}/login`, {waitUntil: 'domcontentloaded'})
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

Tento kód je podobný tomu predchádzajúcemu, rozdiel je v tom, že teraz importujeme zariadenia z `puppeteer/DeviceDescriptors`. Potom vyberáme IPhone X na riadku 3 z objektu devices. Na ďalšiom riadku emulujeme toto zariadenie s [page.emulate()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pageemulateoptions). A potom jednoducho urobíme screenshot presne tým istým spôsobom ako v predchádzajúcom teste. 

### Ako zachytiť request a targetovanie novo otvorených stránok
Teraz sa pozrieme na trochu pokročilejšie vlastnosti, ktoré Puppeteer poskytuje, ako je napríklad zachytávanie requestov. A taktiež vám ukážem ako targetovať novo otvorenú stránku v headless prehliadači.

* premenujte `general.test.js.example` v `src/test` na `general.test.js` a skopírujte tam tento kód:
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
        await page.reload({waitUntil: 'networkidle0'});
        // await jestPuppeteer.debug();
        await page.setRequestInterception(false);
    }, timeout);
```

Tu na prvom riadku nastavujeme [page.setRequestInterception()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pagesetrequestinterceptionvalue) na `true`, čo nám umožňuje zachytiť každý odchádzajúci request. Na riadkoch 3-9 hovoríme aby Puppeteer prerušil každú odchádzajúci request, ktorý konči s `'.png'`. Takže vďaka tomuto kódu sa na stránke nenačíta obrázok, ktorý je na domovskej stránke našej aplikácie, vlastne sa toto stane, až po tom čo znovu načítame stránku, pretože obrázok už bol načítaný, predtým ako sme nastavili zachytávanie requestov. Potom na riadku 10 znovu načítame stránku s [page.reload()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pagereloadoptions), takže budeme môcť vidieť, že sa obrázok nezobrazuje. Ale ako vlastne, keďže Puppeteer testy sú tak rýchle ? Na to využijeme kód na ďalšiom riadku, ktorý je momentálne zakomentovaný, ale k tomuto sa vrátim až neskôr v skecii o debugovaní. A na riadku 12 nastavujeme [page.setRequestInterception()](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-pagesetrequestinterceptionvalue) na `false`, čo je veľmi dôležité! Pretože kebyže to neurobíme tak by zachytávanie requestov ostalo zapnuté po zvyšok nášho testovania a to môže spôsobiť veľa problémov, a byť veľmi ťažké na debugovanie. 

Pridajme teraz náš posledný test, s ktorým vám ukážem ako môžete targetovať novo otvorené stránky v headless prehliadači.

* pridajte tento test do describe bloku:
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

Na riadku 2 vytvárame nový Promise, v ktorom počúvame s  [browser.on('targetcreated')](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-event-targetcreated) ,či nový target (`page`) je vytvorený. Znova máme prístup k globálnej premennej `browser`,  vďaka [jest-puppeteer](https://github.com/smooth-code/jest-puppeteer) preset. Potom klikáme na link na našej domovskej stránke, ktorý otvára novú stránku, konkrétne: [GitHub Starter Project](https://github.com/Zovi343/E2E_Testing_with_Puppeteer_Starter). Na 7 riadku očakávame Promise, ktorý sme vytvorili na riadku 2 a tento Promise vracia novo otvorenú stránku. Takže v závere sme schopný získať title stránky a urobiť naše assertions. 


## Debugovanie vašich testov
Veľakrát budú vaše testy zlyhávať a často je veľmi ťažké takéto testy debugovať, a zistiť čo sa vlastne deje len z terminálu. To je hlavný dôvod, prečo vám chcem ukázať rôzne metódy, pomocou ktorý môžete debugovať vaše testy.

#### Headless a SlowMo argumenty
Takže pre debugovanie budete chcieť spustiť Puppeteer v headless móde a taktiež ho spomaliť, takže budete schopný vidieť, čo sa vlastne deje. Keďže sme nastavili tieto dve možnosti v `jest-puppeteer.config.js` , všetko čo teraz musíme spraviť je poskytnúť dve enviromentálne premenné, keď spúšťete testy z terminálu. 

Do terminálu zadajte:
```
HEADLESS="false" SLOWMO=100 npm run test
```

#### Running single Test Seperately
Niekedy potrebujete rozbehnúť iba jeden test bez toho aby sa spúšťali ďalšie. Aby sme mohli toto urobiť použijeme [jest-cli](https://jestjs.io/docs/en/cli), ktorú sme nainštalovali na začiatku. Vráťme sa teraz naspäť k zachycovaní reqestov , pretože to sme predtým neboli celkom schopný vidieť a pozorovať. 

Do terminálu zadajte: 
```
HEADLESS="false" SLOWMO=100 jest -t 'Intercept Request'
```

Síce sme už teraz mohli vidieť, že sa obrázok nezobrazil, presne ako sme predpokladali, avšak bolo to stále celkom rýchle, nie ? Poďme to napraviť s `jestPuppeteer.debug()`.

#### jestPuppeteer.debug()
[jestPuppeteer.debug()](https://github.com/smooth-code/jest-puppeteer/blob/master/README.md#put-in-debug-mode) zastaví naše testy, aby sme mohli zistiť, čo sa deje v prehliadači. Aby ste znovu spustili testy musíte stlačiť Enter. Takže teraz môžete odkomentovať riadok 11 z testu o zachytávani requestov a zadať predchádzajúci príkaz do terminálu. A budete môcť jasne vidieť, že obrázok nie je zobrazený na domovskej stránke, pretože request preň bola zachytená a prerušená. 

## Bonus Puppeteer Recorder
Nakoniec by som vám rád odporučil jednu Chrome extension, ktorá sa vám môže zísť, keď píšete testy s Puppeteer. Volá sa [Puppeteer Recorder](https://chrome.google.com/webstore/detail/puppeteer-recorder/djeegiggegleadkkbgopoonhjimgehda?hl=en) a umožňuje vám náhravať vaše interakcie s prehliadačom, a následne vygenerovať z toho Puppeteer script. 

## Záver
V tomto článku sme sa zaoberali dvomi veľmi populárnymi frameworkami Jest a Puppeteer. A naučili sme sa, že keď tie dva nástroje skombinujeme, získame veľmi robustné testovacie prostredie. Pokryli sme toho veľa, naučili ste sa ako integrovať Puppeteer a Jest, ako písať testy pre rôzne príležitosti, ako debugovať vaše testy a mnoho viac. Ale môžem vás uistiť, že je ešte veľmi veľa toho, čo sme v tomto tutoriale nepokryli, takže ak máte záujem navštívte oficiálne dokumentácie jednotlivých nástrojov a naučte sa toho ešte oveľa viacej!
