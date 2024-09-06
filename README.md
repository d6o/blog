# My Blog

Welcome to my personal blog repository! This blog is built with [Hugo](https://gohugo.io/), a fast and flexible static
site generator. The theme used is [PaperMod](https://themes.gohugo.io/themes/hugo-papermod/). The blog covers topics on
technology, programming, and personal development.

You can visit the live blog at [blog.diego.dev](https://blog.diego.dev) and learn more about me on
my [personal website](https://diego.dev).

## Table of Contents

- [About](#about)
- [Running it locally](#running-it-locally)
- [Usage](#usage)
- [Configuration](#configuration)
- [Deployment](#deployment)
- [Contributing](#contributing)
- [License](#license)

## About

This blog is a personal project where I write about various topics related to technology, coding, and personal growth.
The blog is generated using Hugo, making it easy to manage and scale over time.

The repository contains all the source files needed to generate the blog, including:

- Markdown files for blog posts.
- Custom theme.
- Configuration files (e.g., `hugo.yaml`).
- Static assets like images and stylesheets.

Visit the live blog at [blog.diego.dev](https://blog.diego.dev).

## Running it locally

To get started with the blog locally, you need to have [Hugo](https://gohugo.io/getting-started/installing) installed.

### Steps to Install:

1. Clone the repository:

```bash
git clone https://github.com/d6o/blog.git && cd blog
```

2. Run the blog locally:

```bash
hugo serve
```

3. Open your browser at `http://localhost:1313` to see the blog in action.

## Usage

You can add new posts by creating a Markdown file inside the `content/posts/` directory. Hugo will automatically
generate the appropriate page for each post.

To create a new post:

```bash
hugo new posts/my-new-post.md
```

After editing your content, you can view the changes live by running `hugo serve` again.

## Configuration

You can customize the blog by modifying the `hugo.yaml` file located in the root of this repository.

## Deployment

Once you're ready to deploy the blog, you can generate the static files using:

```bash
hugo
```

This will generate the static site in the `public/` directory. You can then upload this directory to any web host.

## Contributing

Feel free to open issues or submit pull requests if you find any bugs or have suggestions for improvements.
Contributions are welcome!

## License

This project is licensed under the MIT License.
