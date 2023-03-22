```javascript
const request = require('request');
const fs = require('fs');
const archiver = require('archiver');
const FormData = require('form-data');

const apiKey = 'xxxxxx-token';
const outputDir = './collections';
const archiveName = 'collections.zip';

// Make sure the output directory exists
if (!fs.existsSync(outputDir)) {
  fs.mkdirSync(outputDir);
}

const options = {
  url: 'https://api.getpostman.com/collections',
  headers: {
    'X-Api-Key': apiKey
  },
  proxy: 'http://username:password@localhost:8080'
};

request(options, function (error, response, body) {
  if (error) throw new Error(error);

  const collections = JSON.parse(body).collections;
  const files = [];

  // Loop through all collections and write them to file
  for (let i = 0; i < collections.length; i++) {
    const collectionUid = collections[i].uid;
    const collectionName = collections[i].name;

    const fileName = `${outputDir}/${collectionName}.json`;

    // Check if file already exists
    if (fs.existsSync(fileName)) {
      console.log(`Skipping ${fileName} as it already exists.`);
      continue;
    }

    const options = {
      url: `https://api.getpostman.com/collections/${collectionUid}`,
      headers: {
        'X-Api-Key': apiKey
      },
      proxy: 'http://username:password@localhost:8080'
    };

    request(options, function (error, response, body) {
      if (error) throw new Error(error);
      fs.writeFile(fileName, body, function(err) {
        if (err) throw new Error(err);
        console.log(`Saved ${fileName}`);
        files.push(fileName);
        checkIfAllFilesWritten();
      });
    });
  }

  // Check if all files have been written to disk
  function checkIfAllFilesWritten() {
    if (files.length === collections.length) {
      createArchive();
    }
  }

  // Create archive of all files inside the output directory
  function createArchive() {
    const output = fs.createWriteStream(`${outputDir}/${archiveName}`);
    const archive = archiver('zip', { zlib: { level: 9 } });

    output.on('close', function () {
      console.log(`Archived ${files.length} files into ${archiveName}`);
    //   uploadArchive();
    });

    archive.on('error', function(err){
      throw new Error(err);
    });

    archive.pipe(output);

    files.forEach(function(file) {
      archive.file(file, { name: file.split('/').pop() });
    });

    archive.finalize();
  }

  // Upload the archive to another API endpoint using multipart form data
  function uploadArchive() {
    const formData = new FormData();
    formData.append('archive', fs.createReadStream(`${outputDir}/${archiveName}`));

    const uploadOptions = {
      method: 'POST',
      url: 'https://api.example.com/upload',
      headers: {
        'Authorization': 'Bearer YOUR_TOKEN',
        ...formData.getHeaders()
      },
      formData: formData,
      proxy: 'http://username:password@localhost:8080'
    };

    request(uploadOptions, function (error, response, body) {
      if (error) throw new Error(error);
      console.log('Uploaded archive successfully.');
    });
  }
});

```
