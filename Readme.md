# Playwright
Playwright は Microsoft が開発したブラウザ自動化ツールです。
Chromium, Firefox, WebKit に対応してます。
Playwright を TypeScript 動かします。

## プロジェクトを作成します。
```bash
mkdir playwright && cd playwright
npm init -y
npm install -D playwright
```
### Typescript設定
```bash
npm install -D typescript @types/node ts-node
npx tsc --init
```
### 簡単なコードを実装
```bash
vi main.ts
```
```typescript
import playwright, { devices } from 'playwright';

(async () => {
  // Setup
  const browser = await playwright.chromium.launch();
  const context = await browser.newContext(devices['iPhone 11']);
  const page = await context.newPage();

  await page.goto('https://www.yahoo.co.jp/')
  await page.screenshot({ path: `yahoo.png` })
  console.log(await page.title())
  // Teardown
  await context.close();
  await browser.close();
})()
```
#### コードの実行
```bash
npx ts-node main.ts
```
### リストの処理をするコードを実装
```bash
vi list.ts
```
```typescript
import playwright, { devices } from 'playwright';

(async () => {
	// Setup
	const browser = await playwright.chromium.launch();
	const context = await browser.newContext({ ...devices['Desktop Chrome'], locale: 'ja-JP' });
	// https://github.com/microsoft/playwright/blob/main/packages/playwright-core/src/server/deviceDescriptorsSource.json
	const page = await context.newPage();
	await page.goto('http://www.yahoo.co.jp/');
	//await page.screenshot({ path: `yahoo.png` })
	console.log(await page.title())
	// List
	const li = await page.locator('//*[@id="ToolList"]/ul/li')
	// Pattern 1
	const texts = await li.allTextContents()
	console.log(texts)
	/*
	// Pattern 2
	const count = await li.count()
	for (let i = 0; i < count; i++) {
		console.log(await li.nth(i).textContent())
	}
	*/
	/*
	// Pattern 3
	const texts = await li.evaluateAll(list => list.map(element => element.textContent))
	console.log(texts)
	*/
	// Teardown
	await context.close();
	await browser.close();
})()
```
#### 処理方法、パターン１
```typescript
// Pattern 1
const texts = await li.allTextContents()
console.log(texts)
```
#### 処理方法、パターン２
```typescript
// Pattern 2
const count = await li.count()
for (let i = 0; i < count; i++) {
	console.log(await li.nth(i).textContent())
}
```
#### 処理方法、パターン３
```typescript
// Pattern 3
const texts = await li.evaluateAll(list => list.map(element => element.textContent))
console.log(texts)
```
#### コードの実行
```bash
npx ts-node list.ts
```

### リストの詳細を処理するコードを実装
```bash
vi list_detail.ts
```
```typescript
import playwright, { devices } from 'playwright';

(async () => {
	// Setup
	const browser = await playwright.chromium.launch();
	const context = await browser.newContext({ ...devices['Desktop Chrome'], locale: 'ja-JP' });
	// https://github.com/microsoft/playwright/blob/main/packages/playwright-core/src/server/deviceDescriptorsSource.json
	const page = await context.newPage();
	await page.goto('http://www.yahoo.co.jp/');
	//await page.screenshot({ path: `yahoo.png` })
	console.log(await page.title())

	const li = await page.locator('//*[@id="tabpanelTopics1"]/div/div[1]/ul/li')

	// Pattern 2
	const count = await li.count()
	for (let i = 0; i < count; i++) {
		//console.log(await li.nth(i).textContent())
		console.log(await li.nth(i).locator('//article/a/div/div/h1/span').innerText())
		//console.log(await li.nth(i).locator('a').getAttribute('href'))
		console.log(await li.nth(i).locator('//article/a').getAttribute('href'))
	};

	// Teardown
	await context.close();
	await browser.close();
})()
```
#### コードの実行
```bash
npx ts-node list_detail.ts
```

### リストのリンクを処理するコードを実装
```bash
vi list_link.ts
```
```typescript
import playwright, { devices } from 'playwright';

(async () => {
	// Setup
	const browser = await playwright.chromium.launch();
	const context = await browser.newContext({ ...devices['Desktop Chrome'], locale: 'ja-JP' });
	// https://github.com/microsoft/playwright/blob/main/packages/playwright-core/src/server/deviceDescriptorsSource.json
	const page = await context.newPage();
	await page.goto('http://www.yahoo.co.jp/');
	await page.screenshot({ path: `yahoo.png` })
	console.log(await page.title())

	const li = await page.locator('//*[@id="tabpanelTopics1"]/div/div[1]/ul/li')

	// Pattern 2
	const count = await li.count()
	for (let i = 0; i < count; i++) {
		console.log(await li.nth(i).locator('//article/a/div/div/h1/span').innerText())
		console.log(await li.nth(i).locator('//article/a').getAttribute('href'))
		//await li.nth(i).locator('//article/a').click()
		await Promise.all([
			li.nth(i).locator('//article/a').click(),
			page.waitForNavigation(),
		])
		await page.screenshot({ path: `yahoo_${i}.png` })
		await page.goBack()
	};

	// Teardown
	await context.close();
	await browser.close();
})()
```
#### コードの実行
```bash
npx ts-node list_link.ts
```

### フォーム認証の処理をするコードを実装

```bash
vi auth_form.ts
```
```typescript

```
#### コードの実行
```bash
npx ts-node auth_form.ts
```

### ベーシック認証の処理をするコードを実装
```bash
vi auth_basic.ts
```
```typescript
import playwright, { devices } from 'playwright';

(async () => {
	// Setup
	const browser = await playwright.chromium.launch();
	const context = await browser.newContext({
		...devices['Desktop Chrome'],
		locale: 'ja-JP',
		httpCredentials: {
			username: 'user',
			password: 'pass'
		}
	});
	// https://github.com/microsoft/playwright/blob/main/packages/playwright-core/src/server/deviceDescriptorsSource.json
	// https://playwright.dev/docs/network#http-authentication
	const page = await context.newPage();
	const response = await page.goto('https://example.com')
	console.log(response?.status())
	console.log(await page.title())
	await page.screenshot({ path: `example.png` })

	// Teardown
	await context.close();
	await browser.close();
})()
```
#### コードの実行
```bash
npx ts-node auth_basic.ts
```
