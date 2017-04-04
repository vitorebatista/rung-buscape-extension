## Rung ─ Buscapé extension

This is a demo extension to Rung showing how to be alerted when a product is cheaper according to Buscapé's site.

### Full source

```js
const { create } = require('rung-sdk');
const { Money, String: Text } = require('rung-sdk/dist/types');
const Bluebird = require('bluebird');
const agent = require('superagent');
const promisifyAgent = require('superagent-promise');
const { map, pipe, prop } = require('ramda');

const request = promisifyAgent(agent, Bluebird);

const token = '<<<<YOUR TOKEN HERE>>>>';
const sourceId = '<<<<YOUR SOURCE ID HERE>>>>';
const server = `http://sandbox.buscape.com.br/service/findProductList/lomadee/${token}/BR/?sourceId=${sourceId}&app-token=${token}&format=json&program=lomadee`;
//Se more in http://developer.buscape.com.br/portal/lomadee/api-de-ofertas/recursos#lista-de-produtos

function createAlert({ productshortname, pricemin }) {
    return `${productshortname} no preço mínimo de R$ ${pricemin}`
}

function main(context, done) {
    const { item, value } = context.params;

    return request.get(`${server}&keyword=${item}&priceMax=${value}`)
        .then(({ body }) => {
            const products = body.product;
            const alerts = map(pipe(prop('product'), createAlert), products);
            done(alerts);
        })
        .catch(() => done([]));
}

const params = {
    item: {
        description: 'Informe o produto que você está procurando (Ex: TV):',
        type: Text,
        default: 'TV'
    },
    value: {
        description: 'O valor do produto deve ser inferior a:',
        type: Money,
        default: 500
    }
};

const app = create(main, { params });

module.exports = app;
```

### Test and develop

- Clone the project and `cd` to its directory
- Install the dependencies: `npm install` or `yarn`
- If you don't have `rung-cli`, install it globally by running `sudo npm install -g rung-cli`
- Modify the source providing your token and source id
- Run `rung run` to start the _Query Wizard_ via _CLI_

You'll get this screen and the result:

![](http://i.imgur.com/Z3A9uBh.gif)

Your result will be an array containing all the alerts that would be generated.