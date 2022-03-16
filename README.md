# How every Minnesota legislative district changed in redistricting

This is a tool for visualizing the changes to Minnesota legislative districts caused by the 2022 redistricting. It overlays the outline
of the old district with outlines of all the new districts that intersect it, showing how the previous district was divided in the new plan.

## Get started

Install the dependencies...

```bash
npm install
```

...then start [Rollup](https://rollupjs.org) for local development:

```bash
npm run dev
```

## Building and running in production mode

To create an optimised version of the app:

```bash
npm run build
```

## Deploying

For MinnPost purposes, upload the contents of `public/build/` to S3 and add appropriate links to the files in the cms. The app targets
a `.minnpost-app` element.