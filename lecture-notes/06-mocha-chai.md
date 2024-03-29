# 06: Node.js Mocha & Chai

## Mocha + Chai = delicious testing

**Mocha** is a feature-rich JavaScript test framework running on Node.js and in the browser. It works hand-in-hand with **Chai** which is an assertion framework. You need **Chai** to assert things (true or false), and **Mocha** to run the tests.

To get started we first need to install **Mocha** as a dependency:

```bash
$ npm i mocha
```

Then install **Chai**:

```bash
$ npm i chai
```

Then we'll create a new folder in the root called `test` and put a file inside called `test.js`.

Inside `test.js` add the following example code:

```js
import { assert } from "chai";
describe('Array', function () {
  describe('#indexOf()', function () {
    it('should return -1 when the value is not present', function () {
      assert.equal([1, 2, 3].indexOf(4), -1);
    });
  });
});
```

Finally, in `package.json` add (or edit) your "test" script: `"test" : "mocha"`.

Now, in the terminal, you should be able to run this command:

```bash
$ npm test
```

And see this sort of output:

```bash
  Array
    #indexOf()
      ✔ should return -1 when the value is not present


  1 passing (7ms)
```

### Testing API routes

To test your API routes, we also need to install another dependency that will make the HTTP calls for us: **chai-http**

```bash
$ npm i chai-http
```

Now, before we begin writing our API tests, we need to refactor some code... specifically, we want to reuse some of the seed functions we wrote last time. Inside the `prisma` folder, create a new folder called `seed`. Inside this folder, create a file called `institutions.js`. Cut and paste the two functions `createInstitutions` and `deleteInstitutions` into this new file, along with the `PrismaClient` lines, and the line that loads the institution data (note: you'll have to change the path to the data file slightly, as we are inside another folder now).

Finally, at the bottom of `seed/institutions.js`, export the two functions:

```js
export {
  createInstitutions,
  deleteInstitutions
}
```

Back in `seed.js`, simply fill where the functions *used* to be with:

```js
import { 
    createInstitutions, 
    deleteInstitutions 
} from "./seed/institutions.js"
```

Try running your seed commands to test that everything still works:

```bash
$ npm run institutions:create
```

Replace everything in `test.js` with the following code, and **rename** the file to `institutions.test.js`:

```js
import chai from "chai"
import chaiHttp from "chai-http"
import app from "../app.js";

const { expect } = chai
chai.use(chaiHttp)

import { 
    createInstitutions, 
    deleteInstitutions 
} from "../prisma/seed/institutions.js";

describe('api', () => {
  describe('GET /api/institutions', () => {    
    it('should return a message that no institutions are found', async () => {
        await deleteInstitutions()
        chai.request(app).get("/api/institutions").end((_, res) => {
            expect(res.body.msg).to.be.equal("No institutions found")
            expect(res.status).to.be.equal(200)
        })             
    })
  })

  describe('GET /api/institutions', () => {    
    it('should return an array of institutions', async () => {
        await createInstitutions()        
        chai.request(app).get("/api/institutions").end((_, res) => {
            expect(res.body.data).to.be.a("array")
            expect(res.body.data).to.have.lengthOf.above(0)
            expect(res.status).to.be.equal(200)
        })             
    })
  })
})
```

One more change to `app.js`, add this line at the very bottom:

```js
export default app
```

This allows us to import app into our test files so we can get at the routes that have been defined.

Run your tests using:

```bash
$ npm test
```

### before and after

You can define functions that will fire `before` and `after` your tests, for example, creating all your test data before, and then deleting all your test data after. You can add these before the first `describe` and then also at the very end of the test file, e.g.:

```js
before(async function() {
    await createInstitutions()
});

...

after(async function() {
    await deleteInstitutions()
});
```

### A testing db

You should ideally create a new db for your tests, separate from your dev and production databases. Copy and paste your `dev.db` file and rename it `test.db`. Now in your `schema.prisma` file you can keep a new line commented out, to toggle between your databases:

```js
datasource db {
  provider = "sqlite"
  url      = "file:./dev.db"
  //url      = "file:./test.db"
}
```
