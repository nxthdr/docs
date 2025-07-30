# nxthdr.dev

This is the source code of [https://docs.nxthdr.dev](https://docs.nxthdr.dev).


## Development

This website is built with [Hugo](https://gohugo.io/), a static site generator. To run the site locally, you need to have Hugo installed on your machine.

Once, this is done, you can clone the repository and run the following commands to load the theme:

```sh
git submodule sync && git submodule update --init --recursive
```

Then, you can run the following command to start the development server:

```sh
hugo server
```

The website will be available at [http://localhost:1313](http://localhost:1313).


### Update the theme

Once in a while, we want to update the theme to the latest version. To do so, run the following command:

```sh
git submodule update --remote --merge
```
