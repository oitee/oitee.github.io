# oitee.github.io

This is a personal blog built with Jekyll. Taken from [here](https://github.com/sharu725/hagura)

## Local Development Setup

Follow these steps to set up and run the project on your local machine. These instructions are tailored for macOS with Homebrew. [_For Linux, follow the [official documentation](https://jekyllrb.com/docs/)_]

### 1. Install Prerequisites

First, if you don't have it, install Homebrew, a package manager for macOS.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Next, install Ruby using Homebrew:

```bash
brew install ruby
```

### 2. Configure Your Shell

This is a crucial step to ensure your system uses the correct version of Ruby. You need to add Homebrew's Ruby to your shell's `PATH`. For modern macOS (using Zsh), run the following command to add the configuration to your `.zshrc` file.

```bash
echo 'export PATH="/opt/homebrew/opt/ruby/bin:/opt/homebrew/lib/ruby/gems/3.4.0/bin:$PATH"' >> ~/.zshrc
```
*Note: The Ruby version (3.4.0) might be different if you install a newer version in the future. You can check the correct path in `/opt/homebrew/lib/ruby/gems/`.*

**Important:** You must restart your terminal after running this command for the changes to take effect.

### 3. Verify Your Ruby Installation

Open a **new** terminal window and verify that you are using the Homebrew version of Ruby.

```bash
which ruby && ruby -v
```

The output should point to a path inside `/opt/homebrew/...` and show a version like `3.4.x`. It should **not** be `/usr/bin/ruby`.

### 4. Install Bundler

Bundler is a tool that manages a project's gem dependencies. Install it globally once:

```bash
gem install bundler
```

### 5. Install Project Dependencies

Navigate to the project directory. Now, use Bundler to install all the gems specified in the `Gemfile`, including Jekyll itself.

```bash
bundle install
```

### 6. Run the Jekyll Server

Use `bundle exec` to run the `jekyll` command. This ensures you are using the version of Jekyll installed for this project.

```bash
bundle exec jekyll serve
```

The server will be running. You can view the site by navigating to **http://127.0.0.1:4000** in your web browser.

To stop the server, press `Ctrl + C` in your terminal.
