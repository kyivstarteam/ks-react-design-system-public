### Releases of the disign system are available in packages
https://github.com/kyivstarteam/ks-react-design-system-public/packages/

### Comprehensive installation guide
https://zeroheight.com/1d5a833eb/p/41f267--installation-guide

### Installing design system with GitHub regisrty

1. In GitHub navigate to https://github.com/settings/tokens (Settings > Developer settings > Personal access tokens) and you should see something like this:
![](https://i.stack.imgur.com/nJ9g7.png)
2. Click `Generate new token`
3. From the permissions select at least `read:packages`
![](https://i.stack.imgur.com/XkdBK.png)
4. Click Generate token and copy the token
5. Add the following to your local `.npmrc` file:
```
@kyivstarteam:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${TOKEN}
````
where `${TOKEN}` is the token you copied in step 4

**Note:** make sure that `.npmrc` file is added to `.gitignore`

6. Execuce following in your terminal:

```shell
npm install @kyivstarteam/ks-react-design-system-public
```
Or this if you use `Yarn`:
```shell
yarn add @kyivstarteam/ks-react-design-system-public
```
