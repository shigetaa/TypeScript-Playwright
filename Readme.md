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
import playwright, { devices } from 'playwright';

(async () => {
	// Setup
	const browser = await playwright.chromium.launch();
	const context = await browser.newContext({ ...devices['Desktop Chrome'], locale: 'ja-JP' });
	// https://github.com/microsoft/playwright/blob/main/packages/playwright-core/src/server/deviceDescriptorsSource.json
	const page = await context.newPage();
	await page.goto('https://github.com/login');

	console.log(await page.title())
	await page.locator('//*[@id="login_field"]').fill('user')
	await page.locator('//*[@id="password"]').fill('pass')
	await Promise.all([
		page.locator('//*[@type="submit"]').click(),
		//page.waitForNavigation(),
		page.waitForTimeout(4000)
	])
	await page.screenshot({ path: `github.png` })
	// login auth failure https://github.com/session
	// login auth success https://github.com/
	console.log(page.url())
	// Save signed-in state to `githubStorageState.json`.
	await page.context().storageState({ path: 'githubStorageState.json' })

	// Teardown
	await context.close();
	await browser.close();
})()
```
#### コードの実行
```bash
npx ts-node auth_form.ts
```
認証成功、github の場合は
URL が https://github.com/ にページ遷移します。
```
Sign in to GitHub · GitHub
https://github.com/
```
認証失敗、github の場合は
URL が https://github.com/session にページ遷移します。
```
Sign in to GitHub · GitHub
https://github.com/session
```
### フォーム認証の処理をするコードを実装 (認証成功時に作成した認証ファイルを利用して認証を実施する)
ここでは、前項で認証成功時に、
`await page.context().storageState({ path: 'githubStorageState.json' })`
に指定したファイルを利用して認証必要ページにアクセスしています。
```bash
vi auth_form_success.ts
```
```typescript
import playwright, { devices } from 'playwright';

(async () => {
	// Setup
	const browser = await playwright.chromium.launch();
	const context = await browser.newContext({ ...devices['Desktop Chrome'], locale: 'ja-JP', storageState: 'githubStorageState.json' });
	// https://github.com/microsoft/playwright/blob/main/packages/playwright-core/src/server/deviceDescriptorsSource.json
	const page = await context.newPage();
	await Promise.all([
		page.goto('https://github.com/'),
		//page.waitForNavigation(),
		page.waitForTimeout(4000)
	])

	console.log(await page.title())
	console.log(page.url())
	await page.screenshot({ path: `github_success.png` })

	// Teardown
	await context.close();
	await browser.close();
})()
```
#### コードの実行
```bash
npx ts-node auth_form_success.ts
```
```
GitHub
https://github.com/
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
認証情成功、ステータスコードが`200`を返します。
```
200
認証成功ページタイトル
```
認証失敗、ステータスコードが`401`を返します。
```
401
認証失敗ページタイトル
```