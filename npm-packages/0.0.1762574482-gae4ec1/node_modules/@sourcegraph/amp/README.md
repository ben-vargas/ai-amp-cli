# Amp CLI

Amp is a frontier coding agent for your terminal and editor, built by Sourcegraph.

- **Multi-Model:** Sonnet, GPT-5, fast models—Amp uses them all, for what each model is best at.
- **Opinionated:** You're always using the good parts of Amp. If we don't use and love a feature, we kill it.
- **On the Frontier:** Amp goes where the models take it. No backcompat, no legacy features.
- **Threads:** You can save and share your interactions with Amp. You wouldn't code without version control, would you?

Amp's agent has 2 modes:

- `smart`: unconstrained state-of-the-art model use (Claude Sonnet 4.5, GPT-5, and more)
- `free`: [free of charge](https://ampcode.com/free), using fast basic models

<br/>

<img src="https://static.ampcode.com/content/amp-cli-20251026-1.gif" width="800" height="463" alt="Amp CLI">

## Get Started

1. Sign into [ampcode.com/install](https://ampcode.com/install) and follow the instructions to install the Amp CLI.
2. Run `amp` and ask Amp to do something in your codebase.

See the [Amp Owner's Manual](https://ampcode.com/manual) and [Switch to Amp](https://ampcode.com/manual/switch-from) for more information, and see [ampcode.com/news](https://ampcode.com/news) for what we've recently shipped.

## Installation

Install globally with your preferred package manager:

```bash
pnpm add -g @sourcegraph/amp@latest
# or
yarn global add @sourcegraph/amp@latest
# or
npm install -g @sourcegraph/amp@latest
```

Alternatively, run without installing:

```bash
npx -y @sourcegraph/amp@latest
```

## Usage

After installation, run `amp`.

For non-interactive environments (e.g. scripts, CI/CD pipelines), set your API key in the `AMP_API_KEY` environment variable.

See `amp --help` and the [Amp CLI documentation](https://ampcode.com/manual#cli) for more information.

## Support

For help and feedback: mention [@AmpCode](https://x.com/AmpCode) on X, or email [amp-devs@sourcegraph.com](mailto:amp-devs@sourcegraph.com). You can also join our community [Build Crew](https://buildcrew.team).

For account and billing help, contact [amp-devs@sourcegraph.com](mailto:amp-devs@sourcegraph.com).
