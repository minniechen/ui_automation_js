js使用JavaScript搭建UI自动化测试框架也非常流行，尤其是在前端开发中。通常会使用 WebDriverIO 或 Puppeteer 等工具来实现UI自动化测试。以下是使用 WebDriverIO 来搭建UI自动化框架的详细教程步骤。
一、环境搭建
1.安装Node.js
先安装 Node.js，确保能够使用npm（Node包管理器）。安装好后，使用以下命令检查版本：
npm -v

2.安装WebDriverIO
WebDriverIO 是一个支持多种浏览器自动化的JavaScript测试框架，常用于UI自动化。我们先创建一个项目目录并初始化npm：
cd ui-automation-js
npm init -y

3.安装WebDriverIO及依赖
使用以下命令安装WebDriverIO及相关依赖：
npm install @wdio/cli

4.运行WebDriverIO配置向导
WebDriverIO提供了一个初始化向导，可以帮助你生成配置文件。执行以下命令启动向导：
npx wdio config

你将被询问一系列问题，例如选择测试框架（推荐使用Mocha或Jasmine）、报告插件、服务（如Selenium Standalone或Chromedriver），以及浏览器类型（如Chrome或Firefox）。这些选项将自动生成一个wdio.conf.js文件。
二、项目结构设计
搭建UI自动化测试框架时，建议保持良好的目录结构，以便更好地组织代码。推荐的目录结构如下：
ui-automation-js/
├── tests/                # 测试用例
│   └── login.test.js     # 示例测试用例
├── pages/                # 页面对象模型（POM）
│   └── login.page.js     # 登录页面
├── wdio.conf.js          # WebDriverIO配置文件
└── package.json          # 项目依赖和脚本

三、编写页面对象模型（POM）
页面对象模型（POM）是一种设计模式，用于将页面元素和操作封装在一个类中，使测试脚本更加简洁和可维护。比如，登录页面的对象模型可以如下编写：
// pagesobjects/login.page.js
const { expect } = require('@wdio/globals')
const LoginPage = require('../pageobjects/login.page')
const SecurePage = require('../pageobjects/secure.page')

describe('My Login application', () => {
    it('should login with valid credentials', async () => {
        await LoginPage.open()

        await LoginPage.login('tomsmith', 'SuperSecretPassword!')
        await expect(SecurePage.flashAlert).toBeExisting()
        await expect(SecurePage.flashAlert).toHaveText(
            expect.stringContaining('You logged into a secure area!'))
    })
})

// pagesobjects/page.js
const { browser } = require('@wdio/globals')

/**
* main page object containing all methods, selectors and functionality
* that is shared across all page objects
*/
module.exports = class Page {
    /**
    * Opens a sub page of the page
    * @param path path of the sub page (e.g. /path/to/page.html)
    */
    open (path) {
        return browser.url(`https://the-internet.herokuapp.com/${path}`)
    }
}
// pagesobjects/secure.page.js
const { $ } = require('@wdio/globals')
const Page = require('./page');

/**
 * sub page containing specific selectors and methods for a specific page
 */
class SecurePage extends Page {
    /**
     * define selectors using getter methods
     */
    get flashAlert () {
        return $('#flash');
    }
}

module.exports = new SecurePage();

四、编写测试用例
1.创建测试用例
测试用例将调用页面对象中的方法来模拟用户操作。例如，我们可以编写一个简单的登录测试用例：
// tests/specs/test.e2e.js
const { expect } = require('@wdio/globals')
const LoginPage = require('../pageobjects/login.page')
const SecurePage = require('../pageobjects/secure.page')

describe('My Login application', () => {
    it('should login with valid credentials', async () => {
        await LoginPage.open()

        await LoginPage.login('tomsmith', 'SuperSecretPassword!')
        await expect(SecurePage.flashAlert).toBeExisting()
        await expect(SecurePage.flashAlert).toHaveText(
            expect.stringContaining('You logged into a secure area!'))
    })
})
1.运行测试
在package.json文件中添加以下脚本，以便运行测试：
"scripts": {
    "test": "npx wdio run wdio.conf.js"
}

npm test

五、生成测试报告
1.安装报告插件
WebDriverIO支持丰富的报告插件，可以生成HTML、JSON等格式的报告。我们可以使用allure-reporter生成用户友好的测试报告：
npm install @wdio/allure-reporter --save-dev

2.配置报告插件
在wdio.conf.js中，添加如下配置以启用Allure报告：
reporters: ['spec', ['allure', {
    outputDir: 'allure-results',
    disableWebdriverStepsReporting: true,
    disableWebdriverScreenshotsReporting: false,
}]],

3.生成HTML报告
运行测试后，可以通过以下命令生成HTML格式的报告：
npm install allure-commandline --save-dev
npx allure generate allure-results --clean
npx allure open

六、集成到持续集成（CI/CD）
1.Jenkins集成
在Jenkins中，你可以使用npm脚本来自动化运行测试。设置好Jenkins任务后，添加步骤执行以下命令：
npm install
npm test

2.GitLab CI/CD集成
如果使用GitLab，可以在.gitlab-ci.yml文件中添加以下配置：
stages:
  - test

test_ui:
  script:
    - npm install
    - npm test

七、优化
1.等待机制
在WebDriverIO中，可以使用waitUntil或内置的等待机制来确保元素加载完成后再进行操作。例如，等待某个元素可见：
await browser.waitUntil(
    async () => (await $('#element').isDisplayed()),
    {
        timeout: 5000,
        timeoutMsg: 'expected element to be visible after 5s'
    }
);

2.数据驱动测试
WebDriverIO支持数据驱动测试，可以在测试中使用不同的数据集。例如，通过forEach方法循环执行：
const loginData = [
    { username: 'user1', password: 'pass1' },
    { username: 'user2', password: 'pass2' }
];

loginData.forEach(data => {
    it(`should login with username ${data.username}`, async () => {
        await LoginPage.login(data.username, data.password);
        expect(await browser.getTitle()).toContain('Welcome');
    });
});

八、练习项目
github: https://github.com/minniechen/ui_automation_js/tree/master
git@github.com:minniechen/ui_automation_js.git
branch: master
九、总结
通过JavaScript和WebDriverIO搭建UI自动化框架，可以轻松实现Web应用程序的自动化测试。WebDriverIO拥有丰富的插件和服务支持，可以生成报告、与持续集成平台无缝集成，同时其灵活的API也能够帮助开发者编写可维护、高效的自动化测试代码。
你可以根据项目需求，选择合适的插件与工具，进一步优化和扩展框架。
