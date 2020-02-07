# Random Code Snipits

## copy from in JavaScript

```js
const {Client} = require('pg');
const escape = require('pg-escape');

const client = new Client({
  connectionString: connectionString,
})
await client.connect();
const inputStream = this.s3.getObject(s3Params).createReadStream()
const stream = client.query(copyFrom(escape('COPY %I FROM STDIN  (header, format csv)', this.table)));

try {
  console.log('starting stream');
  await pipeline(inputStream, stream);
} finally {
  console.log('stream done')
  await client.end()
}
```
