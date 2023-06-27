# Reading Large JSON File and sending it to ES

```Javascript
const fs = require('fs');
const ndjson = require('ndjson');
const { Client } = require('@elastic/elasticsearch');

const client = new Client({ node: 'http://localhost:9200' });

const INDEX_NAME = 'your_index_name';

async function createIndex() {
  await client.indices.create({
    index: INDEX_NAME,
    body: {
      mappings: {
        properties: {
          // Define your mappings here based on the structure of your objects
          // For example:
          // name: { type: 'text' },
          // age: { type: 'integer' },
        },
      },
    },
  });
}

function bulkIndex(data) {
  const body = data.flatMap(doc => [{ index: { _index: INDEX_NAME } }, doc]);

  return client.bulk({ refresh: true, body });
}

function readAndIndexFile(filepath) {
  return new Promise((resolve, reject) => {
    let buffer = [];
    const CHUNK_SIZE = 5000; // Adjust chunk size based on your specific use case

    fs.createReadStream(filepath)
      .pipe(ndjson.parse())
      .on('data', async (obj) => {
        buffer.push(obj);

        if (buffer.length >= CHUNK_SIZE) {
          this.pause();
          await bulkIndex(buffer);
          buffer = [];
          this.resume();
        }
      })
      .on('end', async () => {
        // Handle last chunk
        if (buffer.length) {
          await bulkIndex(buffer);
        }

        resolve();
      })
      .on('error', reject);
  });
}

async function main() {
  try {
    await createIndex();
    await readAndIndexFile('./yourfile.json');
    console.log('Indexing completed.');
  } catch (err) {
    console.error(err);
  }
}

main();
```
```Shell
npm install @elastic/elasticsearch ndjson
```
In the createIndex function, you should define the mappings that correspond to the structure of your JSON objects.

In the readAndIndexFile function, the stream is paused before indexing a chunk of data to Elasticsearch, and resumed after the indexing operation is finished. This backpressure mechanism prevents the buffer from overflowing when the data can be read faster than it can be processed.

Keep in mind that Elasticsearch's bulk API has a size limit. It's generally best to stay well below the 100MB default limit. A chunk size of a few thousand is usually good enough. But you might need to adjust it based on the size and complexity of your objects.
