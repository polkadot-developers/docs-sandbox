<h1 align="center">
  <a href="https://www.substrate.io">
    <img alt="Substrate Logo" src="./source/docs/img/sub.gif" width="70%" />
  </a>
</h1>
<h1 align="center"> Substrate Developer Hub SANDBOX </h1>
<h3 align="center"> TDB hosted github pages </h3>
<br/>

<!-- Badges -->

[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://docs.substrate.io/v3/contribute/style-guide/) [![Matrix](https://img.shields.io/matrix/frontier:matrix.org)](https://matrix.to/#/#substrate-technical:matrix.org)

<!-- Description -->

This repository serves as the **developer hub SANDBOX** for the **[Substrate](https://substrate.io)** blockchain
framework. The docs are written in pure [markdown](https://www.markdownguide.org/), processed by [MKdocs](https://www.mkdocs.org/).

## 🦾 Docs Migration Instructions:

The `./old-docs-migration` folder has all the ported docs, renamed for convenience to be `.md` files via this method:

> ###  TIL to migrate old `v3` dir dumped here:
> 
> To get old dir `<folder>/index.mdx` to `<folder>.md`:
> ```bash
> # move all `<folder>/index.mdx` to `<folder>.md`
> find ./ -depth -name "*.mdx" -exec sh -c 'mv $1 "${1%/index.mdx}.md"' _ {} \;
> # delete all empty folders  
> find /path/ -empty -type d -delete
> ```

Now our mission, if we choose to accept it... (we do!), is to port all the files here into the new **framework** outlined in our team Notion:

- [Notion Documentation Plan](https://www.notion.so/paritytechnologies/Documentation-plan-dcb201be22474ee294a9078e29b1e97a).

The framework will be implemented in the `./source/docs` folder.

## 🚀 Quick start


1. Clone the repo

   ```bash
   # create a new folder to get going
   git clone git@github.com:substrate-developer-hub/docs-sandbox.git
   ```

1. Serve the site with `livereload` for local development

   Navigate into your new site’s directory and install all dependencies.

   ```bash
   cd docs-sandbox/
   # options for `--theme [mkdocs|material|readthedocs]`
   mkdocs serve --theme material  --livereload 
   ```
   The site should be ready to view at

1. Make edits... make a PR

   ```bash
   # fancy gh CLI use here....
   ```

1. Deploy to `gh-pages` 


   ```bash
   # fancy gh CLI use here....
   ```


1. Profit 😎

## License

TBD

<!-- Substrate **documentation** is license under the [Apache 2 license](./LICENSE). -->
