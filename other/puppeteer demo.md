* 1\. 使用puppeteer访问url，并把将网页快照出来，生成图片
* 2\. 代码
  ```javascript

  const puppeteer = require("puppeteer");
  // const fs = require("fs").promises;

  function run(url) {
    return new Promise(async (resolve, reject) => {
      try {
        const browser = await puppeteer.launch({
          headless: true,
          dumpio: true,
          devtools: false,
          args: ["--window-size=1920,4000"],
          defaultViewport: null,
        });
        const page = await browser.newPage();
        // await page.setRequestInterception(true);
        // let str = JSON.stringify(reportData);
        // browser.on("targetchanged", async (target) => {
        //   const targetPage = await target.page();
        //   const client = await targetPage.target().createCDPSession();
        //   await client.send("Runtime.evaluate", {
        //     expression: `localStorage.setItem('reportData', ${str})`,
        //   });
        //   console.log("reportData set");
        // });

        page
          .on("console", (message) =>
            console.log(
              `${message.type().substr(0, 3).toUpperCase()} ${message.text()}`
            )
          )
          .on("pageerror", ({ message }) => console.log(message))
          // .on("response", (response) =>
          //   console.log(`${response.status()} ${response.url()}`)
          // )
          .on("requestfailed", (request) =>
            console.log(`${request.failure().errorText} ${request.url()}`)
          );

        // console.info("setting cookies");

        // await page.setCookie(...cookies);

        console.info(`goto ${url}`);
        await page.goto(url, { waitUntil: "networkidle0", timeout: 0 });

        console.log(page.url());

        await page.screenshot({ path: "example.png" });
        browser.close();
        return resolve("success");
      } catch (e) {
        return reject(e);
      }
    });
  }
  run(
    url
  )
    .then(console.log)
    .catch(console.error);


  ``` 