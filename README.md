## Rung ─ Buscapé extension

This is a demo extension to Rung showing how to be alerted when a product is more cheaper.

### Full source

```js
const { create, run } = require('rung-sdk');
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
        });
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
app.run();

module.exports = app;
```

When you clone this repo and install the packages, you can do `node index.js` to start the _Query Wizard_ via _CLI_. We
integrate with a third-party API called `Buscapé`.

You'll get this screen and the result:

![](http://i.imgur.com/Z3A9uBh.gif)

If you get a valid value as output, an alert would be generated. Otherwise, nothing would happen.
