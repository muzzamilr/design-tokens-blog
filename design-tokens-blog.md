In this article, we will be discussing the Design Tokens and their Integration with the newly released version 5 of Ant Design.

Design Tokens are generally all the values needed to construct and maintain a UI. Design tokens can include color, typography, spacing etc. For example if there is color green specificully used by Starbucks, their designers will tokenize the hex value of that color e.g starbucks_green. Now they can use this token in the whole UI design. In future if Starbucks want to change the shade of their color green or even completely replace the color green, designer can just change the value of color starbucks_green token and it will reflect to the whole UI design.

Disclaimer: This hands-on guide is based on a paid product and requires beginner to intermediate skill set in Ant Design and React.

Ant Design v5 has shifted from less variables to token system. Now they have a ConfigProvider component which can be paired with design tokens for styling purpose. They have launced a new paid product named [Ant Design System for Figma](https://www.antforfigma.com/). This product will be mainly used for generating and exporting tokens from Figma and integrating those tokens with Ant Design.

Here's a list of the concepts that will be covered in this blog:
- Setting up the integration mechanism of tokens with Figma using GitHub Actions
- Transforming tokens into format required by Ant Design
- Using transformed tokens in code.
- Overriding the styles provided by tokens.
- Understanding pros, cons and best practices to follow

You can find the complete code at https://github.com/muzzamilr/design-tokens-workshop

## Let's code!
We will be using Vite for the React project setup, by using this command:
```sh
npm create vite
```
this command will prompt some options and you need to name the project, choose React as library and JavaScript as language. You can test if the project is setup is correct by running command:
```sh
npm start dev
```

## Dependencies
After creating the project, we need to install dependencies, which includes,
- Ant Design
- Figma Tokens Transformer

using this command:
```sh
npm install antd figma-tokens-transformer
```

## Figma Token Transformer Config
Figma tokens transformer is a package which helps to transform the tokens exported from figma into the format specified by Ant Design.
### tokens-transformer.config.json
```json
{
  "version": 5,
  "source": {
    "tokensFile": "src/tokens/tokens.json"
  },
  "target": {
    "jsonsDir": "src/ant-tokens"
  }
}
```
This file lives in the root of project and we need to specify the version of Ant Design we are using and the source of tokens file exported by figma and the target directory in which the transformed json files will be generated.

### tokens-transformer.secret.json
This file will be created in the root of the project and it includes the licence key and email of Ant For Figma. 

** Don't commit this file.
```json
{
  "license": {
    "key": "Key",
    "email": "email"
  }
}
```
After completing this step configuration for figma token transformer is complete

## GitHub Workflows
Disclaimer: This step can be skipped because you can handle the tokens update manually.

We need to create Github actions for automating the workflow, whenever the designer exports from figma to GitHub, these actions will transform and push the the new transformed tokens in the branch. Before all this make sure you setup git in your project using this command:
```sh
git init
```
### .github/workflows/transform.yml
```yaml
name: Tranform Tokens

on:
  push:
    paths:
      - 'src/tokens/**'
    branches:
      - master

jobs:
  transform_tokens:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          token: ${{ secrets.COMMIT_TOKEN }}
      - name: Setup Node 18
        uses: actions/setup-node@v3
        with:
          node-version: '18.x'
      - run: npm ci
      - run: LICENSE_KEY=${{ secrets.LICENSE_KEY }} LICENSE_EMAIL=${{ secrets.LICENSE_EMAIL }} npx figma-tokens-transformer transform

      - name: Commit & Push changes
        run: |
          git config --global user.name <username>
          git config --global user.email "email@users.noreply.github.com"
          git add .
          git commit -m "Generated new tokens"
          git push
```
In this piece of code we are creating a job named tranform_tokens which runs on Ubuntu and sets up node v18. It installs the dependencies "npm ci" and then transformation command runs. After transformation it commits and pushes code to the branch.

You will need to setup the secrects for this repository in the settings of that repository, for the secrets setup you can follow this guide https://docs.github.com/en/actions/security-guides/encrypted-secrets

## Tokens Directory Setup
After setting up the GitHub actions, we need to create "tokens" directory in "src" directory and add the tokens.json file exported from the Figma Tokens Plugin.

Create a new directory in "src" directory named "ant-tokens"

After creating these directories and adding tokens.json file in the src/tokens directory we can run the transform command which will generate the light and dark json files in the target directory.

Run:
```sh
npx npx figma-tokens-transformer transform
```
After running this command you can see the json files created in the target directory "ant-tokens"
```
└── /src
    ├── /ant-tokens
        ├── light.json
        ├── dark.json
    ├── /tokens
        ├── tokens.json
    ├── App.jsx
    ├── button.jsx
    ├── child.jsx
    ├── main.jsx
    ├── styles.css
```

## Components Setup
Now we will use different components of Ant Design for testing the tokens in ConFigProvider
### src/App.jsx
```ts
import {
	Button,
	Select,
	Radio,
	Checkbox,
	Input,
	ConfigProvider,
	Space,
	Layout,
	Switch,
	Calendar,
} from "antd";
import { useState } from "react";
import * as lightTheme from "./ant-tokens/light.json";
import * as darkTheme from "./ant-tokens/dark.json";
import { ButtonMod } from "./button";
function App() {
	const [dark, setDark] = useState(false);
	const [startDate, setStartDate] = useState(new Date());
	const handleChecked = (checked) => {
		if (checked) setDark(true);
		else setDark(false);
	};
	return (
		<ConfigProvider
			theme={{
				token: dark ? darkTheme : lightTheme,
			}}
		>
			<Layout
				style={{
					display: "flex",
					alignItems: "center",
					justifyContent: "center",
					height: "100vh",
				}}
			>
				<Space direction="vertical" size={10}>
					<Switch
						checkedChildren="Light"
						unCheckedChildren="Dark"
						onChange={handleChecked}
					/>
					<Button>Button</Button>
					<ButtonMod />
					<Checkbox>Checkbox</Checkbox>
					<Radio>Radio</Radio>
					<Select
						placeholder="Select"
						options={[{ label: "Select", value: "Option" }]}
					/>
					<Calendar fullscreen={false} style={{ width: "300px" }} />
					<Input
						prefix={
							<MsgSvg
								color={dark ? darkTheme.colorPrimary : lightTheme.colorPrimary}
							/>
						}
						placeholder={"Input"}
					/>
				</Space>
			</Layout>
		</ConfigProvider>
	);
}
const MsgSvg = ({ color }) => (
	<svg
		width="16"
		height="16"
		viewBox="0 0 16 16"
		fill="none"
		xmlns="http://www.w3.org/2000/svg"
	>
		<path
			d="M11.9512 0C13.024 0 14.056 0.471111 14.8152 1.31644C15.5752 2.16 16 3.29778 16 4.48889V11.5111C16 13.9911 14.184 16 11.9512 16H4.048C1.8152 16 0 13.9911 0 11.5111V4.48889C0 2.00889 1.8072 0 4.048 0H11.9512ZM12.856 4.62222C12.688 4.61244 12.528 4.67556 12.4072 4.8L8.8 8C8.336 8.42756 7.6712 8.42756 7.2 8L3.6 4.8C3.3512 4.59556 3.0072 4.62222 2.8 4.86222C2.584 5.10222 2.56 5.48444 2.7432 5.75111L2.848 5.86667L6.488 9.02222C6.936 9.41333 7.4792 9.62667 8.048 9.62667C8.6152 9.62667 9.168 9.41333 9.6152 9.02222L13.224 5.81333L13.288 5.74222C13.4792 5.48444 13.4792 5.11111 13.2792 4.85333C13.168 4.72089 13.0152 4.64 12.856 4.62222Z"
			fill={color}
		/>
	</svg>
);
export default App;
```
In the App.jsx file we have used different components like Button, Select, Radio, Checkbox, Input, Switch, Calender and ConfigProvider for setting up the tokens.

We are using Switch Component for switching between dark and light mode. We have imported dark and light json files.

Wrapped the whole App component in the ConfigProvider and passed dark and light as tokens in the theme prop.

There is also a MsgSvg component used in the Input component as Prefix to change the color of according to theme we are reading the value of colorPrimary from dark and light json files.

** Ant Design provides useToken hook for accessing all the tokens provided in the ConfigProvider. Here we are not using that hook because of Top to Bottom data flow of React that hook is not usable in this component but will be availaible in the child components e.g ButtonMod

### src/button.jsx
```ts
import { Button, ConfigProvider, theme } from "antd";
import { Child } from "./child";
const { useToken } = theme;
export const ButtonMod = () => {
  const { token } = useToken();
  return (
    <ConfigProvider
      theme={{
        inherit: false,
        components: {
          Button: {
            // borderRadius: token.borderRadiusSM,
            // colorPrimaryHover: token.colorPrimaryHover,
          },
        },
      }}
    >
      <Button
        size="middle"
        style={{
          borderRadius: token.borderRadiusXS,
          color: token.colorPrimary,
        }}
      >
        Button
      </Button>
      <Child />
    </ConfigProvider>
  );
};
```
In this code we have use ConfigProvider and passed false to the inherit property of theme prop. Now it will not inherit the theme and tokens from previous ConfigProvider, we can also style a specific compnents by providing the type of component in components property of theme.

All tokens are overridable, for example if we are using black as primary color in the parent ConfigPovider, we can override it by wrapping in another ConfigProvider and passing values to it. If the inherit is true (true by default) then it will inherit all the previous properties and only override the provided properties in the child component.

Here we are using useTokens hook and these token values are inherited from the parent ConfigProvider even if the child ConfigProvider inherit is false.
We can also use tokens in in-line styles.

### src/child.jsx
```ts
import { Button } from "antd";

export const Child = () => {
  return <Button>Child</Button>;
};
```
In this code, we are just returning an Ant button component. As you remember we passed false to inherit in its parent component and no token are defined in its parent (ButtonMod) component. So, this component will have the default styles of Ant Design.

Even if we try to call use useToken hook in this component, the token values provided by the hook will be the default values of Ant Design.

## Pros
- Streamlined process by combining figma and github actions
- We have a lot more control over the UI including Fonts, Font Sizes, Colors, Border Radius and much more
- We can make our own custom variants of AntD components (inherit false and define self configuration)
- We can use tokens directly in inline styles of a components

## Cons
- Styling components of a custom library/package
- useToken hook will work for the child components only
- Not able to control the sizes of different components e.g we cannot set the size of button

## Best Practices
- Do not use global css! (it’s gonna hurt you later)

## Resources
- [Ant Design - Theme Customization](https://ant.design/docs/react/customize-theme)
- [Ant Design - ConfigProvider](https://ant.design/components/config-provider)
- [Ant Design - Theme Editor](https://ant.design/theme-editor)
- [Ant For Figma](https://ant.design/theme-editor)
- [Figma Tokens Transformer](https://github.com/Jonarzz/figma-tokens-transformer)
- [Tokens Studio for Figma](https://docs.tokens.studio/)
