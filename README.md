## Create .env file

```bash
# replace iobio-url with the url of the gru backend service.

echo "IOBIO_URL=iobio-url" > .env
```

[Sample env file](./.env.template)

## Starting App

Using node version 8.5.x

```
npm install
npm start
```

Now open [http://localhost:3000](http://localhost:3000).

To watch for client-side file changes, open up a new terminal window and `npm run webpack`.

## Running Client-side Tests

`npm test`
