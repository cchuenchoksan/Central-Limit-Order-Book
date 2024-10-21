# Central Limit Order Book

This project is to create a toy Central Limit Order Book which handles limit orders and display them on the CLOB while updating the successful trade when an order crosses the Bid-Ask spread. All logic is done purely on the frontend.

The project is written in Nuxt 3 (see below for details) with the tailwind css framework. The application requires a file upload which contains all the orders to be processed (file upload was done by papaparse). An example input file is `orders-1.csv`. To use this application, make sure the file is in the same format and uses a `.csv` extension.

## Nuxt 3 Minimal Starter

Look at the [Nuxt 3 documentation](https://nuxt.com/docs/getting-started/introduction) to learn more.

### Setup

Make sure to install the dependencies:

```bash
# npm
npm install

# pnpm
pnpm install

# yarn
yarn install

# bun
bun install
```

### Development Server

Start the development server on `http://localhost:3000`:

```bash
# npm
npm run dev

# pnpm
pnpm run dev

# yarn
yarn dev

# bun
bun run dev
```

### Production

Build the application for production:

```bash
# npm
npm run build

# pnpm
pnpm run build

# yarn
yarn build

# bun
bun run build
```

Locally preview production build:

```bash
# npm
npm run preview

# pnpm
pnpm run preview

# yarn
yarn preview

# bun
bun run preview
```

Check out the [deployment documentation](https://nuxt.com/docs/getting-started/deployment) for more information.
