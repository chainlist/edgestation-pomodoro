# Pomodoro

A focus timer for [EdgeStation](https://github.com/chainlist/edgestation): it alternates focus
sessions and breaks, with the time left, a progress bar and three buttons.

- **Play / Pause** starts or pauses the current phase.
- **Reset** puts the current phase back to its full length.
- **Skip** moves to the other phase.

When a phase ends, the next one waits for a tap to start.

The timer follows the clock rather than counting ticks, and each placement saves its own. It keeps
running while its page is off screen, while the Edge is paused for a full screen app, and while
EdgeStation is closed: it shows the right time again as soon as it is back on screen.

## Settings

Set per placement in Settings → Pages → click the widget:

| Setting      | Choices                      | Default    |
| ------------ | ---------------------------- | ---------- |
| Focus length | 15, 25, 45 or 50 minutes     | 25 minutes |
| Break length | 5, 10 or 15 minutes          | 5 minutes  |

A phase that has already started keeps its time left when you change its length; the new length
applies from the next one.

## Install

In EdgeStation, open Settings → Widgets → Browse, pick **Pomodoro** and click **Install**. New
versions install on their own when EdgeStation starts.

## Development

The widget is a Svelte 5 component built with
[`@edgestation/sdk`](https://www.npmjs.com/package/@edgestation/sdk).

```bash
npm ci
npm run build    # writes dist/main.js
npm run dev      # rebuilds on every change
npm run check    # type-checks
```

To load your build in EdgeStation, link this folder into your widgets folder once, from a Command
Prompt, then click **Rescan** in Settings → Widgets (and **Reload Edge** after each build):

```bat
mklink /J "%APPDATA%\com.edgestation.dashboard\widgets\pomodoro" C:\path\to\edgestation-pomodoro
```

## Releases

Releases are made by [release-please](https://github.com/googleapis/release-please) from
[Conventional Commits](https://www.conventionalcommits.org/) on `main`: `fix:` raises the patch
version, `feat:` the minor one. It keeps a release pull request open; merging it bumps `version`
in `manifest.json`, updates `CHANGELOG.md`, tags `v<version>` and creates the GitHub release, to
which the workflow attaches `manifest.json` and `widget.zip`.

The icon is Lucide's "timer" ([ISC license](https://lucide.dev/license)).
